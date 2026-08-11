# LAB 02 — Linux Infrastructure Dashboard

## Linux Infrastructure Monitoring with Node Exporter, Prometheus and Grafana

This laboratory introduces infrastructure monitoring using real Linux system metrics.

While **LAB 01 — Grafana Dashboard Design Fundamentals** focused on visualization principles, dashboard organization, panel selection, units, thresholds and information hierarchy, this laboratory moves into a real monitoring architecture.

The objective is to progressively build a Linux infrastructure monitoring environment based on:

- Linux;
- Node Exporter;
- Prometheus;
- PromQL;
- Grafana.

The final result of this laboratory will be a custom Linux infrastructure dashboard capable of providing operational visibility into:

- host availability;
- CPU utilization;
- system load;
- memory utilization;
- swap usage;
- filesystem capacity;
- disk activity;
- network traffic;
- system uptime;
- infrastructure capacity.

> This laboratory is intentionally built step by step.  
> Instead of importing a pre-built Grafana dashboard, the panels and queries will be created manually in order to understand the complete monitoring pipeline.

---

# Laboratory Architecture

The monitoring architecture developed throughout this laboratory follows the model below:

```text
Linux Host
    │
    │ exposes system metrics
    ▼
Node Exporter
    │
    │ /metrics
    ▼
Prometheus
    │
    │ PromQL queries
    ▼
Grafana
    │
    ▼
Linux Infrastructure Dashboard
```

Each component has a specific responsibility.

| Component | Responsibility |
|---|---|
| Linux Host | Provides the operating system and infrastructure being monitored |
| Node Exporter | Exposes Linux operating system metrics |
| Prometheus | Collects and stores time-series metrics |
| PromQL | Queries and processes Prometheus metrics |
| Grafana | Visualizes and organizes the monitoring information |

This architecture represents one of the most common patterns used in modern infrastructure observability environments.

---

# Monitoring Pipeline

The complete data flow can be represented as:

```text
Linux Kernel / Operating System
              │
              ▼
        Node Exporter
              │
              │ HTTP /metrics
              ▼
          Prometheus
              │
              │ PromQL
              ▼
            Grafana
              │
              ▼
 Infrastructure Dashboard
```

The Linux operating system continuously exposes information about its internal state.

Node Exporter translates many of these operating system statistics into metrics compatible with Prometheus.

Prometheus periodically collects these metrics.

Grafana queries Prometheus and transforms the collected data into dashboards and operational visualizations.

---

# Laboratory Stages

This laboratory is divided into several stages.

```text
Stage 1
Architecture
Environment Preparation
Node Exporter
        │
        ▼
Stage 2
Prometheus
Metrics Collection
        │
        ▼
Stage 3
Grafana
Prometheus Data Source
        │
        ▼
Stage 4
PromQL
Dashboard Construction
        │
        ▼
Stage 5
Dashboard Refinement
Thresholds
Variables
Capacity Visualization
        │
        ▼
Stage 6
Dashboard Export
Documentation
Version Control
```

This progressive approach makes it possible to understand each layer of the monitoring architecture before introducing the next component.

---

# Stage 1 — Architecture, Environment Preparation and Node Exporter

## Stage Objective

The first stage establishes the foundation of the monitoring environment.

At the end of this stage, the environment should contain a Linux host running Node Exporter and exposing system metrics through an HTTP endpoint.

The expected architecture at this point is:

```text
Linux Host
    │
    ▼
Node Exporter
    │
    ▼
http://localhost:9100/metrics
```

Prometheus and Grafana are intentionally not introduced yet.

The objective is first to understand and validate the metric source.

---

# 1. Understanding the Linux Monitoring Target

Before building a dashboard, it is important to understand what is actually being monitored.

A Linux server continuously maintains information about:

- processor utilization;
- memory allocation;
- filesystem capacity;
- disk activity;
- network interfaces;
- system uptime;
- running processes;
- operating system resources.

Monitoring platforms need a mechanism to expose this information in a structured format.

For Prometheus-based environments, one of the most widely used tools for this purpose is **Node Exporter**.

---

# 2. What Is Node Exporter?

Node Exporter is a Prometheus exporter designed to expose hardware and operating system metrics from Unix-like systems.

It collects information from the operating system and exposes it through an HTTP endpoint.

By default, Node Exporter listens on:

