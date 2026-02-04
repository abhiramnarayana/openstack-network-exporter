# Prometheus rules for OVS metrics

This directory contains **spec-only** [PrometheusRule](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/) fragments for metrics exposed by openstack-network-exporter (OVS/OVN). Prepend one of the headers below to get a full manifest you can apply.

**Sample files:**

- `sample_ovs_pmd_alerts.yaml` – OVS PMD alerts (lost upcalls, non-voluntary context switches).
- `sample_ovs_interface_alerts.yaml` – OVS interface alerts (Rx dropped, Rx errors, Tx errors).
- `sample_openstack_network_exporter_down.yaml` – Critical alert when the openstack-network-exporter scrape target is down.

## Header options

Use one of the following blocks **above** the `spec:` from the sample file. The `spec:` is already in the YAML file; combine with the chosen header to form a complete CRD manifest.

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
```

### Vanilla Prometheus Operator

Use when your cluster uses the upstream Prometheus Operator (`monitoring.coreos.com`):

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: CHANGEME   # e.g. openstack-observability-network-exporter-down
  namespace: openstack                           # or the namespace your Prometheus watches
  labels:
    role: alert-rules                            # must match your Prometheus spec.ruleSelector.matchLabels
```

## Applying rules

1. Create a full manifest: paste one of the headers above, then the contents of the sample file (from `spec:` onward).
2. Apply in the namespace that your Prometheus's `ruleNamespaceSelector` watches (e.g. `openstack` for RHOSO):

   ```bash
   oc apply -f your-complete-rule.yaml -n openstack
   ```

3. Verify (OpenStack/RHOSO):

   ```bash
   oc get prometheusrules.monitoring.rhobs -n openstack
   ```
