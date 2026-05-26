## 1. JWT AUTH 

workflow : 
1. Login → server sets **both tokens** in cookies
2. Every request → browser auto-sends access token cookie
3. Get `401` → call `/refresh` → server sets new access token cookie
4. Retry the original request
#### Microservice Auth with JWT, Access/Refresh Tokens & Redis
##### Core Architecture 
Authentication is **centralized** (Auth Service), authorization is **distributed**. An API Gateway sits in front of all microservices and handles JWT validation, so individual services stay clean.
##### How JWT Works
- Login happens **once** at the Auth Service — credentials are verified, and two tokens are issued.
- The JWT **is** the access token (not wrapped inside it). It contains a payload (userId, role, expiry) signed with a secret key.
- On every request, the server **recomputes the signature** and compares it — no database call needed. If the payload is tampered, the signature won't match and the request is rejected.

##### Access Token vs Refresh Token
|          | Access Token                   | Refresh Token             |
| -------- | ------------------------------ | ------------------------- |
| Format   | JWT                            | Random string (preferred) |
| Lifetime | 5–15 minutes                   | Days/weeks                |
| Sent via | `Authorization: Bearer` header | HTTP-only cookie          |
| Purpose  | Authenticate every request     | Get a new access token    |

##### FastAPI-Specific Flow
- A `get_current_user` dependency validates the JWT on every request. If expired → return `401`, nothing more.
- A separate `/refresh` endpoint handles token renewal. The client calls it when it gets a 401, and the browser automatically sends the refresh token cookie.
- **Never auto-refresh inside `get_current_user`** — that breaks stateless design.
##### Where Redis Fits
- Stores refresh tokens (`token → userId`) enabling logout and revocation.
- Can maintain a blacklist for invalidated access tokens.
- Without Redis, you can use JWT refresh tokens (stateless), but you **lose the ability to revoke** tokens — a security trade-off.
##### Key Rules
- Short-lived access tokens, long-lived refresh tokens
- Rotate refresh tokens on every use
- Never store refresh tokens in localStorage (XSS risk)
- Never store access tokens in cookies (CSRF risk)

```javascript
import axios from "axios";

const api = axios.create({
  baseURL: "http://localhost:8000",
  withCredentials: true, // 👈 set once, applies everywhere
});

export default api;
```

Enable in the frontend. 
in the cors configuration : 
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
  CORSMiddleware,
  allow_origins=["http://localhost:3000"],  
  allow_credentials=True,                   # 👈 required for cookies
  allow_methods=["*"],
  allow_headers=["*"],
)
```

--- 



pg admin command : docker run -d --name pgadmin --network pg-network -e PGADMIN_DEFAULT_EMAIL=admin@example.com -e PGADMIN_DEFAULT_PASSWORD=admin123 -p 8080:80 dpage/pgadmin4 

### Postgres docker images wali shit : 
put all the docker containers of the docker network and the databases and the pgadmin. 

hosted on the localhost: 8080 :-> 
> docker network create pg-network # to start the network 

then start all the databases : 
docker run -d \  
--name AuthDB \  
--network pg-network \  
-e POSTGRES_USER=auth \  
-e POSTGRES_PASSWORD=auth \  
-e POSTGRES_DB=AuthDB \  
-p 5432:5432 \  
postgres:15

docker run -d \  
--name ContentDB \  
--network pg-network \  
-e POSTGRES_USER=content\  
-e POSTGRES_PASSWORD=content \  
-e POSTGRES_DB=ContentDB\  
-p 5432:5432 \  
postgres:15

docker run -d \  
--name AuthDB \  
--network pg-network \  
-e POSTGRES_USER=auth \  
-e POSTGRES_PASSWORD=auth \  
-e POSTGRES_DB=AuthDB \  
-p 5432:5432 \  
postgres:15


docker run -d \  
--name AuthDB \  
--network pg-network \  
-e POSTGRES_USER=auth \  
-e POSTGRES_PASSWORD=auth \  
-e POSTGRES_DB=AuthDB \  
-p 5432:5432 \  
postgres:15

> docker network connect pg-network AuthDB/ContentDB/NotificationDB/FeedDB

