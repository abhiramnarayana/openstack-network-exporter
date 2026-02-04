# Prometheus rules for OVS metrics

This directory contains [Prometheus alerting rule][alerts] files for
metrics exposed by openstack-network-exporter (OVS/OVN). These files can be
used directly with standalone Prometheus or combined with a CRD header for
Kubernetes/RHOSO.

[alerts]: https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/

**Sample files:**

- `sample_ovs_pmd_alerts.yaml` – OVS PMD alerts (lost upcalls, non-voluntary
  context switches).
- `sample_ovs_interface_alerts.yaml` – OVS interface alerts (Rx dropped,
  Rx errors, Tx errors).
- `sample_openstack_network_exporter_down.yaml` – Critical alert when the
  openstack-network-exporter scrape target is down.

## Kubernetes / RHOSO (PrometheusRule CRD)

For Kubernetes or RHOSO deployments using the Prometheus Operator, prepend
one of the following headers to the sample file content to create a complete
PrometheusRule CRD manifest.

### OpenStack / RHOSO (telemetry-operator + metric-storage)

Use when your cluster runs Red Hat Observability (`monitoring.rhobs`):

```yaml
apiVersion: monitoring.rhobs/v1
kind: PrometheusRule
metadata:
  name: CHANGEME   # e.g. openstack-observability-network-exporter-down
  namespace: openstack
  labels:
    service: metricStorage
spec:
```

### Vanilla Prometheus Operator

Use when your cluster uses the upstream Prometheus Operator
(`monitoring.coreos.com`):

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: CHANGEME   # e.g. openstack-observability-network-exporter-down
  namespace: openstack   # or the namespace your Prometheus watches
  labels:
    role: alert-rules    # must match your Prometheus spec.ruleSelector
spec:
```

### Applying rules (Kubernetes / RHOSO)

1. Create a full manifest: paste one of the headers above, then the contents
   of the sample file.
2. Apply in the namespace that your Prometheus's `ruleNamespaceSelector`
   watches (e.g. `openstack` for RHOSO):

   ```bash
   oc apply -f your-complete-rule.yaml -n openstack
   ```

3. Verify (OpenStack/RHOSO):

   ```bash
   oc get prometheusrules.monitoring.rhobs -n openstack
   ```

## Standalone Prometheus (no Kubernetes)

For a standalone Prometheus installation, use the sample files directly:

1. **Add the rule files to `prometheus.yml`:**

   ```yaml
   # Alertmanager configuration
   alerting:
     alertmanagers:
       - static_configs:
           - targets: ['localhost:9093']

   # Load rules once and periodically evaluate them according to the
   # global 'evaluation_interval'.
   rule_files:
     - /path/to/openstack-network-exporter/observability/prometheus/rules/sample_ovs_pmd_alerts.yaml
     - /path/to/openstack-network-exporter/observability/prometheus/rules/sample_ovs_interface_alerts.yaml
     # - /path/to/openstack-network-exporter/observability/prometheus/rules/sample_openstack_network_exporter_down.yaml
   ```

2. **Reload Prometheus** to pick up the new rules:

   ```bash
   curl -X POST http://localhost:9090/-/reload
   # or restart the Prometheus service
   ```
