# PromQL Queries — Linux Infrastructure Dashboard

This document contains the PromQL queries used throughout the  
**LAB 02 — Linux Infrastructure Dashboard**.

The metrics are collected from a Linux host using **Prometheus Node Exporter**, stored by **Prometheus**, queried using **PromQL**, and visualized in **Grafana**.

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

---

# 1. Node Exporter Availability

Checks whether Prometheus is successfully scraping the Node Exporter target.

```promql
up{job="node-exporter"}
```

Expected values:

| Value | Meaning |
|------:|---------|
| `1` | Node Exporter is reachable |
| `0` | Node Exporter is unavailable |

Recommended Grafana visualization:

```text
Stat
```

This query can be used to create a simple infrastructure availability indicator.

---

# 2. Node Exporter Availability by Environment

If the Prometheus configuration includes a custom label such as:

```yaml
environment: "lab"
```

the target can be filtered using:

```promql
up{
  job="node-exporter",
  environment="lab"
}
```

This becomes useful when multiple environments are monitored.

Examples:

```text
lab
development
staging
production
```

---

# 3. Node Exporter Availability by Hostname

If the Prometheus configuration includes:

```yaml
hostname: "linux-lab-01"
```

a specific host can be queried using:

```promql
up{
  job="node-exporter",
  hostname="linux-lab-01"
}
```

This approach becomes particularly useful when multiple Linux hosts are added to the environment.

---

# 4. System Uptime

Returns the amount of time the Linux host has been running since the last boot.

```promql
time() - node_boot_time_seconds{job="node-exporter"}
```

The result is returned in seconds.

Recommended Grafana visualization:

```text
Stat
```

Recommended unit:

```text
seconds (s)
```

Grafana can automatically convert the value to days, hours, minutes, and seconds.

---

# 5. System Boot Time

Returns the Unix timestamp corresponding to the last system boot.

```promql
node_boot_time_seconds{job="node-exporter"}
```

This metric is generally more useful when combined with:

```promql
time()
```

to calculate system uptime.

---

# 6. Number of Logical CPU Cores

Returns the number of logical processors detected on each Linux host.

```promql
count by (instance) (
  node_cpu_seconds_total{
    job="node-exporter",
    mode="idle"
  }
)
```

Recommended Grafana visualization:

```text
Stat
```

This value is particularly useful when interpreting Linux load averages.

Example:

```text
CPU cores: 4
Load average: 1.5
```

This indicates that CPU capacity is likely still available.

Another example:

```text
CPU cores: 4
Load average: 6.0
```

This may indicate processes waiting for CPU or other system resources.

---

# 7. CPU Usage Percentage

Calculates the overall CPU utilization percentage.

```promql
100 - (
  avg by (instance) (
    rate(
      node_cpu_seconds_total{
        job="node-exporter",
        mode="idle"
      }[5m]
    )
  ) * 100
)
```

The calculation works by measuring idle CPU time and subtracting it from 100%.

Example:

```text
CPU Idle = 72%
CPU Usage = 28%
```

Recommended visualization:

```text
Gauge
```

or:

```text
Time series
```

Recommended unit:

```text
Percent (0-100)
```

Example visualization thresholds:

```text
Normal   < 70%
Warning  70% - 85%
Critical > 85%
```

These thresholds are examples for laboratory visualization purposes and should not automatically be interpreted as production alert thresholds.

---

# 8. CPU Usage by Hostname

If the custom `hostname` label is configured:

```promql
100 - (
  avg by (hostname) (
    rate(
      node_cpu_seconds_total{
        job="node-exporter",
        mode="idle"
      }[5m]
    )
  ) * 100
)
```

This produces a more human-readable grouping than using the raw Prometheus `instance` label.

---

# 9. CPU Usage by Mode

Displays CPU activity grouped by Linux CPU mode.

