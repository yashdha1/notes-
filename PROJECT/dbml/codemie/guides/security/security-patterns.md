# Security Patterns Guide

**Source**: api.md, design.md, stories.md
**Auth mechanism**: RS256 JWT (stateless access token + Redis-revocable refresh token)
**Gateway**: Nginx routes only — JWT validation happens inside each microservice

---

## Auth Flow Overview

```
1. Register / Login
   └─► user-service issues:
         access_token  → RS256 JWT, short-lived (15 min)
         refresh_token → opaque UUID, stored in Redis with TTL

2. Client stores tokens:
   access_token  → in memory (Authorization header per request)
   refresh_token → secure storage, used only at /auth/refresh

3. Per-request auth (stateless):
   Client → Authorization: Bearer <access_token>
   Service → decode JWT locally using public key
           → verify: signature, exp, iss, aud
           → extract: sub (user_id), role, uname, avatar
   (No call to Auth Service or Redis on the hot path)

4. Token refresh:
   POST /auth/refresh { "refresh_token": "..." }
   └─► user-service:
         - decodes refresh token JWT → gets jti
         - looks up Redis: refresh:{jti} → must exist (not revoked)
         - issues new access_token
         - optionally rotates refresh token (delete old, insert new in Redis)

5. Logout:
   POST /auth/logout (access token in header)
   └─► user-service:
         - reads jti from refresh token (sent in body)
         - DEL refresh:{jti} from Redis → token permanently revoked
         - access token remains valid until expiry (stateless — cannot be revoked early)
```

---

## JWT Specification

### Access Token (RS256)

| Claim | Type | Value | Notes |
|-------|------|-------|-------|
| `sub` | str | user ID | Primary identity claim |
| `role` | str | `admin \| mod \| user` | Authorization claim |
| `uname` | str | username | Included for write-time denormalization |
| `avatar` | str \| null | avatar URL | Included for write-time denormalization |
| `iss` | str | `"user-service"` | Issuer — validated by all services |
| `aud` | str | `"social-media-api"` | Audience — validated by all services |
| `exp` | int | Unix timestamp | Short-lived (15 min recommended) |

> **Why `uname` and `avatar` in the token?**
> Content-service and others denormalize author info at write time (post creation, comment creation).
> Including these as non-security claims avoids a cross-service call on every write.
> They are NOT used for authorization decisions — only `sub` and `role` are auth-critical.
> Stale values are refreshed when the client calls `/auth/refresh` (token rotation).

### Refresh Token (Redis-backed)

The refresh token is a JWT signed with RS256, with the following shape:

| Claim | Value |
|-------|-------|
| `sub` | user ID |
| `jti` | unique UUID (used as Redis key suffix) |
| `iss` | `"user-service"` |
| `exp` | long-lived (7 days) |

**Redis storage pattern**:
```
Key:   refresh:{jti}
Value: {"user_id": 1, "role": "user"}   (JSON string)
TTL:   604800 seconds (7 days)
```

Validation: decode JWT → extract `jti` → `GET refresh:{jti}` → must not be nil.
Revocation: `DEL refresh:{jti}` (logout or rotation).

---

## Key Management

| Key | Owner | Distribution |
|-----|-------|-------------|
| Private key (RS256) | user-service **only** | Never shared, never exposed |
| Public key (RS256) | All microservices | Via `JWT_PUBLIC_KEY` env var or mounted PEM file |

**Config required in every service** (including user-service):
```
JWT_PUBLIC_KEY=<PEM string>    # for verification
JWT_ISSUER=user-service
JWT_AUDIENCE=social-media-api
```

**Additional config in user-service only**:
```
JWT_PRIVATE_KEY=<PEM string>   # for signing — never put in other services
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=15
JWT_REFRESH_TOKEN_EXPIRE_DAYS=7
```

---

## Per-Service JWT Validation Pattern

Every service has `deps.py` with a `get_current_user` dependency. Services validate the JWT
locally — no network call to user-service or Redis.

```python
# src/<pkg>/api/deps.py
from dataclasses import dataclass
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import jwt, JWTError
from .core.config import settings

security = HTTPBearer()

@dataclass
class UserContext:
    id: int
    role: str
    uname: str
    avatar: str | None

def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
) -> UserContext:
    try:
        payload = jwt.decode(
            credentials.credentials,
            settings.JWT_PUBLIC_KEY,
            algorithms=["RS256"],
            audience=settings.JWT_AUDIENCE,
            issuer=settings.JWT_ISSUER,
        )
    except JWTError:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid or expired token")

    return UserContext(
        id=int(payload["sub"]),
        role=payload["role"],
        uname=payload["uname"],
        avatar=payload.get("avatar"),
    )
```

**Usage in routers**:
```python
@router.post("/posts")
async def create_post(
    body: CreatePostRequest,
    user: UserContext = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
):
    ...
```

---

## Role System

| Role | Capabilities |
|------|-------------|
| `user` | Create/edit/delete own posts and comments |
| `mod` | Delete/edit any post or comment (moderation), promote users to mod |
| `admin` | Everything + promote/demote roles (admin ↔ mod ↔ user) |

Role is stored in `users.role` (user-service DB) and included in the JWT payload as the `role` claim.

**Ownership check pattern**:
```python
def can_modify(user: UserContext, resource_owner_id: int) -> bool:
    return user.id == resource_owner_id or user.role in ("mod", "admin")
```

---

## Token Issuance (user-service only)

