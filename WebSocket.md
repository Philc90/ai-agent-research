Features
- can handle text and binary data
- limit for open websocket connections in server

WebSockets vs REST
- WebSockets require more server resources, more complex to implement than REST API
- no built-in reconnection mechanism
- more difficult to scale horizontally
	- potentially stateful
	- may not work with load balancers (?)

REST: Request/Response
- receiver needs to poll on interval
![[Pasted image 20250921203054.png]]

WebSocket: Full Duplex
- server can broadcast message to receiver
![[Pasted image 20250921203146.png]]

# Browser API Comparison

- [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
	- stable, good browser support
	- doesn't support [backpressure](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API/Concepts#backpressure)
	- "**Note:** If a page has an open WebSocket connection, the browser may not add it to the [bfcache](https://developer.mozilla.org/en-US/docs/Glossary/bfcache). It's therefore good practice to close the connection when the user has finished with the page. See [working with the bfcache](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications#working_with_the_bfcache)."
- [WebSocketStream](https://developer.mozilla.org/en-US/docs/Web/API/WebSocketStream)
	- Experimental, less support
		- not supported by Firefox
	- Promise-based
	- Uses [Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API)
- [Webtransport](https://developer.mozilla.org/en-US/docs/Web/API/WebTransport/WebTransport)
	- less support
		- not all features supported by major browsers
	- eventually replace WebSocket
	- HTTP 3
	- "If standard WebSocket connections are a good fit for your use case and you need wide browser compatibility, you should employ the WebSockets API to get up and running quickly. However, if your application requires a non-standard custom solution, then you should use the WebTransport API."

# Security
Methods
- Use Secure WebSocket (wss://)
- authn & authz with JWT
- user input validation (injection attacks)
	- origin checking (XSS, CSRF)
- rate limiting
- CORS
- https://brightsec.com/blog/websocket-security-top-vulnerabilities/
	- Data masking can stop security tools from analyzing traffic
	- Ticket-based authn
- https://websocket.org/guides/security/
	- ✅ **TLS/SSL encryption** (wss:// protocol only)
	- ✅ **Origin validation** to prevent CSWSH (Cross-site WebSocket Hijacking) attacks
	- ✅ **Authentication** during handshake or immediately after
	- ✅ **Input validation** for all messages
	- ✅ **Rate limiting** per connection and globally
	- ✅ **Message size limits** to prevent DoS
	- ✅ **Timeout mechanisms** for idle connections
	- ✅ **Security headers** configured properly
	- ✅ **Logging and monitoring** for suspicious activity
	- ✅ **Regular security updates** for WebSocket libraries

# Sources
https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API
https://www.geeksforgeeks.org/blogs/how-to-secure-your-websocket-connections/
https://www.youtube.com/watch?v=fG4dkrlaZAA