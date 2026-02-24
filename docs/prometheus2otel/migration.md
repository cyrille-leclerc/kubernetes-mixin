# Dashboard migration ranking: Prometheus → OpenTelemetry

This document ranks the kubernetes-mixin Grafana dashboards by how completely
they can be migrated to OTel Collector native receivers
(`kubeletstatsreceiver`, `k8sclusterreceiver`, `hostmetricsreceiver`),
without falling back to `prometheusreceiver`.

Refer to [otel-mapping.md](otel-mapping.md) for the full metric-by-metric
mapping and the [metrics-inventory.md](metrics-inventory.md) for the
complete inventory of Prometheus metrics.

---

## Summary table

| Dashboard | Source file | Tier | OTel coverage | Blocking gaps |
|-----------|-------------|------|---------------|---------------|
| Persistent Volumes | `persistentvolumesusage.libsonnet` | 1 | ~85% | `kube_persistentvolume_status_phase` (PV state info panel) |
| Cluster network | `network.libsonnet` (cluster-total) | 2 | ~55% | Packet & drop counters |
| Namespace network by pod | `network.libsonnet` (namespace-by-pod) | 2 | ~55% | Packet & drop counters |
| Pod network | `network.libsonnet` (pod-total) | 2 | ~55% | Packet & drop counters |
| Workload network | `network.libsonnet` (workload-total) | 2 | ~55% | Packet & drop counters |
| Namespace network by workload | `network.libsonnet` (namespace-by-workload) | 2 | ~55% | Packet & drop counters |
| Resources — cluster | `resources.libsonnet` | 3 | ~35% | Throttling, disk I/O, node allocatable |
| Resources — namespace | `resources.libsonnet` | 3 | ~35% | Throttling, disk I/O, node allocatable |
| Resources — node | `resources.libsonnet` | 3 | ~35% | Throttling, disk I/O, node allocatable |
| Resources — pod | `resources.libsonnet` | 3 | ~40% | Throttling, disk I/O |
| Resources — workload | `resources.libsonnet` | 3 | ~35% | Throttling, disk I/O, node allocatable |
| Resources — workloads namespace | `resources.libsonnet` | 3 | ~35% | Throttling, disk I/O, node allocatable |
| Kubelet | `kubelet.libsonnet` | 4 | 0% | All metrics are kubelet operational (no native receiver) |
| API server | `apiserver.libsonnet` | 4 | 0% | All metrics are kube-apiserver (no native receiver) |
| Scheduler | `scheduler.libsonnet` | 4 | 0% | All metrics are kube-scheduler (no native receiver) |
| Controller manager | `controller-manager.libsonnet` | 4 | 0% | All metrics are kube-controller-manager (no native receiver) |
| Proxy | `proxy.libsonnet` | 4 | 0% | All metrics are kube-proxy (no native receiver) |
| Windows resources | `windows.libsonnet` | 4 | 0% | No OTel native equivalent for windows-exporter |

---

## Tier 1 — Migrate now (~85% coverage)

### `persistentvolumesusage` (Persistent Volumes Usage)

**Coverage: ~85%**

The volume stats panels are fully covered by `kubeletstatsreceiver`.
Only the PV lifecycle status info panel uses `kube_persistentvolume_status_phase`
which has no native receiver equivalent.

| Prometheus metric | OTel receiver metric | Status |
|-------------------|---------------------|--------|
| `kubelet_volume_stats_capacity_bytes` | `k8s.volume.capacity` | Covered — attribute rename only |
| `kubelet_volume_stats_available_bytes` | `k8s.volume.available` | Covered — attribute rename only |
| `kubelet_volume_stats_used_bytes` | `k8s.pod.volume.usage` (optional) | Covered — enable optional metric |
| `kubelet_volume_stats_inodes` | `k8s.volume.inodes` | Covered — attribute rename only |
| `kubelet_volume_stats_inodes_free` | `k8s.volume.inodes.free` | Covered — attribute rename only |
| `kubelet_volume_stats_inodes_used` | `k8s.volume.inodes.used` | Covered — attribute rename only |
| `kube_persistentvolume_status_phase` | **No equivalent** | **Gap** — drop or replace with `prometheusreceiver` |
| `kube_persistentvolumeclaim_access_mode` | **No equivalent** | **Gap** — label join only, used for info enrichment |
| `kube_persistentvolumeclaim_labels` | Resource attributes via `k8sattributesprocessor` | Covered via processor |

**Required changes:**
- Rename `kubelet_volume_stats_*` → `k8s.volume.*` (and enable `k8s.pod.volume.usage`).
- Replace label selectors (`namespace`, `persistentvolumeclaim`) with OTel resource
  attributes (`k8s.namespace.name`, `k8s.volume.name`).
- Drop the PV phase status panel or keep it behind `prometheusreceiver`.

---

## Tier 2 — Migrate with minor gaps (~55% coverage)

