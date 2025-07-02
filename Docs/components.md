
| Component                    | Purpose                                                                                          | MVP Scope                                   |
| ---------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------- |
| **1. CLI Client**            | Let users expose local ports and create tunnel connections                                       | ✅ Full implementation                       |
| **2. Tunnel Gateway (Edge)** | Accepts WebSocket/TLS connections from CLI, receives incoming traffic, forwards it to the client | ✅ Full implementation                       |
| **3. Tunnel Controller API** | Handles authentication and registration of tunnels (subdomain ↔ tunnel mapping)                  | ✅ Minimal: auth, route registry             |
| **4. TLS Termination**       | Provides secure HTTPS tunnels (e.g., using Let's Encrypt or self-signed cert)                    | ✅ Basic TLS (can be static/self-signed)     |
| **5. Domain Router**         | Resolves incoming requests to tunnels based on subdomain                                         | ✅ Simple in-memory routing (no dynamic DNS) |
