# Prometheus to OpenTelemetry migration reference

This directory documents the Prometheus metrics used in the kubernetes-mixin
project and their relationship to OpenTelemetry Collector native receivers and
semantic conventions. The goal is to support migrating — or partially
migrating — the mixin's dashboards and alerts from Prometheus scraping to
OpenTelemetry-native metric collection.

## Files

### `metrics-inventory.md`

A complete inventory of every Prometheus metric name and label set referenced
in the mixin's PromQL expressions, across alerts, recording rules, and Grafana
dashboards. Metrics are grouped by their source component:

- cAdvisor
- kube-state-metrics
- Kubelet (volume stats and operational metrics)
- node-exporter
- kube-apiserver
- kube-scheduler
- kube-controller-manager
- kube-proxy
- windows-exporter
- Recording rule output metrics (intermediate aggregates produced internally)

Use this file as the authoritative checklist of what must be covered when
replacing Prometheus scraping with an OpenTelemetry-based pipeline.

### `otel-mapping.md`

A mapping table and gap analysis covering three OpenTelemetry Collector
receivers — `kubeletstatsreceiver`, `k8sclusterreceiver`, and
`hostmetricsreceiver` — against the metrics in `metrics-inventory.md` and the
[OpenTelemetry Semantic Conventions for Kubernetes](https://opentelemetry.io/docs/specs/semconv/system/k8s-metrics/).

The file is structured in three parts:

1. **Coverage overview** — per-source summary of how much each receiver covers.
2. **Mapping tables** — for each Prometheus metric: the equivalent receiver
   metric, the equivalent semconv metric, and notes on semantic differences.
3. **Misalignment analysis** — cases where the actual receiver implementations
   diverge from the semconv specification, including naming conventions,
   attribute schemas, and metric semantics.

Key findings:

- CPU throttling, per-container disk I/O, and container-level network metrics
  have no native receiver equivalent and require the `prometheusreceiver`
  scraping cAdvisor.
- Node allocatable capacity (`kube_node_status_allocatable`) is defined in the
  semconv but implemented by no current receiver.
- All control-plane components (kube-apiserver, kube-scheduler,
  kube-controller-manager, kube-proxy) and kubelet operational metrics (PLEG,
  certificates, runtime operations) have no native receiver and must be scraped
  via `prometheusreceiver`.
- The `k8sclusterreceiver` and `kubeletstatsreceiver` use metric names and
  attribute names that differ systematically from the semconv specification
  (underscore vs. dot separators, suffix vs. infix resource-type segments,
  shorthand `direction` vs. namespaced `network.io.direction`).

### `migration.md`

A per-dashboard migration ranking that answers "which dashboards can be migrated
to OTel native receivers with the fewest mapping problems?".

Dashboards are ranked in four tiers:

1. **Tier 1** (`persistentvolumesusage`) — ~85% coverage; all volume stats panels
   map directly to `kubeletstatsreceiver`.
2. **Tier 2** (network dashboards) — ~55% coverage; byte-rate panels covered,
   packet/drop counters require `prometheusreceiver` or stubs.
3. **Tier 3** (`k8s-resources-*`) — ~30–40% coverage; CPU/memory/requests covered,
   CPU throttling and node-allocatable panels blocked.
4. **Tier 4** (kubelet, apiserver, scheduler, controller-manager, proxy, windows) —
   0% native coverage; all metrics require `prometheusreceiver`.

Also includes the recommended migration order and cross-cutting concerns
(attribute name mapping, workload topology processor, receiver-vs-semconv naming).
