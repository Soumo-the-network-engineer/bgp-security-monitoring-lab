# BGP Security Monitoring Lab

A hands-on BGP security monitoring and anomaly-detection lab built with Cisco IOSv, EVE-NG, Logstash, Elasticsearch, Kibana, SSH-based BGP state collection, Cisco syslog, and Chrony/NTP.

## Project objective

Detect and visualize two important BGP security conditions:

1. Prefix hijacking - an unexpected AS originates a monitored prefix.
2. Route leaks - an unexpected AS-path is observed for a monitored prefix.

## Architecture

Cisco IOSv -> SSH/syslog collection -> Logstash -> Elasticsearch -> Kibana

The lab also uses BGP-ELK as the central NTP/Chrony server.

## BGP domains

- AS100 - MAIN-INT-RTR / legitimate origin
- AS200 - iBGP route-reflector domain and hijack test domain
- AS300 - external domain used for path/leak testing

## Security tests

### Prefix hijacking

The lab temporarily originated `100.100.100.0/24` from AS200 while AS100 remained the expected origin. The monitoring pipeline detected `possible_prefix_hijack` with `origin_match=false` and the dashboard changed from 0 to 1. After withdrawing the test route, the active hijack metric returned to 0 in the selected recent-observation window.

### Route leak

The vIOS8 collector compares the expected AS path `100` with an observed `200 100` path. The latter is tagged and classified as `possible_route_leak`.

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

- `report/` - annual project report in DOCX and PDF format
- `assets/` - architecture, topology, dashboard, and validation figures
- `configs/` - intentionally sanitized configuration area
- `LINKEDIN_POST.md` - publication-ready LinkedIn draft

## Validation lifecycle

The prefix-hijack scenario was validated end-to-end:

`Normal 0 -> Hijack 1 -> Withdraw 0`

The route-leak scenario was also validated with repeated observations of the suspicious AS path and a corresponding `possible_route_leak` classification.

## Security note

This repository is designed for public documentation. Do not commit real passwords, private SSH keys, certificates, API tokens, company logs, or other sensitive production data.

## Author

Soumallya Das
