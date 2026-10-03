# Architecture

## Components

1. **BGP lab routers** — establish eBGP/iBGP sessions and generate realistic control-plane events.
2. **Collectors** — gather BGP summaries, route tables, syslog, and session state.
3. **Detection layer** — evaluates normalized events against routing-security rules.
4. **Storage/indexing** — retains events for investigation and trend analysis.
5. **Dashboards** — present peer health, route changes, and alerts.
6. **Operator workflow** — validates an alert against router state before remediation.

## Request/event flow

```
Router
  │
  ├── BGP state / CLI or API
  └── Syslog
       │
       ▼
Collector
       │
       ▼
Normalizer
       │
       ▼
Detection rules
       │
       ├── no match ──► indexed telemetry
       │
       └── match ─────► alert/event record
                              │
                              ▼
                         Investigation
                              │
                              ▼
                       Router validation
```

The repository intentionally documents interfaces and flow without embedding environment-specific secrets or production endpoints.