```text
TCP port 9100
```

The metrics endpoint is normally available at:

```text
http://<linux-host>:9100/metrics
```

For a local installation:

```text
http://localhost:9100/metrics
```

The monitoring flow at this stage is therefore:

```text
Linux Operating System
          │
          ▼
    Node Exporter
          │
          ▼
      Port 9100
          │
          ▼
       /metrics
```

---

# 3. Understanding Exporters

Prometheus does not directly understand every operating system, database, network device or application.

Exporters provide a translation layer between a monitored system and Prometheus.

Examples include:

| Exporter | Typical Use |
|---|---|
| Node Exporter | Linux operating system metrics |
| Windows Exporter | Microsoft Windows metrics |
| SNMP Exporter | Network devices using SNMP |
| Blackbox Exporter | Endpoint and service probing |
| MySQL Exporter | MySQL database metrics |
| PostgreSQL Exporter | PostgreSQL database metrics |

In this laboratory, Node Exporter will provide the infrastructure metrics used by Prometheus and Grafana.

---

# 4. Metric Naming

Node Exporter exposes many metrics using names beginning with:

```text
node_
```

Examples include:

```text
node_cpu_seconds_total
node_memory_MemTotal_bytes
node_memory_MemAvailable_bytes
node_filesystem_size_bytes
node_filesystem_avail_bytes
node_network_receive_bytes_total
node_network_transmit_bytes_total
node_boot_time_seconds
```

These raw metrics will later become the foundation of the PromQL queries used by the Grafana dashboard.

For example:

```text
node_cpu_seconds_total
```

contains CPU time information.

Meanwhile:

```text
node_memory_MemAvailable_bytes
```

provides information about available system memory.

The dashboard will not simply display these raw values.

PromQL will later be used to transform them into operational indicators such as:

```text
CPU Usage (%)
Memory Usage (%)
Disk Usage (%)
Network Receive (bytes/sec)
Network Transmit (bytes/sec)
System Uptime
```

---

# 5. Environment Preparation

Before installing Node Exporter, verify that the Linux environment is available and operational.

The following commands can be used to inspect the system.

## Check the Operating System

Run:

```bash
cat /etc/os-release
```

Example output:

```text
NAME="Ubuntu"
VERSION="24.04 LTS"
ID=ubuntu
```

The exact output depends on the Linux distribution being used.

---

## Check the Kernel

Run:

```bash
uname -a
```

This command displays information about the Linux kernel and system architecture.

---

## Check the System Architecture

Run:

```bash
uname -m
```

Typical results include:

```text
x86_64
```

or:

```text
aarch64
```

The architecture is important because the correct Node Exporter binary must be downloaded for the system.

---

## Check the Hostname

Run:

```bash
hostname
```

The hostname will later help identify the monitored machine inside Prometheus and Grafana.

---

## Check the IP Address

Run:

```bash
hostname -I
```

Example:

```text
192.168.1.100
```

The IP address will become important when Prometheus is configured to collect metrics from this host.

---

# 6. Update the Linux Package Index

Before installing additional software, update the package index.

```bash
sudo apt update
```

Optionally, installed packages can also be upgraded:

```bash
sudo apt upgrade -y
```

> Updating the package index ensures that the operating system has current package metadata before additional tools are installed.

---

# 7. Install Required Utilities

Install the basic utilities that will be used during the laboratory:

```bash
sudo apt install -y curl wget tar
```

These utilities provide:

| Utility | Purpose |
|---|---|
| `curl` | Test HTTP endpoints |
| `wget` | Download files |
| `tar` | Extract compressed archives |

Verify the installation:

```bash
curl --version
```

and:

```bash
wget --version
```

---

# 8. Create a Node Exporter System User

Node Exporter does not require administrative privileges to operate.

For better service isolation, create a dedicated system user.

Run:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin node_exporter
```

Verify the user:

```bash
id node_exporter
```

The output should show the newly created system account.

Example:

```text
uid=xxx(node_exporter) gid=xxx(node_exporter) groups=xxx(node_exporter)
```

Using a dedicated service account follows the principle of least privilege and avoids running Node Exporter as the root user.

---

# 9. Download Node Exporter

Before downloading Node Exporter, identify the version that will be used in the laboratory.

The release should be obtained from the official Prometheus Node Exporter project.

After selecting the version, define it in the shell.

Example:

```bash
NODE_EXPORTER_VERSION="<VERSION>"
```

Then download the Linux AMD64 package:

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v${NODE_EXPORTER_VERSION}/node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz
```

