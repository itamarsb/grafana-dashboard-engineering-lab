# Roadmap

## Phase 1 — Dashboard Engineering Fundamentals

### LAB 01 — Grafana Dashboard Design Fundamentals

- visual hierarchy;
- panel selection;
- units;
- thresholds;
- variables;
- information organization;
- common errors;
- operational vs. executive dashboards.


### LAB 02 — Linux Infrastructure Dashboard

- Node Exporter;
- Prometheus;
- CPU;
- memory;
- disk;
- network;
- availability;
- capacity.


### LAB 03 — Executive Infrastructure Dashboard

Using the same data from LAB 02, but transformed into:

- overall health index;
- capacity trend;
- risks;
- availability;
- key exceptions;
- executive summary.


---


## Phase 2 — Networks, devices, and infrastructure

### LAB 04 — SNMP Network Dashboard

- devices;
- interfaces;
- utilization;
- errors;
- availability;
- latency;
- capacity.


### LAB 05 — Smart PDU and Energy Dashboard

- consumption;
- power;
- demand;
- energy density;
- efficiency;
- anomalies;
- estimated cost;
- available capacity.


### LAB 06 — Infrastructure Comparison Dashboard

Comparison between two environments:

- legacy vs. modern;
- on-premises vs. cloud;
- centralized vs. distributed architecture;
- before vs. after.


---


## Phase 3 — APIs and applications

### LAB 07 — REST API and JSON Dashboard

- Public or custom API;
- Infinity;
- authentication;
- JSON transformation;
- handling missing data;
- external indicators.


### LAB 08 — FastAPI Application Dashboard

- requests;
- P50/P95/P99 latency;
- errors;
- throughput;
- availability;
- most used endpoints.


### LAB 09 — Custom Prometheus Exporter

- Python exporter;
- custom metrics;
- integration with Prometheus;
- visualization in Grafana.


### LAB 10 — OpenTelemetry Application Dashboard

- metrics;
- logs;
- traces;
- correlation between signals;
- technical view and executive view.


---


## Phase 4 — Databases and business indicators

### LAB 11 — PostgreSQL Business Dashboard

- sales, operations, or customer service;
- SQL queries;
- indicators by period;
- targets;
- trends;
- comparisons.


### LAB 12 — MySQL or MariaDB Dashboard

- relational source;
- queries;
- variables;
- filters;
- operational indicators.


### LAB 13 — Microsoft SQL Server Dashboard

- business data;
- departmental indicators;
- direct integration.


### LAB 14 — Oracle Dashboard Integration

- Oracle integration;
- plugin specifics;
- licensing limitations;
- API or exporter alternative.


---


## Phase 5 — Full observability

### LAB 15 — Logs with Loki

- application logs;
- infrastructure logs;
- filters;
- correlation with metrics.


### LAB 16 — Distributed Tracing with Tempo

- traces;
- services;
- dependencies;
- latency;
- bottleneck identification.


### LAB 17 — Unified Observability Dashboard

- Prometheus;
- Loki;
- Tempo;
- OpenTelemetry;
- Grafana;
- correlated metrics, logs, and traces.


### LAB 18 — SLI, SLO, and SLA Dashboard

- availability;
- latency;
- error rate;
- error budget;
- objective attainment;
- technical view;
- contractual view;
- executive view.


---


## Phase 6 — Advanced executive dashboards

### LAB 19 — CIO Technology Health Dashboard

- overall availability;
- risks;
- capacity;
- incidents;
- costs;
- trends;
- critical services.


### LAB 20 — Customer-Facing Service Dashboard

- SLA;
- performance;
- availability;
- significant incidents;
- monthly trends;
- executive summary.


### LAB 21 — Board-Level Technology Dashboard

In this lab, we will take a different approach by building a deliberately simple dashboard:

- five to eight indicators;
- non-technical language;
- risks;
- impact;
- trend;
- recommendations.


### LAB 22 — Executive Benchmark Dashboard

- two environments side-by-side;
- performance;
- cost;
- resilience;
- consumption;
- security;
- executive recommendation.


---


## Phase 7 — Dashboard Lifecycle and Automation

### LAB 23 — Dashboard JSON and Version Control

- export and import;
- JSON structure;
- version control;
- version comparison;
- change review;
- rollback.


### LAB 24 — Dashboard Provisioning

- `dashboards.yml` files;
- data source provisioning;
- dashboard provisioning;
- organization by folders;
- automatic updates.


### LAB 25 — Grafana Git Sync

- Git synchronization;
- branches;
- pull requests;
- history;
- approval workflow.

Grafana currently offers Observability as Code workflows, file-based provisioning, and Git Sync, enabling you to store dashboards in repositories and utilize version control and pull requests.


### LAB 26 — CI/CD Dashboard

- JSON validation;
- linting;
- detection of non-existent data sources;
- UID validation;
- automated deployment;
- GitHub Actions.


### LAB 27 — Variables and Reusable Dashboards

- query variables;
- custom variables;
- interval variables;
- chained variables;
- repeated panels;
- repeated rows;
- selection of environment, region, server, and service.


### LAB 28 — Transformations and Multi-Source Dashboards

- join;
- merge;
- organize fields;
- calculate field;
- reduce;
- filter;
- combining information from different sources.

Transformations allow you to combine, reorganize, and calculate results after the data is returned by the sources.


### LAB 29 — Annotations and Operational Context

- deployments;
- incidents;
- maintenance activities;
- configuration changes;
- periods of unavailability;
- business events.

Annotations allow you to mark events directly on time series, helping to correlate changes and incidents with metric behavior.


### LAB 30 — Dashboard Navigation and Drill-Down

- dashboard links;
- data links;
- links to logs;
- links to traces;
- links to runbooks;
- passing time ranges and variables.

Grafana allows you to preserve the time range and variables when navigating between dashboards, which is important for creating an integrated operational experience.


---


## Phase 8 — Alerting and Operational Response

### LAB 31 — Grafana Alerting Fundamentals

- alert rules;
- evaluation groups;
- contact points;
- notification policies;
- labels;
- annotations;
- Normal, Pending and Firing states.














