# BGP Security Monitoring Lab — Project Report

## End-to-End BGP Security Monitoring, Prefix Hijack Detection and Route-Leak Analysis

| Project | BGP Security Monitoring Lab |
|---|---|
| Environment | EVE-NG / Cisco IOSv |
| Monitoring Stack | Logstash · Elasticsearch · Kibana |
| Time Synchronization | Chrony / NTP |
| BGP Domains | AS100 · AS200 · AS300 |
| Primary Security Tests | Prefix Hijacking · Route Leak |
| Author | Soumallya Das |

> **Project status:** Successfully implemented and validated in a closed laboratory environment.



## Executive Summary

This project implements a closed EVE-NG laboratory for BGP security monitoring using Cisco IOSv, SSH-based BGP state collection, Logstash, Elasticsearch, Kibana and Chrony/NTP. The two primary detection scenarios are prefix hijacking and route leaks.

AS200 was used to inject the monitored prefix `100.100.100.0/24` while AS100 remained the expected origin. The live pipeline classified the observation as `possible_prefix_hijack`, and Kibana changed from zero to one active prefix hijack. After withdrawing the test advertisement and allowing the short observation window to expire, the active metric returned to zero.

A second test used vIOS8 to compare the expected AS path `100` with an observed `200 100` path. The latter was continuously classified as `possible_route_leak`.

## 1. Project Problem Statement

BGP can remain operational while its route decisions become insecure or unintended. A session-up alarm does not explain whether a monitored prefix is being originated by the expected AS or whether an unexpected AS path has appeared.

The engineering question for this lab was:

> Can a small multi-AS EVE-NG topology continuously collect BGP route state, apply explicit security expectations, detect prefix hijacking and route leaks, and present the results in an operational Kibana dashboard?

## 2. Project Objectives

- Build a multi-AS BGP topology in EVE-NG.
- Implement route-reflector-based iBGP in AS200.
- Collect BGP route state over SSH.
- Parse route state into JSON-lines records.
- Enrich records with expected origin and AS-path metadata.
- Store route state and Cisco syslog in Elasticsearch.
- Build a Kibana dashboard for current and historical visibility.
- Validate a prefix-hijack lifecycle.
- Validate a route-leak condition.
- Centralize time synchronization with Chrony on BGP-ELK.

## 3. System Architecture and Topology

### AS100 — Legitimate Origin
MAIN-INT-RTR is the primary monitored router and legitimate origin.

### AS200 — Route-Reflector and Hijack Test Domain
vIOS4 and vIOS3 act as route reflectors. vIOS7 and vIOS6 are route-reflector clients. vIOS7 is also the controlled prefix-hijack injector.

### AS300 — External Observer
vIOS8 is used as the external path/leak observer.

### Monitoring and Analytics
BGP-ELK at `192.168.20.11` provides Logstash, Elasticsearch, Kibana and Chrony/NTP.

### Verified networks

| Network | Purpose |
|---|---|
| 10.10.10.0/24 | AS100-AS200 connectivity |
| 11.11.11.0/24 | AS100-AS200 connectivity |
| 12.12.12.0/24 | AS100-AS300 connectivity |
| 13.13.13.0/24 | AS200-AS300 connectivity |
| 192.168.20.0/24 | Lab services / ELK / NTP |
| 10.158.47.0/24 | Management / upstream network |

## 4. BGP Security Detection Model

### Prefix hijacking

Expected origin for `100.100.100.0/24` is AS100.

If the observed origin becomes AS200:

`expected_origin_as = 100`

`origin_as = 200`

`origin_match = false`

`security.event_type = possible_prefix_hijack`

### Route leak

For the vIOS8 monitored view, expected AS path is `100`.

An observed `200 100` path is classified as:

`security.event_type = possible_route_leak`

`security.as_path_match = false`

## 5. Network and Cisco Configuration

MAIN-INT-RTR uses:

`100.100.100.1/24` on Loopback10

`10.10.10.1/24` on Gi0/0

`11.11.11.1/24` on Gi0/1

`192.168.20.1/24` on Gi0/2.10

`12.12.12.1/24` on Gi0/4

SSH v2, local authentication and RSA 2048-bit host keys are used for route collection.

The router sends syslog to `192.168.20.10` and uses `192.168.20.11` as the NTP server.

vIOS7 originates the normal test prefix:

`70.70.70.7/32`

The controlled hijack is injected with:

`network 100.100.100.0 mask 255.255.255.0`

and withdrawn by removing that network statement and shutting the test loopback.

## 6. Connectivity, NAT and Management

MAIN provides the management boundary between `192.168.20.0/24` and `10.158.47.0/24`. A route-map-based PAT policy excludes the management subnet from self-NAT and permits normal outbound traffic.

The lab also required a Windows route for `192.168.20.0/24` via the MAIN management interface and an ICMP firewall exception on Windows for validation.

## 7. BGP State Collection and Normalization

The MAIN collector uses SSH and parses `show ip bgp` output into JSON-lines objects such as:

`router`

`prefix`

`next_hop`

`as_path`

`origin_as`

`origin`

`best`

The vIOS8 route parser produces records with `possible_route_leak` state.

Example validated records:

```json
{"router":"vIOS8","prefix":"100.100.100.0/24","as_path":"100","origin_as":"100","possible_route_leak":false}
{"router":"vIOS8","prefix":"100.100.100.0/24","as_path":"200 100","origin_as":"100","possible_route_leak":true}
```