```python
# services/user_service/src/user_service/services/auth.py
from jose import jwt
from datetime import datetime, timedelta, timezone
import uuid

def create_access_token(user_id: int, role: str, uname: str, avatar: str | None) -> str:
    payload = {
        "sub": str(user_id),
        "role": role,
        "uname": uname,
        "avatar": avatar,
        "iss": settings.JWT_ISSUER,
        "aud": settings.JWT_AUDIENCE,
        "exp": datetime.now(timezone.utc) + timedelta(minutes=settings.JWT_ACCESS_TOKEN_EXPIRE_MINUTES),
    }
    return jwt.encode(payload, settings.JWT_PRIVATE_KEY, algorithm="RS256")

async def create_refresh_token(user_id: int, role: str, redis: Redis) -> str:
    jti = str(uuid.uuid4())
    payload = {
        "sub": str(user_id),
        "jti": jti,
        "iss": settings.JWT_ISSUER,
        "exp": datetime.now(timezone.utc) + timedelta(days=settings.JWT_REFRESH_TOKEN_EXPIRE_DAYS),
    }
    token = jwt.encode(payload, settings.JWT_PRIVATE_KEY, algorithm="RS256")
    ttl = settings.JWT_REFRESH_TOKEN_EXPIRE_DAYS * 86400
    await redis.set(f"refresh:{jti}", str(user_id), ex=ttl)
    return token
```

---

## Password Reset Flow

```
1. POST /auth/password-reset/request { email }
   └─► Creates password_reset_tokens row { user_id, token, expires_at, used: false }
   └─► Sends OTP/link to email
   └─► Returns 200 EVEN if email doesn't exist (prevents user enumeration)

2. POST /auth/password-reset/confirm { token, new_password }
   └─► Validates: token exists, used == false, expires_at > now
   └─► Updates: users.password = bcrypt(new_password)
   └─► Marks: password_reset_tokens.used = true
   └─► Returns 400 if token expired, already used, or invalid
```

**Schema**:
```dbml
Table password_reset_tokens {
  id           int       [pk, increment]
  user_id      int       [not null]
  token        varchar   [not null]
  expires_at   timestamp [not null]
  used         boolean   [not null, default: false]
  created_at   timestamp
}
```

---

## Internal Endpoint Protection

Internal endpoints are **NOT exposed via Nginx gateway**. Only reachable within the internal network.

| Endpoint | Service | Callers |
|----------|---------|---------|
| `POST /internal/posts/batch` | content-service :8002 | feed-service only |

> **Note**: The `POST /internal/auth/validate` endpoint has been removed.
> JWT validation is now stateless — each service validates the token locally.

---

## Input Validation

All inputs validated via Pydantic schemas at the router layer:

```python
# schemas/auth.py
from pydantic import BaseModel, EmailStr, field_validator

class RegisterRequest(BaseModel):
    email: EmailStr
    uname: str
    password: str

    @field_validator("password")
    def password_strength(cls, v):
        if len(v) < 8:
            raise ValueError("Password must be at least 8 characters")
        return v
```

**Required validations**:
- Email: `EmailStr` (Pydantic built-in)
- Password: minimum length 8
- Unique constraints: enforced at DB level (email, uname) → catch `IntegrityError` → return 409
- Missing required fields: FastAPI returns 422 automatically

---

## HTTP Status Codes for Auth

| Status | Scenario |
|--------|---------|
| 200 | Successful auth operation |
| 400 | Invalid/expired token, validation error |
| 401 | Missing auth header, bad credentials, expired access token |
| 403 | Authenticated but insufficient role |
| 409 | Duplicate email or username on register |

**Never return 404 for "user not found" on login** — always return 401 (prevents user enumeration).

---

## Password Storage

Always hash passwords with bcrypt before storing:
```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(plain: str) -> str:
    return pwd_context.hash(plain)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)
```

---

## Token Security Rules

| Rule | Detail |
|------|--------|
| Refresh token revocation | Redis `DEL refresh:{jti}` — instant revocation |
| Access token expiry | Short-lived (15 min); cannot be revoked (stateless) |
| Refresh token rotation | Optional: DEL old jti, SET new jti in Redis on each /refresh call |
| Password reset token | Single-use (`used = true` after confirm), stored in DB with `expires_at` |
| Enumerate protection | Return 200 even if email not found on password-reset/request |
| Role checks | Check `role` claim from JWT payload — never trust client-supplied role in body |
| Private key | Only in user-service; never in env vars of other services |

---

## WebSocket Auth

WS connections pass the access token as a query parameter (HTTP headers are not available in WS upgrades):

```
wss://host/ws/post/{id}?token=<access_token>
wss://host/ws/notifications?token=<access_token>
```

Each service's WS handler validates the token on connection using the same `jwt.decode()` logic.

---

## Security Boundaries

| ✅ DO | ❌ DON'T |
|-------|---------|
| Validate JWT in each service using the public key | Call user-service for auth validation on every request |
| Store only `sub`, `role`, `uname`, `avatar` in access token | Include sensitive fields (password hash, email) in JWT |
| Store refresh token JTI in Redis with TTL | Store refresh tokens in the DB (hard to revoke) |
| Use bcrypt for password hashing | Use MD5, SHA1, or plain text |
| Return 401 for wrong credentials (not 404) | Reveal whether email exists via different status codes |
| Mark reset tokens as `used = true` immediately after confirm | Allow token reuse |
| Keep private key strictly in user-service only | Share private key via env vars to other services |
| Restrict internal endpoints to internal network only | Expose `/internal/*` through Nginx gateway |
| Check ownership OR role before allowing edit/delete | Allow any authenticated user to modify any resource |
