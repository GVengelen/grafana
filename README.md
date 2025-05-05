# GitOps Observability Stack

This repository manages the deployment of an observability stack using ArgoCD and Helm charts.

## Structure

- `apps/` - Contains ArgoCD Application manifests for each component
- `charts/` - Contains Helm values overrides for each chart
- `app-of-apps.yaml` - The ArgoCD App of Apps manifest to bootstrap all components

## Components
- Kube-prometheus-stack (Prometheus, Alertmanager, Grafana)
- Grafana/k8s-monitoring-helm
- Grafana/loki
- Grafana/tempo
- Faro example app
- Argocd

### Argocd
We use ArgoCD to enable GitOps. GitOps allows you to manage your cluster without depending on pipelines. This makes rolling out clusters much easier. And because you don't need acces to the cluster you don't have to store credentials in you CI/CD platform.

ArgoCD does require some initial, for which you need one-time acces to the cluster. Which is unsafe ;)
1. Install ArgoCD in your cluster.
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
2. Apply `app-of-apps.yaml` to bootstrap the stack:
```bash
kubectl apply -f app-of-apps.yaml
```
3. ArgoCD will manage the rest via GitOps.

If you want to see the state of your ArgoCD deployments, create a port forward.
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:80
```
And use credentials from:
```bash
kubectl get secret argocd-initial-admin-secret  -n argocd -o yaml
```

### Prometheus
We use Prometheus as a metric datasource. We choose Prometheus over Mimir because Prometheus can be run as a single datasource via Helm. The downside of Prometheus is that is does not natively support S3-like storage solutions. For Prometheus to be able to recieve metrics from Alloy we'll need to enable the remote write endpoint. After that we're good to go!

### Alertmanager
We want to minimize traffic between the regional clusters and our main Grafana instance. And we want the regional clusters to be able to send alerts even when our main Grafana instance is not working. In the scope of this example we're only deploying a local cluster so it makes even more sence to deploy Alertmanger in the cluster.

### Grafana
This is a Grafana example, so it makes sense we're going to deploy Grafana. In this example we choose for a cluster deployed instance managed by kube-prometheus-stack. But you can choose to use any form of Grafana that you want, Azure Managed Grafana, Grafana Cloud, Grafana on a VM, etc. 

If you want to login to grafana in this cluster

```bash
kubectl port-forward svc/grafana-server -n monitoring 3000:3000
```

username: admin
password: supersecretpassword

### K8s monitoring
To bootstrap our instrumentation we are using K8s-monitoring Helm chart. This Helm chart allows us to bootstrap our cluster with instrumentation. It scrapes logs from all our pods, it scrapes metrics, it collects traces using eBPF(Beyla), and it supports APM with the Faro SDK.

### Loki
We use Loki to store and expose logs. In this example we use the singleBinary deployment, so we can run Loki in KinD. There is not much to it, we took the basic values and that works fine. For storage this example uses Minio for S3 storage, but any object store will do(AWS, Google, Azure, etc.)

### Tempo
We use Tempo to store and expose traces. Just like Loki we deploy Tempo in a single instance. And again Minio is used for S3 storage.

### Faro app
This Faro app is based on Grafana's Faro-Next-example, with a few additions. We've added a proxy so the client can send all the data through the server. This allows us to keep secrets away from the client and secure our datasource. And we've added a basic Docker file and Github workflow to be able to create images and deploy them on the cluster.