```promql
sum by (instance, mode) (
  rate(
    node_cpu_seconds_total{
      job="node-exporter",
      mode!="idle"
    }[5m]
  )
) * 100
```

Typical CPU modes include:

```text
user
system
nice
iowait
irq
softirq
steal
```

Recommended visualization:

```text
Time series
```

This query helps distinguish whether CPU usage is being generated mainly by:

- user applications;
- kernel activity;
- I/O waiting;
- hardware interrupts;
- software interrupts;
- virtualization overhead.

---

# 10. CPU User Time

Measures CPU time spent executing user-space processes.

```promql
avg by (instance) (
  rate(
    node_cpu_seconds_total{
      job="node-exporter",
      mode="user"
    }[5m]
  )
) * 100
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
Percent (0-100)
```

---

# 11. CPU System Time

Measures CPU time spent executing kernel operations.

```promql
avg by (instance) (
  rate(
    node_cpu_seconds_total{
      job="node-exporter",
      mode="system"
    }[5m]
  )
) * 100
```

Recommended visualization:

```text
Time series
```

---

# 12. CPU I/O Wait

Measures the percentage of CPU time spent waiting for I/O operations.

```promql
avg by (instance) (
  rate(
    node_cpu_seconds_total{
      job="node-exporter",
      mode="iowait"
    }[5m]
  )
) * 100
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
Percent (0-100)
```

A sustained increase in I/O wait may indicate:

- slow storage;
- overloaded disks;
- storage contention;
- high read/write workloads.

---

# 13. CPU Steal Time

Measures CPU time involuntarily spent waiting for a virtual CPU while the hypervisor services another virtual machine.

```promql
avg by (instance) (
  rate(
    node_cpu_seconds_total{
      job="node-exporter",
      mode="steal"
    }[5m]
  )
) * 100
```

This metric is most relevant in virtualized environments.

Recommended visualization:

```text
Time series
```

---

# 14. Load Average — 1 Minute

Returns the Linux load average calculated over approximately one minute.

```promql
node_load1{job="node-exporter"}
```

Recommended visualization:

```text
Time series
```

---

# 15. Load Average — 5 Minutes

```promql
node_load5{job="node-exporter"}
```

Recommended visualization:

```text
Time series
```

---

# 16. Load Average — 15 Minutes

```promql
node_load15{job="node-exporter"}
```

Recommended visualization:

```text
Time series
```

The three load-average metrics can be shown together to understand whether system load is:

- increasing;
- decreasing;
- temporarily spiking;
- remaining persistently high.

---

# 17. Total Memory

Returns the total amount of physical RAM detected by Linux.

```promql
node_memory_MemTotal_bytes{job="node-exporter"}
```

Recommended visualization:

```text
Stat
```

Recommended Grafana unit:

```text
bytes (IEC)
```

---

# 18. Available Memory

Returns the amount of memory available to applications.

```promql
node_memory_MemAvailable_bytes{job="node-exporter"}
```

Recommended visualization:

```text
Stat
```

Recommended unit:

```text
bytes (IEC)
```

---

# 19. Used Memory

Calculates approximate memory currently in use.

```promql
node_memory_MemTotal_bytes{job="node-exporter"}
-
node_memory_MemAvailable_bytes{job="node-exporter"}
```

Recommended visualization:

```text
Stat
```

or:

```text
Time series
```

---

# 20. Memory Usage Percentage

Calculates RAM utilization as a percentage.

```promql
(
  1 -
  (
    node_memory_MemAvailable_bytes{job="node-exporter"}
    /
    node_memory_MemTotal_bytes{job="node-exporter"}
  )
) * 100
```

Recommended visualization:

```text
Gauge
```

Recommended unit:

```text
Percent (0-100)
```

Example visualization thresholds:

```text
Normal   < 70%
Warning  70% - 85%
Critical > 85%
```

---

# 21. Memory Cache

Returns memory used for filesystem cache.