> Replace `<VERSION>` with the Node Exporter version selected for the laboratory.

For ARM64 systems, the corresponding ARM64 release must be used instead.

---

# 10. Extract Node Exporter

Extract the downloaded archive:

```bash
tar xvf node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz
```

Enter the extracted directory:

```bash
cd node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64
```

List its contents:

```bash
ls -la
```

The Node Exporter executable should be visible:

```text
node_exporter
```

---

# 11. Install the Node Exporter Binary

Copy the binary to `/usr/local/bin`:

```bash
sudo cp node_exporter /usr/local/bin/
```

Assign ownership to the dedicated Node Exporter user:

```bash
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

Verify that the binary is available:

```bash
ls -l /usr/local/bin/node_exporter
```

Check the installed version:

```bash
/usr/local/bin/node_exporter --version
```

---

# 12. Create the Node Exporter systemd Service

Node Exporter should run automatically as a Linux service.

Create a new systemd unit:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Add the following configuration:

```ini
[Unit]
Description=Prometheus Node Exporter
Documentation=https://github.com/prometheus/node_exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Save the file.

---

# 13. Reload systemd

After creating the service file, reload the systemd configuration:

```bash
sudo systemctl daemon-reload
```

This allows systemd to recognize the new Node Exporter service.

---

# 14. Start Node Exporter

Start the service:

```bash
sudo systemctl start node_exporter
```

Verify its status:

```bash
sudo systemctl status node_exporter
```

The expected status is:

```text
Active: active (running)
```

---

# 15. Enable Node Exporter at Boot

Configure Node Exporter to start automatically when Linux boots:

```bash
sudo systemctl enable node_exporter
```

The service can now be checked with:

```bash
systemctl is-enabled node_exporter
```

Expected result:

```text
enabled
```

---

# 16. Validate the Listening Port

Node Exporter listens on TCP port `9100` by default.

Verify that the port is open locally:

```bash
ss -tulpn | grep 9100
```

A successful result should show Node Exporter listening on port:

```text
9100
```

The monitoring architecture is now:

```text
Linux Host
    │
    ▼
Node Exporter Service
    │
    ▼
TCP 9100
```

---

# 17. Test the Node Exporter HTTP Endpoint

Node Exporter provides a simple HTTP interface.

Using `curl`, run:

```bash
curl http://localhost:9100/
```

The response should contain the Node Exporter landing page.

The most important endpoint is:

```text
/metrics
```

Test it with:

```bash
curl http://localhost:9100/metrics
```

A large number of Prometheus-formatted metrics should be returned.

---

# 18. Inspect the Metrics Endpoint

Because the `/metrics` endpoint contains a large amount of information, the output can be filtered.

For example:

```bash
curl -s http://localhost:9100/metrics | head
```

To inspect CPU metrics:

```bash
curl -s http://localhost:9100/metrics | grep node_cpu
```

To inspect memory metrics:

```bash
curl -s http://localhost:9100/metrics | grep node_memory
```

To inspect filesystem metrics:

```bash
curl -s http://localhost:9100/metrics | grep node_filesystem
```

To inspect network metrics:

```bash
curl -s http://localhost:9100/metrics | grep node_network
```

This demonstrates that Node Exporter is already collecting real information from the Linux operating system.

---

# 19. Understanding the Prometheus Metric Format

A metric exposed by Node Exporter may look similar to:

```text
node_memory_MemTotal_bytes 1.6650815488e+10
```

This consists of:

```text
metric_name value
```

Metrics may also contain labels.

Example:

```text
node_cpu_seconds_total{cpu="0",mode="idle"} 12345.67
```

The labels provide additional dimensions.

In this example:

```text
cpu="0"
```

identifies the CPU core, while:

```text
mode="idle"
```

identifies the CPU state.

Later, PromQL will use these labels to calculate meaningful infrastructure indicators.

---

# 20. Important Metrics for This Laboratory

The following metrics will become especially important during the construction of the dashboard.

## CPU

```text
node_cpu_seconds_total
```

Used to calculate CPU utilization.

---

## System Load

