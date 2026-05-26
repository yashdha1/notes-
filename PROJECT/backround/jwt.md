
1. Header - decoded -> contains the algorithm and the type of token in the decoded format. 
2. Payload - decoded -> contains the payload and the user details.  (also the ttl)
3. signature (signature) -> contains the user details and the secret key 


Now to store the token in the frontend we have few options: 
1. *Localstorage* : simplest, presist even after the refresh. but vunarable to the XSS attacks. 
2. *Session storage* : cleared whenever the tab closes. still vulnarable to the XSS attacks. 
3. *HTTP Cookies* : safest, not acceessible via the javascript. safer against the XSS attacks. need the CSRF protection. here the backend will send the req to set the cookies - 
	1. Set-Cookie: token=jwt_token; HttpOnly; Secure; SameSite=Strict

then the client send the token with every request everytime. in the request header. 
Autherization : `Bearer ${token}`

ex: 
**CLIENT** : send the authetication token with every request
```js
fetch("/api/profile", {
  method: "GET",
  headers: {
    Authorization: `Bearer ${token}`
  }
}); // sending the request
```

**server** : verify like this for every request. 
```js
const token = req.headers.authorization.split(" ")[1];
const decoded = jwt.verify(token, process.env.JWT_SECRET);
req.user = decoded; 
```

general flow : 
```
User Login
    ↓
Backend verifies credentials
    ↓
Backend generates JWT [sends the access and the refresh token.]
    ↓
Frontend stores JWT (localStorage/cookie)
    ↓
User sends request
    ↓
Authorization: Bearer <token> [This is in the header.]
    ↓
Backend verifies JWT
    ↓
Access granted
```

* access token -> send the token with everyrequest. **TTL is less than 1hr**  
* refresh token -> generate a new access token when the access token expires. **TTL is longer like 30 days or so**    

#### In JWT systems verification is done via the : 
1. You validate a JWT to make sure the token makes sense, adheres to the expected standards, contains the right data.
2. You verify a JWT to make sure the token hasn't been altered maliciously and comes from a trusted source.

#### Difference Between Decoding and Encoding a JWT

*Encoding* a JWT involves transforming the header and payload into a compact, URL-safe format. The header, which states the signing algorithm and token type, and the payload, which includes claims like subject, expiration, and issue time, are both converted to JSON then Base64URL encoded. These encoded parts are then concatenated with a dot, after which a signature is generated using the algorithm specified in the header with a secret or private key. This signature is also Base64URL encoded, resulting in the final JWT string that represents the token in a format suitable for transmission or storage.

*Decoding* a JWT reverses this process by converting the Base64URL encoded header and payload back into JSON, allowing anyone to read these parts without needing a key. However, "decoding" in this context often extends to include verification of the token's signature. This verification step involves re-signing the decoded header and payload with the same algorithm and key used initially, then comparing this new signature with the one included in the JWT. If they match, it confirms the token's integrity and authenticity, ensuring it hasn't been tampered with since issuance.