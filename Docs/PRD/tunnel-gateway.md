# 📄 PRD: Tunnel Gateway (Edge Reverse Proxy)

## 1. Overview
The Tunnel Gateway is the central, internet-facing reverse proxy that:
- Accepts persistent tunnel connections from CLI clients
- Listens for public incoming requests (HTTP/TCP)
- Forwards those requests to the correct local client tunnel over a secure connection

## 2. Goals
- Terminate public traffic (HTTPS, TCP)
- Maintain persistent, multiplexed tunnels from clients
- Route requests based on subdomain / port to appropriate tunnel
- Maintain minimal latency, high availability

## 3. Core Responsibilities
| Function | Description |
|----------|-------------|
| Tunnel Registration | Accepts new tunnel requests (e.g., --subdomain foo) from CLI over WebSocket |
| Routing | Matches incoming public traffic to correct active tunnel |
| Forwarding | Forwards HTTP/TCP payloads to CLI client over the tunnel stream |
| Multiplexing | Supports multiple tunnels per client (future, not MVP) |
| TLS Termination | Handles HTTPS (using self-signed or Let's Encrypt) |
| Keepalive & Heartbeat | Monitors connection health to clients |
| Authentication | Verifies auth token by querying Controller (or local check) |

## 4. MVP Features
| Feature | Description |
|---------|-------------|
| WebSocket tunnel connection | Establish tunnel with CLI client |
| Incoming HTTP/TCP routing | Match subdomain or TCP port |
| Static TLS support | Serve HTTPS via one wildcard cert |
| In-memory routing table | Subdomain → tunnel mapping |
| Auth handshake | Check token inline or with Controller |
| Logging | Request logs, connection state, tunnel activity |
| Retry on failure | Forwarding retries, limited reconnect grace period |

## 5. Endpoints Handled
| Endpoint | Purpose |
|----------|---------|
| wss://tunnel.example.com/connect | WebSocket tunnel endpoint for CLI clients |
| https://*.example.com | Public endpoint to serve tunneled HTTP traffic |
| tcp://gw.example.com:xxxxx | TCP forwarding endpoint (future) |

## 6. Architecture (Simplified)
```
        +----------------+          +-------------------------+
        |   CLI Client   |<=======> |  Tunnel Gateway (Edge)  |
        |  (WebSocket)   |          |  - TLS/WS listener      |
        +----------------+          |  - HTTP/TCP proxy       |
                                   |  - Routing registry     |
     Public Web Traffic           +-----------+-------------+
          |                                   |
          v                                   v
+-------------------+              +---------------------------+
| https://foo.ex.com| --Route-->  | Active Tunnel to CLI App  |
+-------------------+              +---------------------------+
```

## 7. Routing Mechanism (MVP)
Use in-memory HashMap<String, TunnelConnection>:
- Keys: subdomain names (foo, bar)
- Values: open WebSocket tunnel to CLI client

## 8. Security Considerations
- TLS for all public traffic (HTTPS)
- Validate token on tunnel connection
- Rate limit invalid connections
- Prevent subdomain collisions
- Idle tunnel timeout

## 9. Tech Stack Suggestions
| Function | Stack |
|----------|-------|
| TLS/WebSocket | Rust with tokio, hyper, tokio-tungstenite |
| HTTP Proxying | hyper, reqwest, or raw TCP streams |
| TLS | rustls, or nginx in front (simpler TLS) |
| Routing | Internal table, Redis (future for clustering) |
| Auth | JWT validation or REST call to Controller |

## 10. Error Scenarios
| Scenario | Handling |
|----------|----------|
| Tunnel subdomain already in use | Reject with error |
| Client disconnects | Drop tunnel and unregister |
| Public request to dead tunnel | Return 502 |
| TLS handshake fails | Reject connection |
| Unregistered subdomain | Return 404 |

## 11. Dependencies
- Controller Service: for validating tokens, reserving subdomains
- CLI Client: must follow tunnel protocol
- TLS certs: can be provided via env/config
