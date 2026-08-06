# grafana-dashboard-engineering-lab
Hands-on lab for engineering operational, technical, executive, and customer-facing Grafana dashboards using Prometheus, OpenTelemetry, APIs, SQL databases, exporters, and multiple data sources.

---

## Roadmap

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
