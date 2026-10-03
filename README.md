# BGP Security Monitoring Lab

A practical lab for monitoring BGP behavior and detecting common routing-security anomalies.

## Goals

- Build a repeatable BGP lab topology.
- Collect router/BGP state and logs.
- Detect suspicious route changes, unexpected peers, prefix anomalies, and session events.
- Separate lab secrets from committed configuration.
- Document operational response and validation steps.

## Project structure

```
.
├── README.md
├── docs/
│   ├── ARCHITECTURE.md
│   └── AUTHENTICATION.md
├── config/
│   └── README.md
├── detection-rules/
│   └── README.md
├── scripts/
│   └── README.md
├── lab/
│   └── README.md
├── dashboards/
│   └── README.md
└── .gitignore
```

## Security

No passwords, API keys, private keys, tokens, or production credentials belong in this repository. Use environment variables, local secret stores, or untracked files for sensitive values.

See [Authentication](docs/AUTHENTICATION.md) for the project's credential and token-handling model.

## Status

Initial project scaffolding and security documentation are committed on the `main` branch.