```promql
node_memory_Cached_bytes{job="node-exporter"}
```

Recommended visualization:

```text
Time series
```

---

# 22. Memory Buffers

Returns memory allocated to kernel buffers.

```promql
node_memory_Buffers_bytes{job="node-exporter"}
```

Recommended visualization:

```text
Time series
```

---

# 23. Total Swap

Returns the total configured swap capacity.

```promql
node_memory_SwapTotal_bytes{job="node-exporter"}
```

Recommended visualization:

```text
Stat
```

---

# 24. Free Swap

```promql
node_memory_SwapFree_bytes{job="node-exporter"}
```

Recommended visualization:

```text
Stat
```

---

# 25. Used Swap

```promql
node_memory_SwapTotal_bytes{job="node-exporter"}
-
node_memory_SwapFree_bytes{job="node-exporter"}
```

Recommended visualization:

```text
Stat
```

or:

```text
Time series
```

---

# 26. Swap Usage Percentage

```promql
(
  (
    node_memory_SwapTotal_bytes{job="node-exporter"}
    -
    node_memory_SwapFree_bytes{job="node-exporter"}
  )
  /
  node_memory_SwapTotal_bytes{job="node-exporter"}
) * 100
```

Recommended visualization:

```text
Gauge
```

> Note:
>
> If the monitored Linux host does not have swap configured,
> `node_memory_SwapTotal_bytes` may be zero.
>
> In that case, the percentage calculation may return an invalid value.

---

# 27. Filesystem Total Capacity

Returns total capacity for mounted filesystems while excluding common temporary or container filesystems.

```promql
node_filesystem_size_bytes{
  job="node-exporter",
  fstype!~"tmpfs|overlay|squashfs"
}
```

Recommended visualization:

```text
Table
```

Recommended unit:

```text
bytes (IEC)
```

---

# 28. Filesystem Available Space

```promql
node_filesystem_avail_bytes{
  job="node-exporter",
  fstype!~"tmpfs|overlay|squashfs"
}
```

Recommended visualization:

```text
Table
```

---

# 29. Filesystem Used Space

```promql
node_filesystem_size_bytes{
  job="node-exporter",
  fstype!~"tmpfs|overlay|squashfs"
}
-
node_filesystem_avail_bytes{
  job="node-exporter",
  fstype!~"tmpfs|overlay|squashfs"
}
```

Recommended visualization:

```text
Table
```

---

# 30. Filesystem Usage Percentage

Calculates filesystem utilization.

```promql
(
  1 -
  (
    node_filesystem_avail_bytes{
      job="node-exporter",
      fstype!~"tmpfs|overlay|squashfs"
    }
    /
    node_filesystem_size_bytes{
      job="node-exporter",
      fstype!~"tmpfs|overlay|squashfs"
    }
  )
) * 100
```

Recommended visualization:

```text
Bar gauge
```

or:

```text
Gauge
```

Recommended unit:

```text
Percent (0-100)
```

Example visualization thresholds:

```text
Normal   < 70%
Warning  70% - 85%
Critical > 85%
```

---

# 31. Root Filesystem Usage

Displays utilization specifically for the Linux root filesystem.

```promql
(
  1 -
  (
    node_filesystem_avail_bytes{
      job="node-exporter",
      mountpoint="/",
      fstype!~"tmpfs|overlay|squashfs"
    }
    /
    node_filesystem_size_bytes{
      job="node-exporter",
      mountpoint="/",
      fstype!~"tmpfs|overlay|squashfs"
    }
  )
) * 100
```

Recommended visualization:

```text
Gauge
```

This can provide a useful high-level storage indicator.

---

# 32. Filesystem Inodes Usage Percentage

Storage exhaustion can occur because of disk capacity or because all available inodes have been consumed.

