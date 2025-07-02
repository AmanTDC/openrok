# CLI Client (Tunnel Client)

## Overview
The CLI client enables users to securely expose local servers (HTTP/TCP) to the public internet by establishing a tunnel with a public gateway. It is the main interface for developers.

## Goals
- Let developers run simple CLI commands to expose services
- Open and maintain a persistent, secure tunnel connection
- Handle traffic forwarding between local app and remote users
- Support both HTTP(S) and TCP tunnels

## CLI Features

### MVP Scope
| Feature | Description |
|---------|-------------|
| tunnel http <port> | Expose local HTTP service (e.g., localhost:3000) |
| tunnel tcp <port> | Expose raw TCP service (e.g., SSH, database) |
| Random subdomain | Auto-generated e.g., random123.example.com |
| Custom subdomain | Optional flag: --subdomain myapp |
| Auth token | Passed via --token or config file |
| Reconnect logic | Auto-reconnect on dropped tunnel |
| Status info | Print active tunnel URLs |
| Logs | Basic output of requests and connection state |

### Non-MVP (Future)
- YAML config file (.tunnelrc)
- Persistent tunnels (auto-start on boot)
- CLI autocomplete and shell integration
- OAuth login flow
- Tunnel sharing/team access

## CLI Commands & Options
```bash
# Basic HTTP tunnel
$ tunnel http 3000

# With custom subdomain
$ tunnel http 3000 --subdomain myapp

# TCP tunnel
$ tunnel tcp 22

# Auth token
$ tunnel http 5000 --token <YOUR_TOKEN>

# Tunnel status
$ tunnel status
```

## Architecture Diagram
```
+-----------------+
| User's App (3000)|<---> Local TCP Forwarder
+-----------------+
         |
         v
+-------------------------+
|   CLI Tunnel Client     |
|-------------------------|
| - Auth w/ token         |
| - Dial WebSocket/TLS    |
| - Encode traffic stream |
| - Forward local ports   |
+-------------------------+
         |
         v
+----------------------------+
|     Tunnel Gateway (Edge)  |
+----------------------------+
```

## Client Responsibilities
- Initiate and manage control connection (WebSocket or HTTP2)
- Authenticate using bearer token
- Register tunnel (protocol message)
- Listen on local port and forward inbound traffic
- Handle reconnects with backoff
- Handle shutdown cleanly (Ctrl+C, SIGTERM)

## Configuration

### .tunnelrc (Optional)
```toml
token = "your-auth-token"
default_subdomain = "myapp"
```

Or via environment:
```bash
export TUNNEL_TOKEN=your-auth-token
```

## Tech Stack Recommendation
- Language: Rust or Go (Rust preferred for safety + performance)
- Protocol: WebSocket (with Protobuf or JSON messages)
- Local forwarding: TCP and HTTP
- Async runtime: Tokio (Rust), goroutines (Go)

## Error Handling
- Invalid port → show user error
- No internet → retry w/ backoff
- Invalid token → terminate with message
- Gateway unreachable → retry N times

## Dependencies
Requires:
- Public Gateway (to dial)
- Controller API (via Gateway)
- Not tightly coupled to UI, Dashboard, or rate limiter