### Network dashboards (cluster-total, namespace-by-pod, pod-total, workload-total, namespace-by-workload)

**Coverage: ~55%**

Byte-rate panels (the majority of panels) map directly to `k8s.pod.network.io`
from `kubeletstatsreceiver`. Packet and drop counters have no native receiver
equivalent and require either `prometheusreceiver` scraping cAdvisor or a
warning panel.

| Prometheus metric | OTel receiver metric | Status |
|-------------------|---------------------|--------|
| `container_network_receive_bytes_total` | `k8s.pod.network.io{direction=receive}` | Covered — direction attribute consolidation |
| `container_network_transmit_bytes_total` | `k8s.pod.network.io{direction=transmit}` | Covered — direction attribute consolidation |
| `container_network_receive_packets_total` | **No equivalent** | **Gap** — cAdvisor-only |
| `container_network_transmit_packets_total` | **No equivalent** | **Gap** — cAdvisor-only |
| `container_network_receive_packets_dropped_total` | **No equivalent** | **Gap** — cAdvisor-only |
| `container_network_transmit_packets_dropped_total` | **No equivalent** | **Gap** — cAdvisor-only |

**Required changes:**
- Replace `container_network_receive_bytes_total` + `container_network_transmit_bytes_total`
  with a single `k8s.pod.network.io` query filtered by `direction` attribute.
- Note: the receiver uses attribute name `direction`; the OTel semconv specifies
  `network.io.direction` — match the receiver, not the spec (see misalignment §6.2 in
  [otel-mapping.md](otel-mapping.md)).
- Replace `pod` / `namespace` Prometheus label filters with OTel resource attributes
  `k8s.pod.name` / `k8s.namespace.name`.
- Workload-topology joins (recording rules derived from `kube_pod_owner`) must be
  replaced by `k8sattributesprocessor` enriching metrics with `k8s.deployment.name`,
  `k8s.replicaset.name`, etc.
- Insert warning text panels for packet and drop counter panels.

---

## Tier 3 — Migrate with significant gaps (~30–40% coverage)

### `k8s-resources-*` dashboards (cluster, namespace, node, pod, workload, workloads-namespace)

**Coverage: ~30–40%**

CPU and memory usage panels are partially covered. CPU throttling, per-container
disk I/O, and node allocatable capacity are the main blockers.

| Prometheus metric | OTel receiver metric | Status |
|-------------------|---------------------|--------|
| `container_cpu_usage_seconds_total` | `container.cpu.time` / `k8s.pod.cpu.time` | Covered — scope split (container vs pod) |
| `container_memory_working_set_bytes` | `container.memory.working_set` | Covered — direct equivalent |
| `container_memory_rss` | `container.memory.rss` | Covered — direct equivalent |
| `kube_pod_container_resource_requests` | `k8s.container.cpu_request` / `k8s.container.memory_request` | Covered — split by resource type; note `_` vs `.` naming vs semconv |
| `kube_pod_container_resource_limits` | `k8s.container.cpu_limit` / `k8s.container.memory_limit` | Covered — same split |
| `container_cpu_cfs_throttled_periods_total` | **No equivalent** | **Gap** — critical: CPU throttle detection missing |
| `container_cpu_cfs_periods_total` | **No equivalent** | **Gap** — required denominator for throttle ratio |
| `container_fs_reads_bytes_total` | **No equivalent** | **Gap** — cAdvisor-only disk I/O |
| `container_fs_writes_bytes_total` | **No equivalent** | **Gap** — cAdvisor-only disk I/O |
| `kube_node_status_allocatable` | **No equivalent** | **Gap** — critical: namespace/node capacity calculations blocked |
| `kube_pod_status_phase` | `k8s.pod.phase` (integer encoding) | Partial — semantic mismatch: integer vs phase-as-attribute (see §6.1 in otel-mapping.md) |
| `kube_deployment_spec_replicas` | `k8s.deployment.desired` | Covered — name differs from semconv but receiver has it |
| `kube_deployment_status_replicas_available` | `k8s.deployment.available` | Covered |
| `kube_daemonset_status_*` | `k8s.daemonset.*` | Covered — name differs from semconv (suffix vs infix) |
| `kube_statefulset_*` | `k8s.statefulset.*` | Covered — name differs from semconv |

**Required changes:**
- Rewrite CPU usage queries: `container_cpu_usage_seconds_total` → `container.cpu.time`
  (rate of counter); filter by `k8s.container.name` + `k8s.pod.name` resource attributes.
- Rewrite memory queries: `container_memory_working_set_bytes` → `container.memory.working_set`.
- Rewrite resource requests/limits: `kube_pod_container_resource_requests{resource="cpu"}`
  → `k8s.container.cpu_request` (two separate metrics instead of one with `resource` label).
- Replace `kube_node_status_allocatable` usage with `prometheusreceiver` scraped KSM,
  or insert warning panels for all capacity-ratio panels.
