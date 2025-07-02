.

📄 PRD: TLS Termination Service
1. Overview
TLS Termination is responsible for securely handling incoming HTTPS (and optionally TCP+TLS) connections from the public internet, decrypting the request, and forwarding it to the internal Tunnel Gateway.

In MVP, TLS termination may be integrated into the Tunnel Gateway, or run as a standalone edge proxy (e.g., nginx, Caddy, Envoy).

2. Goals
Accept public HTTPS connections securely.

Terminate TLS at edge.

Support wildcard or on-demand certs.

Forward decrypted traffic to internal services (Gateway).

Handle automatic cert renewal and rotation.

3. Responsibilities
Feature	Description
Serve HTTPS	Listen on 443 and decrypt incoming HTTPS requests
Cert Management	Load certs from disk, or dynamically via ACME
Wildcard Certificate Support	MVP: use *.example.com cert
SNI-Based Routing	Route based on domain (e.g., foo.example.com)
Forward to Gateway	Proxy request over HTTP to internal Gateway
Redirect HTTP to HTTPS	Optional 80→443 redirect handler

4. MVP Features
Feature	Scope
TLS 1.2+ support	✅
Wildcard cert support	✅
Self-signed cert fallback	✅
Cert reload without restart	✅
Static cert loading from disk	✅
Forward decrypted request to Gateway	✅
No client cert support	❌ (non-MVP)

5. Deployment Modes
Mode	Description
Integrated Mode	TLS handled by the Tunnel Gateway itself (default MVP)
Proxy Mode	TLS termination handled by a proxy (e.g., nginx, Caddy) → forwards to internal Gateway
Hybrid	Proxy + embedded TLS, supports scaling per site/subdomain

6. Cert Storage Options
Type	Description
Static PEM/Key	Loaded from local disk (certs/) on startup
Let's Encrypt	Automated via ACME (renew every 60–90 days)
Vault Integration	(Future) secure cert storage via secret manager

7. Config Options (MVP)
toml
Copy
Edit
[tls]
enabled = true
cert_path = "./certs/wildcard.pem"
key_path = "./certs/wildcard.key"
auto_reload = true
redirect_http = true
8. Security Requirements
Use TLS 1.2 or 1.3 only.

Disable weak ciphers.

HSTS headers (optional).

Reload certificates without downtime.

Reject expired or mismatched certs.

9. Technology Options
Tool	Reason
rustls	Embedded Rust TLS termination (fast, safe)
| nginx | Simple, stable, easy-to-configure proxy |
Caddy	Automated Let's Encrypt support
Envoy	Scalable, powerful service proxy (advanced use)

10. Interfaces
Accepts incoming traffic from public internet (port 443).

Forwards to:

HTTP-only Tunnel Gateway (localhost:8080)

TCP forwarding module (future)

11. Dependencies
Certificate provider (manual or ACME)

Gateway (next hop)

DNS records for subdomain routing

12. Optional Extensions (non-MVP)
Feature	Description
SNI-based custom certs	Per-domain TLS
Client certificate validation	Mutual TLS (mTLS)
Rate limiting TLS handshakes	Anti-DoS

