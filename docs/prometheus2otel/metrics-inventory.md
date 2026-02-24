# Prometheus Metrics Inventory — kubernetes-mixin

All Prometheus metric names and labels referenced in PromQL expressions across
`alerts/`, `rules/`, and `dashboards/` of this repository.
Labels shown are the union of all labels used in selectors, `by()`/`on()` clauses,
and `group_left()`/`group_right()` across all usages.

---

## 1. cAdvisor (source: `cadvisor`)

| Metric | Labels used | Used in |
|--------|------------|---------|
| `container_cpu_usage_seconds_total` | `namespace`, `pod`, `container`, `image`, `job` | rules, dashboards |
| `container_cpu_cfs_throttled_periods_total` | `namespace`, `pod`, `container`, `job` | alerts, dashboards |
| `container_cpu_cfs_periods_total` | `namespace`, `pod`, `container`, `job` | alerts, dashboards |
| `container_memory_working_set_bytes` | `namespace`, `pod`, `container`, `image`, `job` | rules, dashboards |
| `container_memory_rss` | `namespace`, `pod`, `container`, `image`, `job` | rules, dashboards |
| `container_memory_cache` | `namespace`, `pod`, `container`, `image`, `job` | rules, dashboards |
| `container_memory_swap` | `namespace`, `pod`, `container`, `image`, `job` | rules, dashboards |
| `container_network_receive_bytes_total` | `namespace`, `pod`, `job` | dashboards |
| `container_network_transmit_bytes_total` | `namespace`, `pod`, `job` | dashboards |
| `container_network_receive_packets_total` | `namespace`, `pod`, `job` | dashboards |
| `container_network_transmit_packets_total` | `namespace`, `pod`, `job` | dashboards |
| `container_network_receive_packets_dropped_total` | `namespace`, `pod`, `job` | dashboards |
| `container_network_transmit_packets_dropped_total` | `namespace`, `pod`, `job` | dashboards |
| `container_fs_reads_total` | `namespace`, `pod`, `device`, `job` | dashboards |
| `container_fs_writes_total` | `namespace`, `pod`, `device`, `job` | dashboards |
| `container_fs_reads_bytes_total` | `namespace`, `pod`, `device`, `job` | dashboards |
| `container_fs_writes_bytes_total` | `namespace`, `pod`, `device`, `job` | dashboards |

---

## 2. kube-state-metrics (source: `kube-state-metrics`)

### Pods

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_pod_status_phase` | `namespace`, `pod`, `job`, `phase` | alerts |
| `kube_pod_container_status_waiting_reason` | `namespace`, `pod`, `container`, `reason` | alerts |
| `kube_pod_container_resource_requests` | `namespace`, `pod`, `container`, `resource`, `job` | alerts, rules, dashboards |
| `kube_pod_container_resource_limits` | `namespace`, `pod`, `container`, `resource`, `job` | alerts, rules, dashboards |
| `kube_pod_owner` | `namespace`, `pod`, `owner_kind`, `owner_name` | rules, dashboards |
| `kube_pod_info` | `namespace`, `pod`, `node` | rules |
| `kube_pod_container_info` | `container_id`, `pod`, `namespace`, `container` | rules |

### Deployments

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_deployment_spec_replicas` | `namespace`, `deployment`, `job` | alerts |
| `kube_deployment_status_replicas_available` | `namespace`, `deployment`, `job` | alerts |
| `kube_deployment_status_replicas_updated` | `namespace`, `deployment`, `job` | alerts |
| `kube_deployment_status_observed_generation` | `namespace`, `deployment`, `job` | alerts |
| `kube_deployment_metadata_generation` | `namespace`, `deployment`, `job` | alerts |
| `kube_deployment_status_condition` | `namespace`, `deployment`, `condition`, `status` | alerts |

