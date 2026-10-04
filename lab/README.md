# Lab

This directory is intended for EVE-NG/GNS3 or equivalent topology definitions, router configuration templates, and test scenarios.

## 4.1 Topology Roles

| Node | AS | Role | Relevant interfaces / purpose |
|---|---:|---|---|
| **MAIN-INT-RTR** | 100 | Primary monitored router / provider side | Gi0/0 — 10.10.10.1; Gi0/1 — 11.11.11.1; Gi0/2.10 — 192.168.20.1; Gi0/3 — 10.158.47.88; Gi0/4 — 12.12.12.1; Lo10 — 100.100.100.1 |
| **vIOS4** | 200 | RR1 | AS200 iBGP route-reflector |
| **vIOS3** | 200 | RR2 | AS200 iBGP route-reflector |
| **vIOS7** | 200 | RR client / hijack injector | Gi0/2 — 10.10.10.2; Gi0/3 — 13.13.13.1; Lo100 — 70.70.70.7 |
| **vIOS6** | 200 | RR client | Gi0/2 — 11.11.11.2 |
| **vIOS8** | 300 | Customer / leak observer | Gi0/0 — 12.12.12.2; Gi0/1 — 13.13.13.2 |
| **BGP-ELK** | — | Analytics / NTP server | 192.168.20.11 |
| **Old syslog** | — | Legacy syslog collector | 192.168.20.10 |