```text
node_load1
node_load5
node_load15
```

These metrics represent the system load average over approximately:

- 1 minute;
- 5 minutes;
- 15 minutes.

---

## Memory

```text
node_memory_MemTotal_bytes
node_memory_MemAvailable_bytes
node_memory_MemFree_bytes
```

These metrics will be used to calculate memory utilization.

---

## Swap

```text
node_memory_SwapTotal_bytes
node_memory_SwapFree_bytes
```

These metrics provide information about swap usage.

---

## Filesystem

```text
node_filesystem_size_bytes
node_filesystem_avail_bytes
```

These metrics will be used to calculate filesystem capacity and utilization.

---

## Disk

Examples include:

```text
node_disk_read_bytes_total
node_disk_written_bytes_total
```

These metrics expose disk activity.

---

## Network

```text
node_network_receive_bytes_total
node_network_transmit_bytes_total
```

These metrics expose network traffic.

---

## System Boot Time

```text
node_boot_time_seconds
```

This metric will later be used to calculate system uptime.

---

# 21. Validate Specific Metrics

Check whether important metrics are being exposed.

## CPU

```bash
curl -s http://localhost:9100/metrics | grep '^node_cpu_seconds_total'
```

## Memory

```bash
curl -s http://localhost:9100/metrics | grep '^node_memory_MemTotal_bytes'
```

## Filesystem

```bash
curl -s http://localhost:9100/metrics | grep '^node_filesystem_size_bytes'
```

## Network

```bash
curl -s http://localhost:9100/metrics | grep '^node_network_receive_bytes_total'
```

## Boot Time

```bash
curl -s http://localhost:9100/metrics | grep '^node_boot_time_seconds'
```

If these commands return values, Node Exporter is successfully collecting the fundamental metrics required for this laboratory.

---

# 22. Access Node Exporter from a Web Browser

Node Exporter can also be accessed through a browser.

From the Linux host itself:

```text
http://localhost:9100
```

From another machine on the same network:

```text
http://<LINUX_HOST_IP>:9100
```

The metrics endpoint is:

```text
http://<LINUX_HOST_IP>:9100/metrics
```

Example:

```text
http://192.168.1.100:9100/metrics
```

> The exact IP address depends on the Linux environment.

If the page is accessible from another machine, the Node Exporter endpoint is ready to be consumed remotely by Prometheus.

---

# 23. Firewall Considerations

If Node Exporter works locally but cannot be accessed from another machine, verify the Linux firewall configuration.

Check the firewall status:

```bash
sudo ufw status
```

If UFW is enabled and remote access to Node Exporter is required, port `9100` may need to be allowed.

For a controlled laboratory network, an example rule is:

```bash
sudo ufw allow 9100/tcp
```

A more restrictive rule should be preferred when the Prometheus server IP address is known.

For example:

```bash
sudo ufw allow from <PROMETHEUS_SERVER_IP> to any port 9100 proto tcp
```

> Exposing monitoring endpoints broadly is not recommended in production environments. Firewall rules should restrict access to trusted monitoring systems whenever possible.

---

# 24. Troubleshooting Node Exporter

If Node Exporter does not start correctly, inspect the service status:

```bash
sudo systemctl status node_exporter
```

Check recent service logs:

```bash
sudo journalctl -u node_exporter
```

Display the latest log entries:

```bash
sudo journalctl -u node_exporter -n 50
```

Follow the logs in real time:

```bash
sudo journalctl -u node_exporter -f
```

Verify the executable:

```bash
ls -l /usr/local/bin/node_exporter
```

Verify the port:

```bash
ss -tulpn | grep 9100
```

Test the endpoint locally:

```bash
curl http://localhost:9100/metrics
```

This troubleshooting sequence helps isolate problems between:

```text
Binary
   ↓
systemd Service
   ↓
Process
   ↓
TCP Port
   ↓
HTTP Endpoint
   ↓
Metrics
```

---

# 25. Stage 1 Validation Checklist

Before proceeding to Prometheus, all items below should be validated.

