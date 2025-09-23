
Overview
- https://websocket.org/guides/security/
	- **TLS/SSL encryption** (wss:// protocol only)
	- **Origin validation** to prevent CSWSH (Cross-site WebSocket Hijacking) attacks
	- **Authentication** during handshake or immediately after
	- **Input validation** for all messages
	- **Rate limiting** per connection and globally
	- **Message size limits** to prevent DoS
	- **Timeout mechanisms** for idle connections
	- **Security headers** configured properly
	- **Logging and monitoring** for suspicious activity
	- **Regular security updates** for WebSocket libraries

# Types of Attacks
## TLS/SSL encryption
Use wss:// in production (secure WebSockets)

### Manage Certificates Effectively:

Proper certificate management is crucial for ensuring the security of your WebSocket connections. Using TLS certificates issued by trusted Certificate Authorities (CAs) and ensuring they are kept up-to-date helps protect against various security threats, including Man-in-the-Middle (MitM) attacks and unauthorized access. Here's a detailed look at how to manage certificates effectively:

****Importance of Certificate Management:****

- ****Encryption:**** TLS certificates encrypt the data transmitted between the client and server, ensuring that sensitive information remains confidential and is not accessible to unauthorized parties.
- ****Authentication:**** Certificates verify the identity of the server, ensuring that clients are connecting to the legitimate server and not an imposter, thereby preventing MitM attacks.
- ****Trust:**** Certificates issued by trusted CAs are recognized by clients as legitimate, fostering trust between the client and the server.

## Origin Checking
Attacks: CSWSH, CSRF
- validate origin headers on server side
- utilize CSRF Tokens
- Implement CORS

## Compression vulnerabilities

Attacks: [CRIME](https://en.wikipedia.org/wiki/CRIME)/[BREACH](https://en.wikipedia.org/wiki/BREACH)

- https://websocket.org/guides/websocket-protocol/#compression-extension-rfc-7692---security-considerations

Safe Compression Patterns

**DO:**

- Compress only public, non-sensitive data
- Use separate connections for sensitive vs. public data
- Apply compression selectively per message type
- Rate-limit to prevent timing analysis

**DON’T:**

- Mix secrets with attacker-controlled data
- Compress authentication headers or tokens
- Use compression for small messages (overhead > benefit)
- Ignore compression ratios in logs

Disable `permessage-deflate` compression unless specifically needed. Compression can introduce security vulnerabilities similar to CRIME/BREACH attacks where compression combined with secret data can leak information.

```
// Node.js - disable compression for security const
wss = new WebSocket.Server({   
	perMessageDeflate: false
});
```
## Authentication
Attacks: CSWSH
**Authentication** during handshake or immediately after
**Security headers** configured properly

Don't assume WebSocket connection equals unlimited access. Check authorization for each action: https://cheatsheetseries.owasp.org/cheatsheets/WebSocket_Security_Cheat_Sheet.html#message-level-authorization

## Rate Limiting
Attacks: DoS

**Rate limiting** per connection and globally

## Message Size Limit
Attacks: DoS

set limit to avoid memory exhaustion

## Input Validation
Attacks: injection attacks - XSS, SQL injection, command injection
- **Use parameterized queries** to prevent injection

**Validate message structure and content** using JSON schemas and allow-lists. Set reasonable size limits (typically 64KB or less) and implement rate limiting to prevent message flooding.

**For binary data**, verify file types using magic numbers rather than trusting content-type headers. Scan uploads for malware when appropriate, and use safe deserialization for protocols like protobuf or MessagePack.


**Always use `JSON.parse()` instead of `eval()`** for JSON processing - `eval()` enables code execution from untrusted input.

see https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html


## Heartbeat

Attacks: connection exhaustion attack

detect stale connections
- **Implement connection timeouts** for idle connections



Persistent WebSocket connections increase DoS risk.

**Limit connections and resources** by restricting total connections and implementing per-user limits (preferred) or per-IP limits where user identification isn't available. Set **message size limits** (typically 64KB or less) and implement **rate limiting** to prevent message flooding - 100 messages per minute is a common starting point.

**Handle idle and dead connections** by implementing idle timeouts to close inactive connections. Use **heartbeat monitoring** with ping/pong frames to detect and clean up dead connections.

**Implement backpressure controls** to prevent memory exhaustion from fast message producers. Many WebSocket implementations lack proper flow control, allowing attackers to overwhelm server memory by sending messages faster than they can be processed.


### Session Management

WebSocket connections often outlive normal sessions, requiring special handling.

**Use SameSite cookies** (`SameSite=Lax` or `Strict`) to prevent cross-site cookie transmission and strengthen CSWSH defenses.

**Handle session expiration** by implementing server-side validation for long-running connections. Close WebSocket connections when sessions expire. Re-validate user sessions periodically (every 30 minutes is common) to ensure they remain valid.


**When users log out**, close all their WebSocket connections immediately. Maintain a mapping of sessions to active connections so you can invalidate WebSocket access the moment logout occurs.

**Token-based authentication:** -- use JWT

For enhanced security, use token-based authentication instead of relying solely on cookies. Tokens can be passed in query strings (note: tokens will appear in access logs and should be redacted) or as part of WebSocket messages after connection establishment. Message-based token passing avoids log exposure but requires protocol design considerations.


## Security Monitoring
Log Security Events
- **Monitor for attack patterns** in real-time

Traditional HTTP access logs only capture the initial WebSocket upgrade request, not subsequent message traffic. You'll miss auth failures, injection attempts, rate-limit violations, and abuse.

**Log WebSocket events** including connection establishment and termination (with user identity, IP, and origin), authentication and authorization events during handshake and message processing, security violations like rate limiting triggers and message validation failures, and abnormal disconnections and protocol errors.

**Avoid logging sensitive data** - never log complete message contents, authentication tokens, session IDs, or personal information that could violate privacy regulations.

See the [Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html) for more details.

Detect anomalies

NOTE: Data masking can stop security tools from analyzing traffic
https://brightsec.com/blog/websocket-security-top-vulnerabilities/


## Keep Libraries Updated
Keep libraries updated to get latest security patches

- **Use Content Security Policy** headers

## Security Audit
Regular security audits
- **Have an incident response plan** for security breaches
- **Regular security audits** and penetration testing
- **Monitoring gaps**: Traditional HTTP logs only capture the initial upgrade request, missing all message traffic



### Service Tunneling Risks[¶](https://cheatsheetseries.owasp.org/cheatsheets/WebSocket_Security_Cheat_Sheet.html#service-tunneling-risks "Permanent link")

While WebSockets can tunnel TCP services (VNC, FTP, SSH), this creates security risks. If your application has XSS vulnerabilities, attackers could access these services directly from victims' browsers. If tunneling is necessary, implement additional authentication and access controls beyond the WebSocket layer.




# Sources
https://www.geeksforgeeks.org/blogs/how-to-secure-your-websocket-connections/
https://websocket.org/guides/security/
https://cheatsheetseries.owasp.org/cheatsheets/WebSocket_Security_Cheat_Sheet.html