```promql
(
  1 -
  (
    node_filesystem_files_free{
      job="node-exporter",
      fstype!~"tmpfs|overlay|squashfs"
    }
    /
    node_filesystem_files{
      job="node-exporter",
      fstype!~"tmpfs|overlay|squashfs"
    }
  )
) * 100
```

Recommended visualization:

```text
Bar gauge
```

---

# 33. Disk Read Throughput

Measures bytes read from storage devices per second.

```promql
rate(
  node_disk_read_bytes_total{
    job="node-exporter",
    device!~"loop.*|ram.*"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
bytes/sec
```

---

# 34. Disk Write Throughput

Measures bytes written to storage devices per second.

```promql
rate(
  node_disk_written_bytes_total{
    job="node-exporter",
    device!~"loop.*|ram.*"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
bytes/sec
```

---

# 35. Disk Read Operations per Second

```promql
rate(
  node_disk_reads_completed_total{
    job="node-exporter",
    device!~"loop.*|ram.*"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
operations/sec
```

---

# 36. Disk Write Operations per Second

```promql
rate(
  node_disk_writes_completed_total{
    job="node-exporter",
    device!~"loop.*|ram.*"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
operations/sec
```

---

# 37. Disk I/O Utilization

Measures the percentage of time a block device spends performing I/O operations.

```promql
rate(
  node_disk_io_time_seconds_total{
    job="node-exporter",
    device!~"loop.*|ram.*"
  }[5m]
) * 100
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
Percent (0-100)
```

A sustained value near 100% may indicate storage saturation.

Interpretation depends on:

- storage technology;
- workload;
- device topology;
- RAID configuration;
- virtualization layer.

---

# 38. Disk Read Time

Measures accumulated time spent performing disk reads.