- [ ] Linux environment is operational
- [ ] Operating system information was verified
- [ ] System architecture was identified
- [ ] Hostname was identified
- [ ] Host IP address was identified
- [ ] Required utilities were installed
- [ ] `node_exporter` system user was created
- [ ] Node Exporter binary was installed
- [ ] Node Exporter version was verified
- [ ] systemd service was created
- [ ] Node Exporter service is running
- [ ] Node Exporter is enabled at boot
- [ ] TCP port `9100` is listening
- [ ] Node Exporter HTTP endpoint is accessible
- [ ] `/metrics` endpoint returns metrics
- [ ] CPU metrics were validated
- [ ] Memory metrics were validated
- [ ] Filesystem metrics were validated
- [ ] Network metrics were validated
- [ ] Boot time metric was validated

---

# 26. Stage 1 Evidence

The following screenshots should be collected during the execution of this stage.

Suggested files:

```text
images/
├── 01-linux-system-information.png
├── 02-node-exporter-version.png
├── 03-node-exporter-service-running.png
├── 04-node-exporter-port-9100.png
├── 05-node-exporter-web-interface.png
└── 06-node-exporter-metrics.png
```

These screenshots provide evidence that the environment was configured and validated during the laboratory.

> Screenshots should document meaningful implementation and validation steps rather than every command executed.

---

# 27. Stage 1 Result

At this point, the first monitoring component is operational.

The environment now provides the following architecture:

```text
┌─────────────────────────────┐
│         Linux Host          │
│                             │
│  CPU                        │
│  Memory                     │
│  Filesystem                 │
│  Disk                       │
│  Network                    │
│  System Information         │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│        Node Exporter        │
│                             │
│       TCP Port 9100         │
│                             │
│         /metrics            │
└──────────────┬──────────────┘
               │
               ▼
       Prometheus Metrics
          Ready for
           Collection
```

The Linux host is now capable of exposing real infrastructure telemetry in Prometheus format.

---

# What Was Learned in Stage 1

This stage introduced the fundamental concepts behind Linux infrastructure metric collection.

The main concepts covered were:

- Linux infrastructure monitoring;
- exporters;
- Node Exporter;
- Prometheus metric format;
- metric names;
- metric labels;
- systemd services;
- service isolation;
- TCP monitoring endpoints;
- Linux CPU metrics;
- Linux memory metrics;
- filesystem metrics;
- disk metrics;
- network metrics;
- system uptime metrics;
- basic troubleshooting;
- monitoring endpoint validation.

Most importantly, this stage demonstrates that dashboards are not the beginning of an observability pipeline.

Before a visualization can exist, telemetry must first be:

```text
Generated
    ↓
Exposed
    ↓
Collected
    ↓
Queried
    ↓
Visualized
```

Stage 1 implemented the first two parts of this pipeline:

```text
Linux
   ↓
Generate System Information
   ↓
Node Exporter
   ↓
Expose Metrics
```

---

# Current Laboratory Status

```text
[✓] Architecture
[✓] Environment Preparation
[✓] Node Exporter

[ ] Prometheus
[ ] Metrics Collection
[ ] PromQL
[ ] Grafana Data Source
[ ] Dashboard Construction
[ ] CPU Visualization
[ ] Memory Visualization
[ ] Storage Visualization
[ ] Network Visualization
[ ] Availability
[ ] Capacity
[ ] Dashboard Variables
[ ] Thresholds
[ ] Dashboard Export
```

---

# Next Stage

## Stage 2 — Prometheus and Metrics Collection

The next stage will introduce Prometheus.

Prometheus will periodically collect the metrics currently exposed by Node Exporter.

The architecture will evolve from:

```text
Linux Host
    ↓
Node Exporter
    ↓
/metrics
```

to:

```text
Linux Host
    ↓
Node Exporter
    ↓
Prometheus
    ↓
Time-Series Metrics
```

The next stage will cover:

- Prometheus architecture;
- installation;
- `prometheus.yml`;
- scrape configuration;
- monitoring targets;
- Node Exporter integration;
- target health;
- Prometheus expression browser;
- basic metric queries;
- validation of collected infrastructure telemetry.

Once this stage is complete, the laboratory will have a functioning metrics collection pipeline ready for Grafana.

---

## Repository Navigation

[← Back to Repository](../../README.md)

[← LAB 01 — Grafana Dashboard Design Fundamentals](../01-grafana-dashboard-design-fundamentals/README.md)

[Roadmap](../../roadmap.md)

---

## Status

**LAB 02 — In Progress**

Current stage:

**Stage 1 — Architecture + Environment Preparation + Node Exporter**
