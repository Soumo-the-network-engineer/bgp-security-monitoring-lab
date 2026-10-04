# BGP Security Monitoring Lab

> **An end-to-end network security monitoring lab for detecting BGP prefix hijacking and route leaks using Cisco IOSv, EVE-NG, Logstash, Elasticsearch, and Kibana.**

---

## Overview

The **BGP Security Monitoring Lab** is a hands-on network security engineering project that demonstrates how BGP routing telemetry can be collected, normalized, analyzed, and visualized to identify abnormal routing behavior.

The lab recreates a multi-AS BGP environment in **EVE-NG** and integrates it with an **ELK-based monitoring pipeline**. BGP state is collected from Cisco IOSv routers, processed through Logstash, indexed in Elasticsearch, and presented through Kibana dashboards.

The project focuses on two security scenarios:

- **Prefix hijacking** — an unauthorized AS originates a monitored prefix.
- **Route leaks** — an unexpected AS-path is observed for a monitored prefix.

The complete detection lifecycle was validated in the lab from route manipulation through to dashboard visibility and alert classification.

---

## Project Objectives

The primary objectives are to:

1. Build a realistic multi-AS BGP laboratory environment.
2. Collect BGP routing state from Cisco IOSv devices.
3. Centralize BGP telemetry using Logstash and Elasticsearch.
4. Develop detection logic for abnormal BGP origin and AS-path behavior.
5. Visualize routing-security events in Kibana.
6. Validate the detection pipeline using controlled BGP security test cases.
7. Document the architecture, configuration, validation results, and engineering lessons.

---

## Architecture

### Monitoring Pipeline

```text
 Cisco IOSv Routers
        |
        | SSH / Syslog
        v
 BGP State Collection
        |
        v
    Logstash
        |
        v
 Elasticsearch
        |
        v
     Kibana
```

The lab also uses **BGP-ELK** as the central **Chrony/NTP time-synchronization server**, helping maintain consistent timestamps across routing events and monitoring data.

### End-to-End Flow

```text
BGP Event
   |
   v
Router State / Syslog
   |
   v
Collection & Parsing
   |
   v
Logstash Processing
   |
   +--> Prefix-Hijack Detection
   |
   +--> Route-Leak Detection
   |
   v
Elasticsearch
   |
   v
Kibana Dashboard
   |
   v
Security Visibility
```

---

## BGP Laboratory Domains

| AS | Device / Role | Purpose |
|---|---|---|
| **AS100** | MAIN-INT-RTR | Legitimate origin / primary monitored router |
| **AS200** | vIOS3 / vIOS4 / vIOS6 / vIOS7 | iBGP route-reflector domain and controlled hijack test domain |
| **AS300** | vIOS8 | External customer / route-leak observation domain |

### Supporting Infrastructure

| Component | Role |
|---|---|
| **BGP-ELK** | Logstash, Elasticsearch, Kibana and Chrony/NTP |
| **Old Syslog** | Legacy Cisco syslog collector |
| **EVE-NG** | Network emulation platform |

---

## Security Test Cases

### 1. Prefix Hijacking Detection

The monitored prefix is:

`100.100.100.0/24`

AS100 is the legitimate origin. During the controlled test, AS200 temporarily originates the same prefix.

The monitoring pipeline identifies the abnormal origin and classifies the event as:

```text
possible_prefix_hijack
origin_match=false
```

The dashboard lifecycle was validated as:

```text
Normal 0  ->  Hijack 1  ->  Withdraw 0
```

After the test route was withdrawn, the active hijack metric returned to zero within the selected observation window.

### 2. Route Leak Detection

The route-leak test uses vIOS8 in AS300.

The expected path is:

```text
100
```

The suspicious observed path is:

```text
200 100
```

The parser classifies the unexpected path as:

```text
possible_route_leak=true
```

This demonstrates detection of an unexpected intermediate AS in the observed routing path.

---

## Detection Capabilities

The project provides a foundation for detecting:

