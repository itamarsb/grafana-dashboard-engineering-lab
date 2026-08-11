# PromQL Queries — Linux Infrastructure Dashboard

This document contains the PromQL queries used throughout the
**LAB 02 — Linux Infrastructure Dashboard**.

The metrics are collected from a Linux host using **Prometheus Node Exporter**,
stored by **Prometheus**, queried using **PromQL**, and visualized in **Grafana**.

---

## Monitoring Flow

```text
Linux Host
    │
    │ Operating System Metrics
    ▼
Node Exporter
    │
    │ HTTP :9100/metrics
    ▼
Prometheus
    │
    │ PromQL
    ▼
Grafana
    │
    ▼
Linux Infrastructure Dashboard

```
