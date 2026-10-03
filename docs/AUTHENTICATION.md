# Authentication, Credentials, and Token Handling

## Scope

This project is a lab repository. Authentication is designed so that credentials are supplied at runtime and are never committed to Git.

## Components

### 1. GitHub repository access

Git operations are authenticated by the developer's GitHub credential/session outside the project code. The repository does not contain a GitHub password, PAT, SSH private key, or OAuth token.

### 2. Router access

Collectors may need to authenticate to lab routers using SSH or a router API. The application should read the username and secret from environment variables or a local secret provider.

Example environment contract:

- `BGP_ROUTER_HOST`
- `BGP_ROUTER_USER`
- `BGP_ROUTER_PASSWORD` (local runtime only)

A production-style deployment should prefer SSH keys or a managed secret store over plaintext passwords.

### 3. Monitoring/backend APIs

When the lab integrates with a monitoring or indexing service, the client sends a short-lived or scoped token using the provider's supported authorization header. Tokens must be supplied at runtime and excluded from logs.

## Request flow

### Router collection

```
Collector
   │
   │  1. Resolve endpoint
   │  2. Load runtime credentials
   │  3. Establish SSH/API session
   ▼
Router
   │
   │  4. Request BGP state
   ▼
Collector
   │
   │  5. Normalize response
   │  6. Emit sanitized event
   ▼
Detection / storage
```

### Backend API call

```
Detector/Collector
       │
       │ load runtime token
       ▼
HTTPS client
       │
       │ Authorization: Bearer <runtime-token>
       ▼
Monitoring/API service
       │
       │ 2xx response / error
       ▼
Client validation + sanitized logging
```

## Credential lifecycle

1. **Provision** — an operator stores a secret outside the Git repository.
2. **Load** — the process reads the secret at startup or request time.
3. **Use** — credentials are presented only to the target service over the appropriate secure protocol.
4. **Redact** — logs and error messages must omit passwords, private keys, access tokens, and authorization headers.
5. **Rotate** — revoke/replace secrets without changing committed source code.

## What must never be committed

- Passwords
- API keys
- Personal access tokens
- OAuth client secrets
- SSH private keys
- Router enable secrets
- Production connection strings containing credentials
- Session cookies

## Token handling rules

- Prefer least-privilege scopes.
- Prefer short-lived tokens where the provider supports them.
- Keep TLS certificate validation enabled.
- Never print an Authorization header.
- Never persist access tokens in telemetry payloads.
- Treat a leaked token as compromised: revoke/rotate it immediately.

## Git authentication vs application authentication

GitHub authentication used to push this repository is separate from credentials used by the lab's router/API collectors. The project code must not assume or reuse the GitHub session credential for runtime network access.

## Example pattern

```python
import os

token = os.environ["MONITORING_API_TOKEN"]
headers = {"Authorization": f"Bearer {token}"}

# Send the request over HTTPS; do not log headers.
```

The example intentionally leaves the HTTP client and endpoint provider unspecified so it can be adapted to the lab without hard-coding a service or secret.
