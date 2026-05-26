Two way interactive communication between user and server Fast API provides built-in, high-performance support for Web Sockets, a protocol that enables **bidirectional, full-duplex communication** between a client and a server over a single, persistent TCP connection. This makes it ideal for real-time applications like chat apps, live notifications, and collaborative tools. 

https://medium.com/@nmjoshi/getting-started-websocket-with-fastapi-b41d244a2799 


features : 
1. Unlike traditional HTTP (which is request-response based), WebSockets allow both the client and server to send messages at any time without repeatedly opening new connections.
2. **`@app.websocket` decorator:** You define a WebSocket endpoint using the `@app.websocket` decorator instead of standard HTTP verb decorators like `@app.get` or `@app.post`. in FastAPI. 
3. **`WebSocket` object:** The endpoint function takes a `WebSocket` object as a parameter, which provides methods for managing the connection.
	- **`await websocket.accept()`:** This method must be called explicitly to accept the incoming connection from the client; otherwise, the connection will be rejected.
	- **`WebSocketDisconnect` exception:** This exception is raised when a client disconnects, allowing you to handle the disconnection gracefully (e.g., cleanup resources, notify other clients).
	