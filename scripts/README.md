# Scripts

Automation and collection helpers for the BGP Security Monitoring Lab.

## Purpose

This directory is intended for scripts that support:

- BGP state collection from Cisco IOSv devices
- SSH-based collection and command execution
- JSON-lines parsing and normalization
- Logstash/Elasticsearch ingestion helpers
- BGP anomaly validation and test automation
- Repeatable lab evidence collection

## Collection workflow

```text
Cisco IOSv
    |
    | SSH / BGP state
    v
Collection script
    |
    | normalized events
    v
Logstash
    |
    v
Elasticsearch
    |
    v
Kibana
```

## Runtime configuration

Scripts should obtain runtime credentials and connection details from environment variables or an external secret provider.

Do not hard-code passwords, API keys, tokens, private keys, or other credentials in scripts.

## Expected output

Collection helpers should produce structured, machine-readable output where possible so that BGP events can be processed consistently by the Logstash pipeline and used by the detection rules.