### StatefulSets

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_statefulset_replicas` | `namespace`, `statefulset`, `job` | alerts |
| `kube_statefulset_status_replicas_ready` | `namespace`, `statefulset`, `job` | alerts |
| `kube_statefulset_status_replicas_updated` | `namespace`, `statefulset`, `job` | alerts |
| `kube_statefulset_status_observed_generation` | `namespace`, `statefulset`, `job` | alerts |
| `kube_statefulset_metadata_generation` | `namespace`, `statefulset`, `job` | alerts |
| `kube_statefulset_status_current_revision` | `namespace`, `statefulset`, `job` | alerts |
| `kube_statefulset_status_update_revision` | `namespace`, `statefulset`, `job` | alerts |

### DaemonSets

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_daemonset_status_desired_number_scheduled` | `namespace`, `daemonset`, `job` | alerts |
| `kube_daemonset_status_current_number_scheduled` | `namespace`, `daemonset`, `job` | alerts |
| `kube_daemonset_status_number_misscheduled` | `namespace`, `daemonset`, `job` | alerts |
| `kube_daemonset_status_updated_number_scheduled` | `namespace`, `daemonset`, `job` | alerts |
| `kube_daemonset_status_number_available` | `namespace`, `daemonset`, `job` | alerts |

### HorizontalPodAutoscalers

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_horizontalpodautoscaler_spec_max_replicas` | `namespace`, `horizontalpodautoscaler`, `job` | alerts |
| `kube_horizontalpodautoscaler_spec_min_replicas` | `namespace`, `horizontalpodautoscaler`, `job` | alerts |
| `kube_horizontalpodautoscaler_status_current_replicas` | `namespace`, `horizontalpodautoscaler`, `job` | alerts |
| `kube_horizontalpodautoscaler_status_desired_replicas` | `namespace`, `horizontalpodautoscaler`, `job` | alerts |

### Jobs

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_job_status_start_time` | `namespace`, `job_name`, `job` | alerts |
| `kube_job_status_active` | `namespace`, `job_name`, `job` | alerts |
| `kube_job_failed` | `namespace`, `job_name`, `job` | alerts |

### Nodes

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_node_status_condition` | `node`, `condition`, `status` | alerts |
| `kube_node_spec_unschedulable` | `node` | alerts |
| `kube_node_spec_taint` | `node`, `key`, `value`, `effect` | alerts |
| `kube_node_status_allocatable` | `node`, `resource`, `job` | alerts, rules, dashboards |
| `kube_node_status_capacity` | `node`, `resource` | alerts |
| `kube_node_role` | `node`, `role` | alerts |
| `kube_node_info` | `node` | alerts |

### PersistentVolumes / PVCs

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_persistentvolume_status_phase` | `persistentvolume`, `phase` | alerts |
| `kube_persistentvolumeclaim_access_mode` | `namespace`, `persistentvolumeclaim`, `access_mode`, `job` | alerts |
| `kube_persistentvolumeclaim_labels` | `namespace`, `persistentvolumeclaim` | alerts |

### PodDisruptionBudgets

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_poddisruptionbudget_status_desired_healthy` | `namespace`, `poddisruptionbudget`, `job` | alerts |
| `kube_poddisruptionbudget_status_current_healthy` | `namespace`, `poddisruptionbudget`, `job` | alerts |

### ReplicaSets / ReplicationControllers

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_replicaset_owner` | `namespace`, `replicaset`, `owner_kind`, `owner_name` | rules |

### Resource Quotas

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kube_resourcequota` | `namespace`, `resource`, `type`, `resourcequota` | alerts, dashboards |

---

## 3. Kubelet (source: `kubelet`)

### Pod / Container / Node Runtime

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kubelet_running_pods` | `node`, `instance`, `job` | alerts, dashboards |
| `kubelet_running_containers` | `instance`, `job` | dashboards |
| `kubelet_node_name` | `node`, `instance`, `job` | alerts, rules, dashboards |
| `kubelet_node_config_error` | `instance` | dashboards |
| `kubelet_evictions` | `instance`, `eviction_signal` | alerts |
| `kubelet_pod_worker_duration_seconds_bucket` | `instance`, `operation_type`, `le` | alerts, dashboards |
| `kubelet_pod_start_duration_seconds_bucket` | `instance`, `le` | dashboards |

### PLEG

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kubelet_pleg_relist_duration_seconds_bucket` | `instance`, `le` | alerts, rules, dashboards |
| `kubelet_pleg_relist_duration_seconds` | `instance` | rules |
| `kubelet_pleg_relist_interval_seconds_bucket` | `instance`, `le` | dashboards |

### Certificate Management

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kubelet_certificate_manager_client_ttl_seconds` | `node` | alerts |
| `kubelet_certificate_manager_server_ttl_seconds` | `node` | alerts |
| `kubelet_certificate_manager_client_expiration_renew_errors` | `node` | alerts |
| `kubelet_server_expiration_renew_errors` | `node` | alerts |

