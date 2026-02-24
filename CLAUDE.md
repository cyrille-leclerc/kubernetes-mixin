# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Generate outputs

```sh
make generate          # Generate prometheus_alerts.yaml, prometheus_rules.yaml, and dashboards_out/
```

### Format

```sh
make fmt               # Format all jsonnet and markdown files in-place
make jsonnet-fmt       # Format jsonnet/libsonnet only
make markdownfmt       # Format markdown only
```

### Lint

```sh
make lint              # Run all linters: jsonnet, alerts, dashboards, vale, pint
make alerts-lint       # Validate alert/rule YAML with promtool
make dashboards-lint   # Lint generated Grafana dashboards
make pint-lint         # Lint PromQL expressions with pint
```

### Test

```sh
make test              # Run all promtool unit tests against tests/*.yaml
```

Run a single test file:

```sh
./tmp/bin/promtool test rules tests/resource_alerts-test.yaml
```

### Full CI check

```sh
make all               # fmt + generate + lint + test
```

### Install tooling

All tools (jsonnet, jb, promtool, pint, dashboard-linter, etc.) are installed locally into `tmp/bin/`:

```sh
cd scripts && go list -e -mod=mod -tags tools -f '{{ range .Imports }}{{ printf "%s\n" .}}{{end}}' ./ | xargs -tI % go build -mod=mod -o ../tmp/bin %
```

Or simply run any `make` target; tools are auto-installed as prerequisites.

### Local dev cluster (kind + LGTM stack)

```sh
make dev               # Create kind cluster and deploy LGTM stack
make dev-port-forward  # Forward ports (Grafana :3000, Prometheus :9090)
make dev-reload        # Regenerate and redeploy alerts/rules to the running cluster
make dev-down          # Delete the kind cluster
```

## Architecture

This is a **Prometheus Monitoring Mixin** — a jsonnet-based package that bundles Grafana dashboards, Prometheus alerts, and Prometheus recording rules for Kubernetes.

### Entry point

`mixin.libsonnet` — imports the three top-level modules and the shared config:

```
mixin.libsonnet
├── alerts/alerts.libsonnet       → prometheusAlerts
├── dashboards/dashboards.libsonnet → grafanaDashboards
├── rules/rules.libsonnet         → prometheusRules
└── config.libsonnet              → _config (all tunable defaults)
```

### Configuration (`config.libsonnet`)

All user-facing tunables live in `_config`. Key fields:

- **Job selectors** (`cadvisorSelector`, `kubeletSelector`, `kubeStateMetricsSelector`, etc.) — inserted verbatim into PromQL `{}`.
- **`showMultiCluster` / `clusterLabel`** — opt-in multi-cluster support.
- **`grafanaK8s`** — dashboard name prefix, tags, link prefix, refresh interval.
- **`SLOs.apiserver`** — multi-burn-rate SLO windows for the API server.
- **`common_join_labels` / `*_join_labels`** — labels joined from `kube_*_labels` metrics.
- **`grafana72` / `grafanaIntervalVar`** — controls use of `$__rate_interval` vs `$__interval`.

Override `_config` in a consumer `mixin.libsonnet` using jsonnet `+::` merging.

### Alerts (`alerts/`)

Each file covers one Kubernetes component:

| File | Covers |
|------|--------|
| `apps_alerts.libsonnet` | Deployments, DaemonSets, StatefulSets, Jobs, HPA |
| `resource_alerts.libsonnet` | CPU/memory quota, resource requests |
| `storage_alerts.libsonnet` | PersistentVolumes |
| `system_alerts.libsonnet` | Node conditions, OOM, clock skew |
| `kubelet.libsonnet` | Kubelet health |
| `kube_apiserver.libsonnet` | API server availability/errors |
| `kube_scheduler.libsonnet` | Scheduler errors |
| `kube_controller_manager.libsonnet` | Controller manager errors |
| `kube_proxy.libsonnet` | Kube-proxy sync |

`lib/absent_alert.libsonnet` provides the reusable `*Down` alert template (component disappeared from Prometheus scrape targets).

### Rules (`rules/`)

Recording rules pre-aggregate expensive queries. The API server rules are split across multiple files (`kube_apiserver-availability.libsonnet`, `kube_apiserver-burnrate.libsonnet`, etc.) due to complexity.

### Dashboards (`dashboards/`)

Dashboards are built with [grafonnet](https://github.com/grafana/grafonnet) (the sole jsonnet dependency, vendored via `jb`). `dashboards/defaults.libsonnet` holds shared panel/row construction helpers. The `resources/` and `network-usage/` subdirectories contain larger dashboard definitions split by topic.

### Library helpers (`lib/`)

- `utils.libsonnet` — `mapRuleGroups(f)` for transforming all rules, `wrap_rule_for_labels` for joining kube resource labels, `ifShowMultiCluster`.
- `add-runbook-links.libsonnet` — adds runbook URLs to alert annotations.
- `alerts.jsonnet`, `rules.jsonnet`, `dashboards.jsonnet` — thin wrappers that render the mixin object to YAML/JSON for `make generate`.

### Tests (`tests/`)

Unit tests use `promtool test rules`. Each `*-test.yaml` provides synthetic time series and asserts which alerts fire. `tests.yaml` is the primary test file loading both `prometheus_alerts.yaml` and `prometheus_rules.yaml`.

### Dependency management

`jsonnetfile.json` / `jsonnetfile.lock.json` — managed by `jb` (jsonnet-bundler). Dependencies are vendored into `vendor/`. The only runtime dependency is `grafonnet`.

### Customising the mixin (consumer pattern)

To override defaults without forking:

```jsonnet
local kubernetes = import 'kubernetes-mixin/mixin.libsonnet';

kubernetes {
  _config+:: {
    kubeStateMetricsSelector: 'job="my-ksm"',
    showMultiCluster: true,
    clusterLabel: 'cluster',
    grafanaK8s+:: {
      dashboardNamePrefix: 'My Org / ',
    },
  },
}
```

Use `lib/utils.libsonnet`'s `mapRuleGroups` to add extra annotations to all alerts without modifying upstream.

## Reference documentation

- `docs/prometheus2otel/metrics-inventory.md` — complete inventory of every Prometheus metric used in alerts, rules, and dashboards, grouped by source component (cAdvisor, kube-state-metrics, kubelet, node-exporter, control-plane components, windows-exporter, recording rule outputs)
- `docs/prometheus2otel/otel-mapping.md` — mapping to `kubeletstatsreceiver` / `k8sclusterreceiver` / `hostmetricsreceiver` metrics and OTel semantic conventions, including gap analysis and misalignment analysis between receiver implementations and the spec