## 8. Logstash Ingestion and Detection Pipelines

The lab uses three functional pipelines:

| Pipeline | Purpose |
|---|---|
| main | Cisco syslog collection |
| route_state | MAIN BGP route-state collection and origin validation |
| vios8_route | vIOS8 route-state collection and AS-path validation |

Route-state documents are indexed into `bgp-routes-YYYY.MM.dd`.

Security enrichment adds fields such as:

`security.expected_origin_as`

`security.origin_match`

`security.expected_as_path`

`security.as_path_match`

`security.event_type`

## 9. Elasticsearch Data Layer

Elasticsearch cluster:

`bgp-security-lab`

Node:

`BGP-ELK`

The cluster runs as a single-node lab deployment. Route-state and syslog indices are queried from Kibana using the `bgp-*` data-view pattern.

## 10. Kibana Security Dashboard

The final dashboard contains:

- Active Route Leaks
- Active Prefix Hijacks
- BGP Event Types
- BGP Events Over Time
- BGP Neighbor Activity
- BGP Notifications
- BGP Prefix Hijacking evidence
- BGP Route Leaks
- BGP Route Leaks Over Time
- BGP AS topology map

Operational time windows use recent observations so repeated 60-second snapshots are not mistaken for separate simultaneous incidents.

## 11. Time Synchronization

BGP-ELK provides central time synchronization through Chrony and UDP/123.

The server was verified with:

`chronyc tracking`

`chronyc sources -v`

`chronyc clients -n`

and `ss -lunp | grep :123`.

vIOS8 was observed synchronized to `192.168.20.11`. MAIN synchronization remained a separate unresolved lab issue.

## 12. Test Plan and Validation Results

### TC-01 — Baseline

Expected:

`Active Prefix Hijacks = 0`

Result: **Passed**

### TC-02 — Prefix Hijack Injection

AS200 originated `100.100.100.0/24`.

Expected:

`Active Prefix Hijacks = 1`

Result: **Passed**

### TC-03 — Elasticsearch Detection

Detected records contained:

`possible_prefix_hijack`

`origin_as = 200`

`expected_origin_as = 100`

`origin_match = false`

Result: **Passed**

### TC-04 — Hijack Withdrawal

The AS200 advertisement was withdrawn and the test loopback shut down.

The legitimate AS100 path returned as best.

Result: **Passed**

### TC-05 — Recovery Window

Using a short dashboard time window, the active hijack metric returned from 1 to 0.

Lifecycle:

`Normal 0 -> Hijack 1 -> Withdraw 0`

Result: **Passed**

### TC-06 — Route Leak

vIOS8 observed:

`200 100`

instead of the expected:

`100`

The event was classified as `possible_route_leak`.

Result: **Passed**

### TC-07 — NTP

Chrony was listening on UDP/123 and clients were visible.

Result: **Passed**

## 13. Troubleshooting and Engineering Lessons

### SSH after router reload

A MAIN reload removed RSA host keys, causing SSH to be disabled. Regenerating RSA 2048-bit keys restored the route collector.

### Elasticsearch startup

Elasticsearch encountered a machine-learning native-code startup failure during the project session. The log pointed to disabling X-Pack ML as the bypass for this lab environment.

### Windows reachability

The management route and ICMP firewall policy had to be corrected before Windows could reach the lab management subnet.

## 14. Limitations and Future Enhancements

This is a deterministic, prefix-specific laboratory detector. It is not a production BGP security service.

Future production-oriented enhancements include RPKI validation, authoritative prefix ownership data, allowlists, multi-source correlation, stateful incident tracking, alerting, webhook integration and richer topology analytics.

The AS topology map uses synthetic lab display coordinates and is not a real Internet geolocation feed.

## 15. Conclusion

The lab achieved its primary objective: a closed multi-AS BGP environment can continuously collect routing state, classify known anomalies and expose them in an operational dashboard.

The most important live validation proved an end-to-end lifecycle from route change to analytics and demonstrated that the monitoring pipeline can distinguish a legitimate origin from a controlled unauthorized origin:

`Normal 0 -> Hijack 1 -> Withdraw 0`

A persistent AS-path anomaly remained visible as an active route leak.

## Appendix A — Key Verification Commands

`show ip bgp`

`show ip bgp <prefix>`

`show ip ssh`

`show crypto key mypubkey rsa`

`show ntp associations`

`show ntp associations detail`

`show dhcp lease`

`show ip nat statistics`

`show access-lists BGPELK-NAT`

`systemctl status logstash --no-pager`

`chronyc tracking`

`chronyc clients -n`

`ss -lunp | grep :123`

## Appendix B — Credential Handling

Passwords, Elasticsearch credentials, private keys and other secrets are redacted.

Rotate all lab credentials before reuse outside the original environment.

Do not commit real production logs, private keys, passwords, tokens or other sensitive data.

## Appendix C — Reproduction Procedure

1. Power up the EVE-NG topology.
2. Verify interface addressing and BGP sessions.
3. Verify MAIN SSH collection.
4. Verify Elasticsearch, Logstash, Kibana and Chrony.
5. Confirm a normal dashboard state.
6. Inject the AS200 prefix hijack.
7. Observe the Elasticsearch event and Kibana metric.
8. Withdraw the hijacked route.
9. Confirm recovery to zero active hijacks.
10. Observe the persistent vIOS8 route-leak condition.
