# Detection Rules

Recommended BGP security detections:

- Unexpected BGP neighbor establishment.
- BGP session flaps above a defined baseline.
- New or withdrawn prefixes outside an approved change window.
- Route advertisement from an unauthorized ASN.
- Sudden AS-path changes.
- Unexpected next-hop changes.
- Excessive route churn.
- Administrative shutdown/startup events.

Each rule should record the observed event, source peer, affected prefix/session, timestamp, and a reason for raising the alert.
