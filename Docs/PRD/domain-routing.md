📄 PRD: Domain Router
1. Overview
The Domain Router manages the mapping from incoming public subdomains (like foo.example.com) or TCP ports to active tunnel connections. It enables dynamic routing of user-defined or randomly assigned domains to the correct tunnel at runtime.

It is often embedded in the Tunnel Gateway, but can also be a standalone routing service if supporting multiple gateways.

2. Goals
Map inbound domain requests to active tunnels.
- Support HTTP/HTTPS subdomains
- Support TCP port forwarding
- Enable dynamic routing updates
Prevent subdomain collisions.

Allow custom subdomain registration.

Support real-time updates to routing tables.

Scale to 100K+ concurrent mappings (memory or external store).

3. Responsibilities
Feature	Description
Subdomain Registration	Add new routing entry (e.g., foo → tunnel#A)
Collision Detection	Prevent two clients from claiming same subdomain
Routing Lookup	Given an incoming request (SNI or Host header), find the tunnel
Route Cleanup	Expire/remove tunnel routes on disconnect or timeout
Sync with Gateway(s)	Expose routing table or update interface for distributed use
Thread-Safe Updates	Ensure concurrent-safe insert/delete/read operations

4. Supported Mappings
Key	Type	Maps To
foo.example.com	HTTP/S Subdomain	Tunnel Client Session
tcp:49000	TCP Port	Tunnel Client TCP socket
uuid:xyz123	Internal Tunnel ID	Metadata (port, owner, expiry)

5. MVP Features
Feature	Scope
HTTP Host/SNI mapping	✅
In-memory routing map	✅
Add/Delete routes	✅
Random + custom subdomains	✅
Thread-safe read/write	✅
Route expiration on disconnect	✅
API-less embedding in Gateway	✅

6. Advanced (Post-MVP)
Feature	Description
Redis-backed routing map	Shared across multiple Gateways
DNS record injection	Dynamic wildcard DNS
Persistent routing metadata	Sync with DB for audit/reuse
Namespacing	Teams or user-isolated domains
Access control / IP filtering	Fine-grained tunnel-level ACLs

7. Internal API
If separated from Gateway, expose via gRPC or REST:

Method	Endpoint	Description
POST	/routes	Add route (subdomain → tunnel)
DELETE	/routes/{subdomain}	Remove route
GET	/routes/{subdomain}	Lookup route
GET	/routes	List all active routes (admin)

8. Data Model Example (In-Memory)
rust
Copy
Edit
HashMap<String, TunnelConnection>
json
Copy
Edit
{
  "foo.example.com": {
    "client_id": "cli-xyz123",
    "socket": "...",
    "expires_at": "2025-07-03T12:00:00Z"
  }
}
9. Technology Stack
Role	Stack
Embedded	Rust DashMap, Go sync.Map, Python dict + RLock
External (future)	Redis or etcd
Routing Layer	Custom or plug into NGINX via Lua/Redis
TTL / Expiry	tokio background task or Redis expiry keys

10. Failure Scenarios & Handling
Scenario	Response
Subdomain collision	Reject tunnel registration
Tunnel disconnect	Clean up route within N seconds
Lookup miss	Return HTTP 404 or TCP RST
Gateway restart	In-memory mode: all routes lost (MVP)

11. Security Considerations
Only authenticated users may reserve subdomains.

Input sanitation on subdomain strings.

Prevent SSRF via Host header spoofing.

Sanitize wildcards and illegal domains.

12. Dependencies
Required by: Tunnel Gateway (for routing traffic)

May sync with: Controller (for tunnel registration)

Optional: Redis or shared store for multi-node setups

Diagram (MVP Embedded in Gateway)
text
Copy
Edit
[ Incoming Request ]
       |
       v
+--------------+    Lookup Host
| Tunnel GW    |------------------+
| w/ TLS + WS  |                  |
|              |<--- foo.example.com ---+
+--------------+                    |
           |                        v
           +------> [Tunnel Socket Stream]
