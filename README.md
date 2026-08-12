<p align="center">
    <img src="docs/images/EN_grafana_lab_overview_11_27_36.png">
</p>



# Grafana Dashboard Engineering Lab

Hands-on lab for engineering operational, technical, executive, and customer-facing Grafana dashboards using Prometheus, OpenTelemetry, APIs, SQL databases, exporters, and multiple data sources.

---

## Repository structure

```markdown

grafana-dashboard-engineering-lab/
│
├── README.md
├── roadmap.md
├── LICENSE
├── .gitignore
│
├── docs/
│   ├── dashboard-design-guidelines.md
│   ├── dashboard-review-checklist.md
│   ├── naming-conventions.md
│   ├── visualization-selection-guide.md
│   ├── executive-dashboard-principles.md
│   └── repository-architecture.md
│
├── templates/
│   ├── lab-readme-template.md
│   ├── dashboard-requirements-template.md
│   ├── dashboard-review-template.md
│   └── executive-summary-template.md
│
├── shared/
│   ├── provisioning/
│   ├── datasources/
│   ├── dashboards/
│   ├── sample-data/
│   └── scripts/
│
├── labs/
│   ├── 01-grafana-dashboard-design-fundamentals/
│   │   ├── README.md
│   │   ├── images/
│   │   │   ├── dashboard-overview.png
│   │   │   ├── panel-comparison.png
│   │   │   └── final-dashboard.png
│   │   ├── dashboards/
│   │   │   └── dashboard.json
│   │   ├── provisioning/
│   │   │   ├── dashboards.yml
│   │   │   └── datasources.yml
│   │   ├── queries/
│   │   ├── sample-data/
│   │   ├── scripts/
│   │   └── docker-compose.yml
│   │
│   └── 02-linux-infrastructure-dashboard/
│       ├── README.md
|       │
|       ├── images/
|       |   └── .gitkeep
|       │
|       ├── dashboards/
|       │   └── linux-infrastructure-dashboard.json
|       │
|       ├── prometheus/
|       │   └── prometheus.yml
|       │
|       └── queries/
|           └── promql.md
│
└── .github/
    ├── ISSUE_TEMPLATE/
    ├── pull_request_template.md
    └── workflows/
        ├── validate-dashboard-json.yml
        └── markdown-lint.yml

```


---


## Internal structure of each laboratory

Each laboratory will operate as an independent project. An example follows below:

```markdown

labs/02-linux-infrastructure-dashboard/
├── README.md
├── images/
├── dashboards/
│   └── linux-infrastructure-dashboard.json
├── provisioning/
│   ├── dashboards.yml
│   └── datasources.yml
├── prometheus/
│   └── prometheus.yml
├── queries/
│   └── promql-examples.md
├── alerting/
│   └── alert-rules.yml
├── scripts/
│   └── generate-load.sh
└── docker-compose.yml

```


---


---

## Architecture


```mermaid
flowchart TB
    subgraph Sources["Data Sources"]
        NE["Node Exporter"]
        SNMP["SNMP Exporter"]
        API["REST / JSON APIs"]
        CE["Custom Exporters"]
        PG["PostgreSQL"]
        ORA["Oracle Database"]
        MYSQL["MySQL"]
        MSSQL["Microsoft SQL Server"]
        LOGS["Application and System Logs"]
        OTELAPP["OpenTelemetry Instrumented Applications"]
    end

    subgraph Collection["Collection and Processing"]
        PROM["Prometheus"]
        OTEL["OpenTelemetry Collector"]
        LOKI["Loki"]
        TEMPO["Tempo"]
        ALLOY["Grafana Alloy"]
        INFINITY["Infinity Data Source"]
    end

    subgraph Visualization["Visualization and Analysis"]
        GRAFANA["Grafana"]
        OPS["Operational Dashboards"]
        TECH["Technical Management Dashboards"]
        EXEC["Executive and Customer Dashboards"]
    end

    NE --> PROM
    SNMP --> PROM
    CE --> PROM
    OTELAPP --> OTEL
    OTEL --> PROM
    OTEL --> LOKI
    OTEL --> TEMPO
    LOGS --> LOKI
    API --> INFINITY

    PG --> GRAFANA
    ORA --> GRAFANA
    MYSQL --> GRAFANA
    MSSQL --> GRAFANA

    PROM --> GRAFANA
    LOKI --> GRAFANA
    TEMPO --> GRAFANA
    ALLOY --> GRAFANA
    INFINITY --> GRAFANA

    GRAFANA --> OPS
    GRAFANA --> TECH
    GRAFANA --> EXEC

```

---

## 📈 Repository Metrics

<p align="center">

<a href="https://info.flagcounter.com/fvOS"><img src="https://s01.flagcounter.com/count/fvOS/bg_FFFFFF/txt_000000/border_CCCCCC/columns_8/maxflags_100/viewers_0/labels_1/pageviews_1/flags_0/percent_0/" alt="Flag Counter" border="0"></a>

</p>
