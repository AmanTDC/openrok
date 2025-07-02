# 📄 PRD: Web Dashboard (Optional UI)
1. Overview
The Web Dashboard is a user-facing UI to view, inspect, and manage tunnels.

2. Goals
View active tunnels and URLs.

See request logs (via Gateway).

View usage limits or quotas.

Revoke tokens / manage subdomains (future).

3. User Stories
As a user, I want to log in and see my tunnels.

I want to inspect live requests per tunnel.

I want to delete or restart tunnels.

I want to:
- Update my authentication token
- Manage account settings and preferences

4. Pages / UI Views
View	Description
Dashboard	List of active tunnels, with URLs and ports
Request Inspector	View latest incoming HTTP requests
Auth Settings	Manage tokens (static or OAuth)
Error Logs	View recent tunnel errors/disconnects

5. Tech Stack
Role	Tech
Frontend	React + Tailwind
Backend	Same API as Controller
Charts	Recharts, or LiveView (WebSocket logs)

6. Authentication
Login via token only (MVP)

Use Authorization: Bearer <token> for all API requests

No OAuth in MVP

7. Dependencies
Tunnel Controller API (for data)

Gateway logs (optional for request inspector)

Auth token for session