```promql
rate(
  node_disk_read_time_seconds_total{
    job="node-exporter",
    device!~"loop.*|ram.*"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

---

# 39. Disk Write Time

Measures accumulated time spent performing disk writes.

```promql
rate(
  node_disk_write_time_seconds_total{
    job="node-exporter",
    device!~"loop.*|ram.*"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

---

# 40. Network Receive Throughput

Measures incoming network traffic in bytes per second.

```promql
rate(
  node_network_receive_bytes_total{
    job="node-exporter",
    device!="lo"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
bytes/sec
```

---

# 41. Network Transmit Throughput

Measures outgoing network traffic in bytes per second.

```promql
rate(
  node_network_transmit_bytes_total{
    job="node-exporter",
    device!="lo"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
bytes/sec
```

---

# 42. Network Receive Traffic — Bits per Second

Network bandwidth is commonly represented in bits per second.

```promql
rate(
  node_network_receive_bytes_total{
    job="node-exporter",
    device!="lo"
  }[5m]
) * 8
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
bits/sec
```

---

# 43. Network Transmit Traffic — Bits per Second

```promql
rate(
  node_network_transmit_bytes_total{
    job="node-exporter",
    device!="lo"
  }[5m]
) * 8
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
bits/sec
```

---

# 44. Total Network Receive Traffic

Aggregates incoming traffic from all non-loopback interfaces.

```promql
sum by (instance) (
  rate(
    node_network_receive_bytes_total{
      job="node-exporter",
      device!="lo"
    }[5m]
  )
)
```

Recommended visualization:

```text
Time series
```

---

# 45. Total Network Transmit Traffic

Aggregates outgoing traffic from all non-loopback interfaces.

```promql
sum by (instance) (
  rate(
    node_network_transmit_bytes_total{
      job="node-exporter",
      device!="lo"
    }[5m]
  )
)
```

Recommended visualization:

```text
Time series
```

---

# 46. Network Receive Errors

Measures incoming network errors per second.

```promql
rate(
  node_network_receive_errs_total{
    job="node-exporter",
    device!="lo"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

Under normal conditions, this value should generally remain close to:

```text
0
```

Persistent errors may justify deeper investigation.

---

# 47. Network Transmit Errors

```promql
rate(
  node_network_transmit_errs_total{
    job="node-exporter",
    device!="lo"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

---

# 48. Network Receive Dropped Packets

```promql
rate(
  node_network_receive_drop_total{
    job="node-exporter",
    device!="lo"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

---

# 49. Network Transmit Dropped Packets

```promql
rate(
  node_network_transmit_drop_total{
    job="node-exporter",
    device!="lo"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

---

# 50. Network Receive Packets per Second

```promql
rate(
  node_network_receive_packets_total{
    job="node-exporter",
    device!="lo"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
packets/sec
```

---

# 51. Network Transmit Packets per Second

```promql
rate(
  node_network_transmit_packets_total{
    job="node-exporter",
    device!="lo"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

Recommended unit:

```text
packets/sec
```

---

# 52. Running Processes

Returns the number of processes currently running or runnable.

```promql
node_procs_running{job="node-exporter"}
```

Recommended visualization:

```text
Stat
```

or:

```text
Time series
```

---

# 53. Blocked Processes

Returns the number of processes blocked while waiting for I/O.

```promql
node_procs_blocked{job="node-exporter"}
```

Recommended visualization:

```text
Time series
```

A sustained increase may indicate:

- storage contention;
- slow disks;
- overloaded filesystems;
- I/O bottlenecks.

---

# 54. CPU Context Switches

Measures the rate of Linux context switches.

```promql
rate(
  node_context_switches_total{
    job="node-exporter"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

Context switches are normal, but large changes may help identify:

- high concurrency;
- scheduling pressure;
- excessive thread activity;
- heavy process workloads.

---

# 55. System Interrupts

Measures hardware and software interrupt activity.

```promql
rate(
  node_intr_total{
    job="node-exporter"
  }[5m]
)
```

Recommended visualization:

```text
Time series
```

---

# 56. Allocated File Descriptors

Returns the number of file descriptors currently allocated.

```promql
node_filefd_allocated{job="node-exporter"}
```

Recommended visualization:

```text
Stat
```

---

# 57. Maximum File Descriptors

Returns the configured maximum number of file descriptors.

```promql
node_filefd_maximum{job="node-exporter"}
```

Recommended visualization:

```text
Stat
```

---

# 58. File Descriptor Usage Percentage

```promql
(
  node_filefd_allocated{job="node-exporter"}
  /
  node_filefd_maximum{job="node-exporter"}
) * 100
```

Recommended visualization:

```text
Gauge
```

Recommended unit:

```text
Percent (0-100)
```

---

# 59. System Entropy

Returns the number of entropy bits currently available to the Linux kernel.

```promql
node_entropy_available_bits{job="node-exporter"}
```

Entropy is used by the operating system for cryptographic random number generation.

Very low entropy may affect workloads that depend heavily on secure random number generation.

---

# 60. Operating System Information

Node Exporter exposes Linux operating system and kernel information using:

```promql
node_uname_info{job="node-exporter"}
```

Typical labels include:

```text
domainname
machine
nodename
release
sysname
version
```

Example Grafana legend:

```text
{{nodename}} - {{sysname}} {{release}}
```

Recommended visualization:

```text
Table
```

---

# 61. Kernel Version

Kernel information is available through the `release` label of:

```promql
node_uname_info{job="node-exporter"}
```

Example:

```text
release="6.x.x-linux"
```

Recommended visualization:

```text
Table
```

or:

```text
Stat
```

---

# 62. Node Exporter Version

Returns information about the installed Node Exporter build.

```promql
node_exporter_build_info{job="node-exporter"}
```

Typical labels may include:

```text
branch
goversion
revision
version
```

Recommended visualization:

```text
Table
```

---

# 63. Prometheus Scrape Duration

Measures how long Prometheus takes to scrape the Node Exporter endpoint.

```promql
scrape_duration_seconds{job="node-exporter"}
```

Recommended visualization:

```text
Time series
```

An increasing value may indicate:

- host performance problems;
- network latency;
- exporter performance issues.

---

# 64. Prometheus Samples Scraped

Returns the number of metric samples collected during each scrape.

```promql
scrape_samples_scraped{job="node-exporter"}
```

Recommended visualization:

```text
Time series
```

This metric can help validate that Node Exporter is exposing a realistic set of infrastructure metrics.

---

# 65. Prometheus Scrape Success

The simplest health check remains:

```promql
up{job="node-exporter"}
```

Prometheus automatically generates the `up` metric for every configured target.

---

# PromQL Functions Used in This Laboratory

The queries above introduce several fundamental PromQL concepts.

---

## `rate()`

Example:

```promql
rate(
  node_network_receive_bytes_total[5m]
)
```

`rate()` calculates the average per-second rate of increase of a counter over a selected time range.

It is commonly used with monotonically increasing metrics such as:

```text
node_network_receive_bytes_total
node_disk_read_bytes_total
node_context_switches_total
```

The expression:

```promql
[5m]
```

means that Prometheus should use samples collected during the previous five minutes.

---

## `avg`

Example:

```promql
avg(
  rate(
    node_cpu_seconds_total{
      mode="idle"
    }[5m]
  )
)
```

`avg` calculates the average value across matching time series.

---

## `sum`

Example:

```promql
sum(
  rate(
    node_network_receive_bytes_total[5m]
  )
)
```

`sum` combines the values of multiple matching time series.

---

## `count`

Example:

```promql
count(
  node_cpu_seconds_total{
    mode="idle"
  }
)
```

`count` returns the number of matching time series.

It can be useful for determining the number of logical CPU cores.

---

## `by()`

Example:

```promql
avg by (instance) (
  rate(
    node_cpu_seconds_total{
      mode="idle"
    }[5m]
  )
)
```

`by()` preserves selected labels during aggregation.

Common labels in this laboratory include:

```text
instance
hostname
device
mode
mountpoint
```

---

# Prometheus Metric Types

Understanding metric types is important when writing PromQL.

---

## Counter

A counter normally increases continuously until a process or machine restarts.

Examples:

```text
node_network_receive_bytes_total
node_disk_reads_completed_total
node_context_switches_total
```

Counters are frequently queried using:

```promql
rate()
```

or:

```promql
increase()
```

---

## Gauge

A gauge can increase or decrease.

Examples:

```text
node_memory_MemAvailable_bytes
node_load1
node_procs_running
node_filesystem_avail_bytes
```

Gauges are usually queried directly or used in mathematical expressions.

---

# Why Use a Five-Minute Window?

Most counter-based queries in this laboratory initially use:

```promql
[5m]
```

For example:

```promql
rate(
  node_network_receive_bytes_total[5m]
)
```

A five-minute rate window provides a useful compromise between:

- responsiveness;
- stability;
- readability;
- reduced metric noise.

A shorter interval such as:

```promql
[1m]
```

reacts faster but may produce noisier graphs.

A longer interval such as:

```promql
[15m]
```

produces smoother graphs but reacts more slowly to infrastructure changes.

---

# Grafana `$__rate_interval`

When the dashboard is integrated more deeply with Grafana, fixed rate intervals can be replaced by Grafana's dynamic variable:

```text
$__rate_interval
```

For example:

```promql
100 - (
  avg by (instance) (
    rate(
      node_cpu_seconds_total{
        job="node-exporter",
        mode="idle"
      }[$__rate_interval]
    )
  ) * 100
)
```

Grafana dynamically adjusts this interval according to:

- dashboard time range;
- panel resolution;
- Prometheus scrape interval.

This is particularly useful when switching between ranges such as:

```text
Last 15 minutes
Last 1 hour
Last 6 hours
Last 24 hours
Last 7 days
```

---

# Future Grafana `$instance` Variable

A later version of the dashboard may introduce a Grafana variable named:

```text
$instance
```

A query could then become:

```promql
100 - (
  avg by (instance) (
    rate(
      node_cpu_seconds_total{
        job="node-exporter",
        mode="idle",
        instance=~"$instance"
      }[$__rate_interval]
    )
  ) * 100
)
```

This allows users to select which Linux host should be displayed.

---

# Future Grafana `$hostname` Variable

Because this laboratory uses a custom Prometheus label called:

```text
hostname
```

we may also create a Grafana variable named:

```text
$hostname
```

Example:

```promql
100 - (
  avg by (hostname) (
    rate(
      node_cpu_seconds_total{
        job="node-exporter",
        mode="idle",
        hostname=~"$hostname"
      }[$__rate_interval]
    )
  ) * 100
)
```

This creates a more human-readable host selector than raw `IP-address:port` values.

---

# Recommended Initial Dashboard Queries

Not every query documented in this file needs to be used immediately.

The first operational version of the Linux Infrastructure Dashboard can focus on the most important infrastructure indicators.

| Dashboard Panel | Query / Metric |
|---|---|
| Node Status | `up{job="node-exporter"}` |
| Uptime | `time() - node_boot_time_seconds` |
| CPU Usage | `node_cpu_seconds_total` |
| CPU I/O Wait | `node_cpu_seconds_total{mode="iowait"}` |
| CPU Cores | `node_cpu_seconds_total` |
| Load Average | `node_load1`, `node_load5`, `node_load15` |
| Memory Usage | `node_memory_MemAvailable_bytes` |
| Swap Usage | `node_memory_Swap*` |
| Root Filesystem | `node_filesystem_*` |
| Disk Read | `node_disk_read_bytes_total` |
| Disk Write | `node_disk_written_bytes_total` |
| Disk I/O | `node_disk_io_time_seconds_total` |
| Network RX | `node_network_receive_bytes_total` |
| Network TX | `node_network_transmit_bytes_total` |
| Network Errors | `node_network_*_errs_total` |
| Running Processes | `node_procs_running` |
| Blocked Processes | `node_procs_blocked` |

---

# Suggested Dashboard Sections

The queries documented here can later be organized into Grafana dashboard sections.

---

## System Overview

Possible panels:

```text
Node Status
Hostname
Kernel Version
Uptime
CPU Cores
```

---

## CPU

Possible panels:

```text
CPU Usage
CPU User
CPU System
CPU I/O Wait
Load Average
```

---

## Memory

Possible panels:

```text
Memory Usage
Memory Available
Memory Cache
Swap Usage
```

---

## Storage

Possible panels:

```text
Filesystem Usage
Root Filesystem Usage
Inode Usage
Disk Read Throughput
Disk Write Throughput
Disk I/O Utilization
```

---

## Network

Possible panels:

```text
Network RX
Network TX
Packets RX
Packets TX
Receive Errors
Transmit Errors
Dropped Packets
```

---

## Processes and System

Possible panels:

```text
Running Processes
Blocked Processes
Context Switches
System Interrupts
File Descriptor Usage
```

---

# Query Validation

Before using a query in Grafana, it can be tested directly in the Prometheus web interface.

Prometheus normally listens on:

```text
http://localhost:9090
```

The basic validation flow is:

```text
Node Exporter
     │
     ▼
Prometheus Target = UP
     │
     ▼
Prometheus Query
     │
     ▼
Valid Time Series
     │
     ▼
Grafana Panel
```

A useful first test is:

```promql
up{job="node-exporter"}
```

Expected result:

```text
1
```

Then test:

```promql
node_uname_info{job="node-exporter"}
```

and:

```promql
node_memory_MemTotal_bytes{job="node-exporter"}
```

If these metrics return valid values, Node Exporter and Prometheus are communicating correctly.

---

# Query Troubleshooting

If:

```promql
up{job="node-exporter"}
```

returns:

```text
0
```

Prometheus knows about the target but cannot successfully scrape it.

Possible causes include:

- Node Exporter is stopped;
- incorrect TCP port;
- incorrect target address;
- firewall rules;
- container networking;
- incorrect `prometheus.yml` configuration.

If the query returns no data at all, verify whether the job name defined in:

```text
prometheus.yml
```

matches:

```text
node-exporter
```

---

# Metric Discovery

Node Exporter exposes raw metrics through:

```text
http://localhost:9100/metrics
```

This endpoint can be used to inspect available metric names.

Examples include:

```text
node_cpu_seconds_total
node_memory_MemTotal_bytes
node_filesystem_size_bytes
node_disk_read_bytes_total
node_network_receive_bytes_total
node_load1
node_boot_time_seconds
```

Prometheus stores these metrics as time series and makes them queryable using PromQL.

---

# LAB 02 Query Philosophy

The purpose of this file is not only to store copy-and-paste queries.

It also documents the relationship between:

```text
Linux Resource
      │
      ▼
Node Exporter Metric
      │
      ▼
PromQL Expression
      │
      ▼
Operational Indicator
      │
      ▼
Grafana Visualization
```

For example:

```text
Linux CPU
    │
    ▼
node_cpu_seconds_total
    │
    ▼
rate(...)
    │
    ▼
CPU Usage %
    │
    ▼
Grafana Gauge / Time Series
```

And:

```text
Linux RAM
    │
    ▼
node_memory_MemAvailable_bytes
    │
    ▼
PromQL Calculation
    │
    ▼
Memory Usage %
    │
    ▼
Grafana Gauge
```

This approach makes the dashboard easier to:

- understand;
- reproduce;
- troubleshoot;
- maintain;
- extend.

---

# Future Improvements

Later stages of LAB 02 may introduce:

- multiple Linux hosts;
- Grafana dashboard variables;
- dynamic `$instance` filtering;
- dynamic `$hostname` filtering;
- `$__rate_interval`;
- recording rules;
- alerting rules;
- CPU saturation indicators;
- filesystem inode monitoring;
- storage latency analysis;
- network packet analysis;
- capacity planning;
- predictive monitoring;
- Grafana alerts;
- Prometheus alert rules;
- Alertmanager integration.

---

# Future Recording Rule Example

A later Prometheus configuration could pre-calculate CPU utilization.

Example:

```yaml
groups:
  - name: linux-infrastructure
    rules:
      - record: instance:node_cpu_utilization:ratio
        expr: |
          1 - avg by (instance) (
            rate(
              node_cpu_seconds_total{
                mode="idle"
              }[5m]
            )
          )
```

Instead of repeating a large PromQL expression, Grafana could then query:

```promql
instance:node_cpu_utilization:ratio
```

Recording rules should be introduced only after the fundamental PromQL concepts are understood.

---

# Future Alert Example

Node Exporter availability could eventually be monitored using:

```promql
up{job="node-exporter"} == 0
```

A future alert rule could use this condition to detect when a monitored Linux host becomes unavailable.

Other possible alert conditions include:

```text
High CPU usage
High memory usage
Filesystem capacity exhaustion
High I/O wait
Node Exporter unavailable
High file descriptor utilization
```

These conditions should be treated separately from the initial dashboard construction.

---

# References

The official documentation should be treated as the authoritative source for Prometheus, PromQL, Node Exporter, and Grafana behavior.

Recommended references:

- Prometheus Documentation
- PromQL Documentation
- Prometheus Node Exporter
- Grafana Documentation

---

# LAB 02 — Linux Infrastructure Dashboard

Monitoring stack:

```text
Linux
  +
Node Exporter
  +
Prometheus
  +
PromQL
  +
Grafana
```

Repository objective:

> Build a reproducible Linux infrastructure monitoring dashboard while documenting not only the visualizations, but also the metrics, queries, monitoring architecture, and reasoning behind each panel.
