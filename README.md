# BGP Security Monitoring Lab

A hands-on BGP security monitoring and anomaly-detection lab built with Cisco IOSv, EVE-NG, Logstash, Elasticsearch, Kibana, SSH-based BGP state collection, Cisco syslog, and Chrony/NTP.

## Project objective

Detect and visualize two important BGP security conditions:

1. Prefix hijacking — an unexpected AS originates a monitored prefix.
2. Route leaks — an unexpected AS-path is observed for a monitored prefix.

## Architecture

Cisco IOSv -> SSH/syslog collection -> Logstash -> Elasticsearch -> Kibana

The lab also uses BGP-ELK as the central NTP/Chrony server.

## BGP domains

- AS100 — MAIN-INT-RTR / legitimate origin
- AS200 — iBGP route-reflector domain and hijack test domain
- AS300 — external domain used for path/leak testing


## Dashboard

The Kibana dashboard includes:

- Active Route Leaks
- Active Prefix Hijacks
- BGP Event Types
- BGP Events Over Time
- BGP Route Leaks Over Time
- BGP Notifications
- BGP Neighbor Activity
- BGP Prefix Hijacking details
- BGP AS World Map

## Repository contents

- `config/` — sanitized Cisco, collection, Logstash and detection configuration reference
- `detection-rules/` — detection logic overview
- `scripts/` — collection/automation documentation
- `lab/` — topology and device-role documentation
- `dashboards/` — dashboard documentation
- `assets/` — asset manifest for topology, architecture and dashboard evidence
- `report/` — Markdown annual project report
- `docs/` — architecture and authentication documentation
- `PROJECT_PUBLICATION_STATUS.md` — publication checklist/status

## Validation lifecycle

The prefix-hijack scenario was validated end-to-end:

`Normal 0 -> Hijack 1 -> Withdraw 0`

The route-leak scenario was also validated with repeated observations of the suspicious AS path and a corresponding `possible_route_leak` classification.

## Author

Soumallya Das
