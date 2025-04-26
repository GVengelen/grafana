# GitOps Observability Stack

This repository manages the deployment of an observability stack using ArgoCD and Helm charts.

## Structure

- `apps/` - Contains ArgoCD Application manifests for each component
- `charts/` - Contains Helm values overrides for each chart
- `app-of-apps.yaml` - The ArgoCD App of Apps manifest to bootstrap all components

## Components
- kube-prometheus-stack (Prometheus, Alertmanager, Grafana)
- grafana/k8s-monitoring-helm
- grafana/loki
- grafana/tempo

## Usage
1. Install ArgoCD in your cluster.
2. Apply `app-of-apps.yaml` to bootstrap the stack:
   ```sh
   kubectl apply -f app-of-apps.yaml
   ```
3. ArgoCD will manage the rest via GitOps.