- Unauthorized BGP neighbor establishment
- BGP session flaps
- Unexpected prefix advertisements
- Prefix hijacking
- Route leaks
- Unauthorized ASN announcements
- AS-path changes
- Next-hop changes
- Excessive route churn
- BGP administrative events
- BGP notifications

The implemented validation focuses specifically on **prefix hijacking** and **route-leak detection**.

---

## Kibana Dashboard

The monitoring dashboard provides visibility into:

- **Active Route Leaks**
- **Active Prefix Hijacks**
- **BGP Event Types**
- **BGP Events Over Time**
- **BGP Route Leaks Over Time**
- **BGP Notifications**
- **BGP Neighbor Activity**
- **BGP Prefix Hijacking Details**
- **BGP AS World Map**

The dashboard is designed to provide both high-level security status and detailed routing-event visibility.

---

## Technology Stack

| Layer | Technology |
|---|---|
| Network Emulation | EVE-NG |
| Routing Platform | Cisco IOSv |
| Routing Protocol | BGP |
| Log Collection | SSH / Cisco Syslog |
| Data Processing | Logstash |
| Search & Analytics | Elasticsearch |
| Visualization | Kibana |
| Time Synchronization | Chrony / NTP |
| Operating Environment | Linux / Virtualized Lab |

---

## Repository Structure

```text
bgp-security-monitoring-lab/
│
├── assets/
│   ├── BINARY_ARTIFACT_MANIFEST.md
│   └── README.md
│
├── config/
│   └── README.md
│
├── dashboards/
│   └── README.md
│
├── detection-rules/
│   └── README.md
│
├── docs/
│   ├── ARCHITECTURE.md
│   └── AUTHENTICATION.md
│
├── evidence/
│   └── ELK_BGP_validation_evidence.txt
│
├── lab/
│   └── README.md
│
├── report/
│   └── PROJECT_REPORT.md
│
├── scripts/
│   └── README.md
│
├── .gitignore
├── PROJECT_PUBLICATION_STATUS.md
└── README.md
```

---

## Project Documentation

| Document | Description |
|---|---|
| **[Project Report](report/PROJECT_REPORT.md)** | Complete project report covering architecture, implementation, testing and results |
| **[Architecture](docs/ARCHITECTURE.md)** | System architecture and component relationships |
| **[Configuration](config/README.md)** | Sanitized router, collection, Logstash and detection configuration reference |
| **[Detection Rules](detection-rules/README.md)** | BGP security detection logic |
| **[Lab](lab/README.md)** | Topology, device roles and laboratory environment |
| **[Dashboards](dashboards/README.md)** | Kibana dashboard documentation |
| **[Evidence](evidence/ELK_BGP_validation_evidence.txt)** | Validation evidence from the ELK/BGP lab |
| **[Publication Status](PROJECT_PUBLICATION_STATUS.md)** | Repository publication and artifact status |

---

## Validation Summary

The project was validated using controlled routing events rather than relying only on static configuration.

### Prefix Hijack

```text
Legitimate State
      |
      v
Unauthorized Origin Introduced
      |
      v
Detection: possible_prefix_hijack
      |
      v
Dashboard: 0 -> 1
      |
      v
Route Withdrawn
      |
      v
Dashboard: 1 -> 0
```

### Route Leak

```text
Expected AS Path
      |
      v
100
      |
      v
Observed Suspicious Path
      |
      v
200 100
      |
      v
Detection: possible_route_leak
```

---

## Engineering Outcome

This project demonstrates a practical approach to **BGP security observability** by connecting routing infrastructure with centralized telemetry, detection logic, search, and visualization.

The lab provides a reproducible foundation for further development such as:

- Automated alerting
- Additional BGP anomaly detections
- Threat-intelligence enrichment
- Historical route analysis
- Automated incident response
- Prometheus/Grafana integration
- Production-scale telemetry ingestion
- Advanced AS-path anomaly scoring

---

## Project Status

**Status: Completed and validated in a closed laboratory environment.**

The core prefix-hijacking and route-leak scenarios have been implemented and validated end-to-end.

---

## Author

**Soumallya Das**  
Network Security Engineer

BGP Security Monitoring Lab — Network Security / Routing Security Engineering Project
