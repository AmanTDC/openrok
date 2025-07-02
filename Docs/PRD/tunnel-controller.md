# 📄 PRD: Tunnel Controller

## 1. Overview
The Tunnel Controller is the control plane of the system. It handles:
- Authentication of clients
- Subdomain reservation and validation
- Registration and tracking of active tunnels
- Central coordination for multi-node Gateways (future)

## 2. Goals
- Provide API endpoints for Tunnel Gateway and CLI to coordinate tunnel lifecycle
- Enforce security/authentication
- Avoid tunnel collisions
- Enable future dashboard integration

## 3. Core Responsibilities
| Feature | Description |
|---------|-------------|
| Token Validation | Check client auth tokens (JWT or static keys) |
| Tunnel Registration | Accept new tunnel creation requests |
| Subdomain Registry | Map subdomains to client tunnel sessions |
| Tunnel Lifecycle Tracking | Add/remove active tunnels, expiry |
| Rate Limiting & Quotas | Optional enforcement rules |
| Expose REST/gRPC API | Used by Gateway and Web Dashboard |

## 4. REST API Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/auth/validate | Validate token from client |
| POST | /api/tunnels/register | Reserve subdomain |
| DELETE | /api/tunnels/{id} | Remove a tunnel |
| GET | /api/tunnels/active | List active tunnels |
| GET | /api/users/me | (optional) Authenticated user info |

## 5. Tunnel Registry Example
```json
{
  "subdomain": "myapp",
  "client_id": "cli-7sdf23",
  "expires_at": "2025-07-03T16:00:00Z"
}
```

## 6. Technology Stack
| Role | Stack |
|------|-------|
| Server | FastAPI (Python), or Actix/Web (Rust), or Go gRPC |
| DB | PostgreSQL (subdomains), Redis (active tunnels) |
| Auth | JWT or static token with HMAC |
| Rate Limiting | Redis or in-memory token bucket |

## 7. Security
- Secure all APIs with token-based auth
- Sanitize subdomain input
- Add CORS protection for APIs
- Track per-user tunnel usage

## 8. Dependencies
- Used by: Tunnel Gateway and CLI Client
- Shared DB with: Dashboard UI (optional)