### Runtime Operations

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kubelet_runtime_operations_total` | `instance`, `operation_type` | dashboards |
| `kubelet_runtime_operations_errors_total` | `instance`, `operation_type` | dashboards |
| `kubelet_runtime_operations_duration_seconds_bucket` | `instance`, `operation_type`, `le` | dashboards |

### cgroup / Storage

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kubelet_cgroup_manager_duration_seconds_count` | `instance`, `operation_type` | dashboards |
| `kubelet_cgroup_manager_duration_seconds_bucket` | `instance`, `operation_type`, `le` | dashboards |
| `storage_operation_duration_seconds_count` | `instance`, `operation_name`, `volume_plugin` | dashboards |
| `storage_operation_errors_total` | `instance`, `operation_name`, `volume_plugin` | dashboards |
| `storage_operation_duration_seconds_bucket` | `instance`, `operation_name`, `volume_plugin`, `le` | dashboards |
| `volume_manager_total_volumes` | `instance`, `state` | dashboards |

### Volume Stats

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kubelet_volume_stats_available_bytes` | `namespace`, `persistentvolumeclaim`, `job` | alerts |
| `kubelet_volume_stats_capacity_bytes` | `namespace`, `persistentvolumeclaim`, `job` | alerts, dashboards |
| `kubelet_volume_stats_used_bytes` | `namespace`, `persistentvolumeclaim`, `job` | alerts |
| `kubelet_volume_stats_inodes` | `namespace`, `persistentvolumeclaim`, `job` | alerts |
| `kubelet_volume_stats_inodes_free` | `namespace`, `persistentvolumeclaim`, `job` | alerts |
| `kubelet_volume_stats_inodes_used` | `namespace`, `persistentvolumeclaim`, `job` | alerts |

---

## 4. node-exporter (source: `node-exporter`)

| Metric | Labels used | Used in |
|--------|------------|---------|
| `node_cpu_seconds_total` | `instance`, `job`, `mode` | rules |
| `node_memory_MemAvailable_bytes` | `instance`, `job` | rules, dashboards |
| `node_memory_MemTotal_bytes` | `instance`, `job` | dashboards |
| `node_memory_MemFree_bytes` | `instance`, `job` | rules |
| `node_memory_Buffers_bytes` | `instance`, `job` | rules |
| `node_memory_Cached_bytes` | `instance`, `job` | rules |
| `node_memory_Slab_bytes` | `instance`, `job` | rules |

---

## 5. kube-apiserver (source: `kube-apiserver`)

| Metric | Labels used | Used in |
|--------|------------|---------|
| `apiserver_request_total` | `verb`, `code`, `resource`, `job` | alerts, rules, dashboards |
| `apiserver_request_sli_duration_seconds_bucket` | `verb`, `scope`, `resource`, `le` | rules, dashboards |
| `apiserver_request_sli_duration_seconds_count` | `verb`, `scope` | rules |
| `apiserver_request_terminations_total` | `job` | alerts |
| `apiserver_client_certificate_expiration_seconds_bucket` | `le`, `job` | alerts |
| `apiserver_client_certificate_expiration_seconds_count` | `job` | alerts |
| `aggregator_unavailable_apiservice_total` | `instance`, `name`, `reason` | alerts |
| `aggregator_unavailable_apiservice` | `instance`, `name`, `namespace` | alerts |

---

## 6. kube-scheduler (source: `kube-scheduler`)

| Metric | Labels used | Used in |
|--------|------------|---------|
| `scheduler_scheduling_attempt_duration_seconds_bucket` | `instance`, `le` | rules, dashboards |
| `scheduler_scheduling_attempt_duration_seconds_count` | `instance` | dashboards |
| `scheduler_scheduling_algorithm_duration_seconds_bucket` | `instance`, `le` | rules, dashboards |
| `scheduler_scheduling_algorithm_duration_seconds_count` | `instance` | dashboards |
| `scheduler_pod_scheduling_sli_duration_seconds_bucket` | `instance`, `le` | rules, dashboards |
| `scheduler_pod_scheduling_sli_duration_seconds_count` | `instance` | dashboards |
| `scheduler_volume_scheduling_duration_seconds_bucket` | `instance`, `le` | dashboards |
| `scheduler_volume_scheduling_duration_seconds_count` | `instance` | dashboards |

---

## 7. kube-controller-manager (source: `kube-controller-manager`)

| Metric | Labels used | Used in |
|--------|------------|---------|
| `workqueue_adds_total` | `instance`, `name` | dashboards |
| `workqueue_depth` | `instance`, `name` | dashboards |
| `workqueue_queue_duration_seconds_bucket` | `instance`, `name`, `le` | dashboards |

---

## 8. kube-proxy (source: `kube-proxy`)

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kubeproxy_sync_proxy_rules_duration_seconds_count` | `instance` | dashboards |
| `kubeproxy_sync_proxy_rules_duration_seconds_bucket` | `instance`, `le` | dashboards |
| `kubeproxy_network_programming_duration_seconds_count` | `instance` | dashboards |
| `kubeproxy_network_programming_duration_seconds_bucket` | `instance`, `le` | dashboards |