- Insert warning panels for throttle panels and disk I/O panels.
- Replace `kube_pod_owner`-based recording rules with `k8sattributesprocessor` resource
  attribute joins.

---

## Tier 4 — No native OTel coverage (0%)

These dashboards are entirely composed of metrics from sources that have no native
OTel Collector receiver. They require `prometheusreceiver` scraping the respective
component endpoints. No panels can be converted without a `prometheusreceiver` fallback.

### `kubelet` — Kubelet operational metrics

All 24 panels use kubelet `/metrics` endpoint:
PLEG latency, certificate TTL, eviction counters, runtime operation duration,
cgroup manager, pod worker latency, storage operations. No native receiver covers
any of these. See `dashboards/kubelet-otel.libsonnet` for the annotated version
with warning panels on every panel.

### `apiserver` — Kubernetes API server

All panels use `apiserver_*` metrics scraped from the kube-apiserver `/metrics`
endpoint: request rates, SLO-based burn-rate alerting, latency histograms,
client certificate expiration, aggregator availability.

### `scheduler` — kube-scheduler

All panels use `scheduler_*` metrics: scheduling attempt durations, preemption
events, pod scheduling SLI histograms. Entirely dependent on `prometheusreceiver`.

### `controller-manager` — kube-controller-manager

All panels use `workqueue_*` metrics: depth, adds, latency per workqueue name.
Entirely dependent on `prometheusreceiver`.

### `proxy` — kube-proxy

All panels use `kubeproxy_*` metrics: sync proxy rules duration,
network programming latency, rule sync failure counters.
Entirely dependent on `prometheusreceiver`.

### `windows` — Windows node resources

All panels use `windows_*` metrics from windows-exporter. No OTel native
equivalent for Windows system metrics currently exists in the Collector.

---

## Recommended migration order

1. **`persistentvolumesusage`** — highest coverage, isolated metric set, clear receiver
   mapping for all main panels. Drop or stub the PV phase panel.

2. **Network dashboards** (`cluster-total`, `namespace-by-pod`, `pod-total`,
   `workload-total`, `namespace-by-workload`) — most panels covered; only packet/drop
   panels need stubs. Migrate together since they share the same metric patterns.

3. **`k8s-resources-pod`** — pod-scoped dashboard avoids the node-allocatable gap that
   blocks the cluster/namespace/node resource dashboards; slightly higher coverage.

4. **`k8s-resources-namespace`, `k8s-resources-workload`, `k8s-resources-workloads-namespace`**
   — share the workload topology pattern; migrate together once `k8sattributesprocessor` is
   confirmed to carry `k8s.deployment.name` through to container metrics.

5. **`k8s-resources-cluster`, `k8s-resources-node`** — require the node-allocatable gap to
   be resolved (either via `prometheusreceiver` + KSM or when a receiver implements
   `k8s.node.*.allocatable` from the semconv).

6. **All Tier 4 dashboards** — defer until OTel native receivers exist for control-plane
   metrics, or accept `prometheusreceiver` as a permanent part of the pipeline for those
   components.

---

## Cross-cutting migration concerns

These issues apply to every tier and must be addressed once at the pipeline level:

### Attribute name mapping (Prometheus labels → OTel resource attributes)

| Prometheus label | OTel resource attribute |
|-----------------|------------------------|
| `namespace` | `k8s.namespace.name` |
| `pod` | `k8s.pod.name` |
| `container` | `k8s.container.name` |
| `node` | `k8s.node.name` |
| `persistentvolumeclaim` | `k8s.volume.name` (kubeletstatsreceiver) |

OTel resource attributes are attached to the metric series, not carried as metric
labels in the traditional Prometheus sense. Grafana dashboard queries must use the
appropriate attribute path for the backend (e.g., Tempo/Mimir label selectors vs
OTLP-native query engines).

### Workload topology (`kube_pod_owner` replacement)

The mixin's recording rules derive `workload` and `workload_type` labels by joining
`kube_pod_owner` with workload metrics. In OTel pipelines, this join is replaced by
the `k8sattributesprocessor`, which enriches all metrics with:
- `k8s.deployment.name`
- `k8s.replicaset.name`
- `k8s.statefulset.name`
- `k8s.daemonset.name`
- `k8s.job.name`
- `k8s.cronjob.name`

All dashboard panels that filter or group by `workload` label must be rewritten to
use these resource attributes instead.

### Receiver naming vs OTel semconv

The actual receiver metric names differ from the OTel semantic conventions in
systematic ways (see [otel-mapping.md §6](otel-mapping.md)). Dashboard queries
must target the **receiver metric names** as emitted, not the semconv names:

- Use `k8s.container.cpu_request` (receiver), not `k8s.container.cpu.request` (semconv)
- Use `k8s.deployment.desired` (receiver), not `k8s.deployment.pod.desired` (semconv)
- Use `direction` attribute (receiver), not `network.io.direction` (semconv)
