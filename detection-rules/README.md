# Detection Rules

Detection logic used by the BGP Security Monitoring Lab to identify suspicious routing behavior and BGP control-plane anomalies.

## Primary lab detections

### 1. Prefix hijacking

Raise a `possible_prefix_hijack` event when a monitored prefix is observed with an unexpected origin AS.

**Example**

- Expected origin: AS100
- Monitored prefix: `100.100.100.0/24`
- Observed origin: AS200
- Result: `origin_match=false`

Validation lifecycle:

```text
Normal 0 -> Hijack 1 -> Withdraw 0
```

### 2. Route leak

Raise a `possible_route_leak` event when a monitored prefix is observed with an unexpected AS path.

**Example**

- Expected AS path: `100`
- Observed AS path: `200 100`
- Result: `possible_route_leak`

## Supporting BGP detections

The lab design can also be extended to detect:

- Unexpected BGP neighbor establishment
- BGP session flaps above a defined baseline
- New or withdrawn prefixes outside an approved change window
- Route advertisements from unauthorized ASNs
- Sudden AS-path changes
- Unexpected next-hop changes
- Excessive route churn
- Administrative shutdown/startup events
- BGP notification and session-reset events

## Event fields

Each detection event should preserve enough context for investigation, including:

| Field | Purpose |
|---|---|
| Event type | Detection classification |
| Prefix | Affected network/prefix |
| Source peer | Router or BGP peer producing the observation |
| Origin AS | Observed origin ASN |
| AS path | Observed BGP path |
| Timestamp | Time of the observation |
| Reason | Why the event was classified as suspicious |
| Detection status | Current/active or historical state |

## Detection pipeline

```text
BGP state / syslog
       |
       v
Collection + normalization
       |
       v
Logstash
       |
       v
Detection logic
       |
       +----> possible_prefix_hijack
       |
       +----> possible_route_leak
       |
       v
Elasticsearch
       |
       v
Kibana dashboard
```