---

## 9. Shared Go / REST client metrics (apiserver, scheduler, controller-manager, proxy)

| Metric | Labels used | Used in |
|--------|------------|---------|
| `rest_client_requests_total` | `code`, `instance`, `job`, `verb` | alerts, dashboards |
| `rest_client_request_duration_seconds_bucket` | `instance`, `verb`, `le` | dashboards |
| `process_resident_memory_bytes` | `instance` | dashboards |
| `process_cpu_seconds_total` | `instance` | dashboards |
| `go_goroutines` | `instance` | dashboards |

---

## 10. Other Kubernetes metrics

| Metric | Labels used | Used in |
|--------|------------|---------|
| `kubernetes_build_info` | `git_version` | alerts |

---

## 11. windows-exporter (source: `kubernetes-windows-exporter`)

| Metric | Labels used | Used in |
|--------|------------|---------|
| `windows_system_boot_time_timestamp_seconds` | `instance` | rules |
| `windows_cpu_time_total` | `instance`, `core`, `mode` | rules |
| `windows_memory_available_bytes` | `instance` | rules |
| `windows_os_visible_memory_bytes` | `instance` | rules |
| `windows_memory_cache_bytes` | `instance` | rules |
| `windows_memory_modified_page_list_bytes` | `instance` | rules |
| `windows_memory_standby_cache_core_bytes` | `instance` | rules |
| `windows_memory_standby_cache_normal_priority_bytes` | `instance` | rules |
| `windows_memory_standby_cache_reserve_bytes` | `instance` | rules |
| `windows_memory_swap_page_operations_total` | `instance` | rules |
| `windows_logical_disk_read_seconds_total` | `instance` | rules |
| `windows_logical_disk_write_seconds_total` | `instance` | rules |
| `windows_logical_disk_size_bytes` | `instance`, `volume` | rules |
| `windows_logical_disk_free_bytes` | `instance`, `volume` | rules |
| `windows_net_bytes_total` | `instance` | rules |
| `windows_net_packets_received_discarded_total` | `instance` | rules |
| `windows_net_packets_outbound_discarded_total` | `instance` | rules |
| `windows_container_available` | `container_id` | rules |
| `windows_container_cpu_usage_seconds_total` | `container_id` | rules |
| `windows_container_memory_usage_commit_bytes` | `container_id` | rules |
| `windows_container_memory_usage_private_working_set_bytes` | `container_id` | rules |
| `windows_container_network_receive_bytes_total` | `container_id` | rules |
| `windows_container_network_transmit_bytes_total` | `container_id` | rules |

---

## 12. Recording rule output metrics

These are produced internally by this mixin's recording rules and consumed by dashboards/alerts.
They are **not** scraped from an external source.

### Container / pod / node aggregates

