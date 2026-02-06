# Grafana Dashboards

GitOps-managed Grafana dashboards for the k3s homelab cluster.

## Overview

This repository contains Grafana dashboards as Kubernetes ConfigMaps. Dashboards are automatically discovered and loaded by Grafana's sidecar container using the `grafana_dashboard: "1"` label.

## Directory Structure

```
grafana-dashboards/
├── kubernetes/              # Kubernetes cluster dashboards
├── applications/            # Custom application dashboards (Longhorn, Traefik, etc.)
├── kube-prometheus-defaults/ # Selected defaults from kube-prometheus-stack
└── templates/               # ConfigMap template for adding new dashboards
```

## How It Works

1. **ArgoCD Application** watches this repository
2. **ConfigMaps with `grafana_dashboard: "1"` label** are discovered by Grafana sidecar
3. **Dashboard JSON** embedded in ConfigMap data is loaded into Grafana
4. **Folder organization** controlled via `grafana_folder` annotation

## Adding a New Dashboard

1. Export dashboard JSON from Grafana (Share > Export > Save to file)
2. Create ConfigMap using the template:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboard-<dashboard-name>
  namespace: monitoring
  labels:
    grafana_dashboard: "1"
  annotations:
    grafana_folder: "<folder-name>"
data:
  <dashboard-name>.json: |-
    <paste dashboard JSON here>
```

3. Place in appropriate directory:
   - `kubernetes/` - K8s resource dashboards
   - `applications/` - Application-specific dashboards
   - `kube-prometheus-defaults/` - Standard monitoring dashboards

4. Commit and push - ArgoCD will sync automatically

## Requirements

- Dashboard ConfigMap MUST have `grafana_dashboard: "1"` label
- Dashboard JSON MUST be valid (exported from Grafana or official source)
- Datasource references should use `${datasource}` template variable for portability

## Datasource Convention

All dashboards should use template variables for datasource selection:

```json
"templating": {
  "list": [
    {
      "name": "datasource",
      "type": "datasource",
      "query": "prometheus"
    }
  ]
}
```

This allows dashboards to work with any Prometheus-compatible datasource (Prometheus, VictoriaMetrics, etc.).

## Related

- **Deployed by:** ArgoCD Application in `manifests/argocd/apps/grafana-dashboards.yaml`
- **Loaded by:** Grafana sidecar with `searchNamespace: ALL` configuration
- **Main cluster repo:** [tcude/k3s-homelab](https://github.com/tcude/k3s-homelab)
