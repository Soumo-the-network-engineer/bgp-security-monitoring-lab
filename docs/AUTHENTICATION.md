# Authentication, Credentials, and Token Handling

## Scope

This repository documents a closed BGP security lab. Credentials are runtime concerns and must not be committed to Git.

## Authentication components

### GitHub repository access

Git operations use the authenticated GitHub developer session outside the project code. No GitHub password, PAT, OAuth secret, or SSH private key is stored in the repository.

### Cisco router collection

The lab uses SSH-based BGP collection. The documented MAIN collector account is `bgpcollector`; its secret is intentionally redacted.

The expected flow is:

```
Collector
  |
  | load runtime SSH credentials
  v
MAIN / vIOS8
  |
  | execute BGP state command
  v
JSON-lines parser
  |
  v
Logstash -> Elasticsearch
```

For the vIOS8 collector, SSH compatibility options are documented in `config/README.md`. Runtime credentials must be injected outside Git.

### Elasticsearch / Logstash

The Logstash pipelines connect to Elasticsearch over HTTPS/TLS using the dedicated `bgp_logstash` writer account. The password is redacted. The CA is referenced from the local Logstash certificate path.

The request flow is:

```
Logstash
   |
   | HTTPS + TLS validation
   | authenticated writer account
   v
Elasticsearch
   |
   v
bgp-* / bgp-routes-* indexes
```

## Credential lifecycle

1. Provision secrets outside the repository.
2. Load them only at runtime.
3. Use encrypted transport where supported.
4. Redact secrets from logs and telemetry.
5. Rotate/revoke credentials before reuse outside the lab.

## Never commit

- Router passwords
- Elasticsearch passwords
- API keys or bearer tokens
- OAuth client secrets
- SSH private keys
- TLS private keys
- Production connection strings containing credentials
- Company logs containing sensitive data

## Token handling rules

- Prefer least-privilege accounts.
- Prefer short-lived/scoped tokens where supported.
- Keep TLS certificate validation enabled.
- Never log Authorization headers or passwords.
- Never store credentials in dashboard exports.
- Treat a leaked token/password as compromised and rotate it immediately.

## Example runtime pattern

```python
import os

token = os.environ["MONITORING_API_TOKEN"]
headers = {"Authorization": f"Bearer {token}"}

# Perform the HTTPS request without logging the token or headers.
```

GitHub authentication and application/router authentication are separate trust boundaries.