| Recording rule output metric | Labels | Inputs |
|------------------------------|--------|--------|
| `node_namespace_pod_container:container_cpu_usage_seconds_total:sum_rate5m` | `namespace`, `pod`, `container`, `node` | `container_cpu_usage_seconds_total` + `kube_pod_info` |
| `node_namespace_pod_container:container_cpu_usage_seconds_total:sum_irate` | `namespace`, `pod`, `container`, `node` | same (deprecated) |
| `node_namespace_pod_container:container_memory_working_set_bytes` | `namespace`, `pod`, `container`, `node` | `container_memory_working_set_bytes` + `kube_pod_info` |
| `node_namespace_pod_container:container_memory_rss` | `namespace`, `pod`, `container`, `node` | `container_memory_rss` + `kube_pod_info` |
| `node_namespace_pod_container:container_memory_cache` | `namespace`, `pod`, `container`, `node` | `container_memory_cache` + `kube_pod_info` |
| `node_namespace_pod_container:container_memory_swap` | `namespace`, `pod`, `container`, `node` | `container_memory_swap` + `kube_pod_info` |
| `node_namespace_pod:kube_pod_info:` | `namespace`, `pod`, `node` | `kube_pod_info` + `kubelet_node_name` |
| `node:node_num_cpu:sum` | `node` | `kube_node_status_allocatable` |
| `:node_memory_MemAvailable_bytes:sum` | _(cluster-wide)_ | `node_memory_MemAvailable_bytes` |
| `node:node_cpu_utilization:ratio_rate5m` | `node` | `node_cpu_seconds_total` + `node:node_num_cpu:sum` |
| `cluster:node_cpu:ratio_rate5m` | _(cluster-wide)_ | same |

### Resource requests / limits aggregates

| Recording rule output metric | Labels | Inputs |
|------------------------------|--------|--------|
| `cluster:namespace:pod_memory:active:kube_pod_container_resource_requests` | `namespace`, `pod` | `kube_pod_container_resource_requests` + `kube_pod_status_phase` |
| `namespace_memory:kube_pod_container_resource_requests:sum` | `namespace` | same |
| `cluster:namespace:pod_cpu:active:kube_pod_container_resource_requests` | `namespace`, `pod` | same |
| `namespace_cpu:kube_pod_container_resource_requests:sum` | `namespace` | same |
| `cluster:namespace:pod_memory:active:kube_pod_container_resource_limits` | `namespace`, `pod` | `kube_pod_container_resource_limits` + `kube_pod_status_phase` |
| `namespace_memory:kube_pod_container_resource_limits:sum` | `namespace` | same |
| `cluster:namespace:pod_cpu:active:kube_pod_container_resource_limits` | `namespace`, `pod` | same |
| `namespace_cpu:kube_pod_container_resource_limits:sum` | `namespace` | same |

### Workload topology

| Recording rule output metric | Labels | Inputs |
|------------------------------|--------|--------|
| `namespace_workload_pod:kube_pod_owner:relabel` | `namespace`, `pod`, `workload`, `workload_type` | `kube_pod_owner` + `kube_replicaset_owner` |

### Kubelet PLEG quantiles

| Recording rule output metric | Labels |
|------------------------------|--------|
| `node_quantile:kubelet_pleg_relist_duration_seconds:histogram_quantile` | `node`, `quantile` |

### API server SLO / burn-rate

| Recording rule output metric | Labels |
|------------------------------|--------|
| `code_verb:apiserver_request_total:increase1h` | `code`, `verb` |
| `code_verb:apiserver_request_total:increase30d` | `code`, `verb` |
| `code:apiserver_request_total:increase30d` | `code`, `verb` |
| `cluster_verb_scope_le:apiserver_request_sli_duration_seconds_bucket:increase1h` | `verb`, `scope`, `le` |
| `cluster_verb_scope_le:apiserver_request_sli_duration_seconds_bucket:increase30d` | `verb`, `scope`, `le` |
| `cluster_verb_scope:apiserver_request_sli_duration_seconds_count:increase1h` | `verb`, `scope` |
| `cluster_verb_scope:apiserver_request_sli_duration_seconds_count:increase30d` | `verb`, `scope` |
| `apiserver_request:availability30d` | `verb` |
| `code_resource:apiserver_request_total:rate5m` | `code`, `resource`, `verb` |
| `cluster_quantile:apiserver_request_sli_duration_seconds:histogram_quantile` | `verb`, `quantile`, `resource` |
| `apiserver_request:burnrate5m` | `verb` |
| `apiserver_request:burnrate30m` | `verb` |
| `apiserver_request:burnrate1h` | `verb` |
| `apiserver_request:burnrate6h` | `verb` |

### Scheduler quantiles

| Recording rule output metric | Labels |
|------------------------------|--------|
| `cluster_quantile:scheduler_scheduling_attempt_duration_seconds:histogram_quantile` | `quantile` |
| `cluster_quantile:scheduler_scheduling_algorithm_duration_seconds:histogram_quantile` | `quantile` |
| `cluster_quantile:scheduler_pod_scheduling_sli_duration_seconds:histogram_quantile` | `quantile` |
