<h1 align="center">📊 Prometheus Monitoring Stack on EKS</h1>

<p align="center">
  Full monitoring setup for a microservices application running in Kubernetes:
  <br>Prometheus, Grafana, Alertmanager and Redis Exporter deployed via Helm.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes">
  <img src="https://img.shields.io/badge/Amazon%20EKS-FF9900?style=for-the-badge&logo=amazon-eks&logoColor=white" alt="Amazon EKS">
  <img src="https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white" alt="Prometheus">
  <img src="https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white" alt="Grafana">
  <img src="https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white" alt="Helm">
  <img src="https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white" alt="Redis">
  <img src="https://img.shields.io/badge/YAML-CB171E?style=for-the-badge&logo=yaml&logoColor=white" alt="YAML">
</p>

---

## 📚 Table of Contents

- [Overview](#-overview)
- [Deploy Microservices in EKS](#-deploy-microservices-in-eks)
- [Deploy Prometheus Operator Stack](#-deploy-prometheus-operator-stack)
- [Access the UIs](#-access-the-uis)
- [Load & Stress Testing](#-load--stress-testing)
- [Deploy Redis Exporter](#-deploy-redis-exporter)

---

## 🧭 Overview

This repository contains the manifests and commands used to deploy a complete
observability stack (**Prometheus**, **Grafana**, **Alertmanager**) for a
microservices application running on **AWS EKS**, along with alerting rules
and a **Redis Exporter** for cache metrics.

| Component | Purpose |
|---|---|
| 🧩 EKS | Kubernetes cluster hosting the microservices |
| 🔥 Prometheus | Metrics collection & alerting rules engine |
| 📈 Grafana | Dashboards & visualization |
| 🚨 Alertmanager | Alert routing (e.g. email notifications) |
| 🧵 Redis Exporter | Redis metrics for Prometheus |

---

## 🚀 Deploy Microservices in EKS

```bash
eksctl create cluster
```

#### If you have a free tier account, use this configuration instead:

```bash
eksctl create cluster \
  --name monitoring-cluster \
  --region eu-north-1 \
  --node-type t3.small \
  --nodes 4 \
  --nodes-min 2 \
  --nodes-max 4
```

```bash
kubectl create namespace online-shop
kubectl apply -f config-microservices.yaml -n online-shop
```

#### 🌀 OPTIONAL — for Linode

```bash
chmod 400 ~/Downloads/online-shop-kubeconfig.yaml
export KUBECONFIG=~/Downloads/online-shop-kubeconfig.yaml
```

---

## ⚙️ Deploy Prometheus Operator Stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```

#### Or with a custom timeout and Grafana password:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --timeout 10m \
  --set grafana.adminPassword=prom-operator

helm ls
```

📎 Chart reference: [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)

### Check Prometheus Stack Pods

```bash
kubectl get all -n monitoring
```

---

## 🖥️ Access the UIs

### Prometheus

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring &
```

### Grafana

```bash
kubectl port-forward svc/monitoring-grafana 8080:80 -n monitoring &
```

| Field | Value |
|---|---|
| User | `admin` |
| Password | `prom-operator` |

#### If the default password doesn't work, retrieve it manually:

```bash
kubectl get secret monitoring-grafana -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 -d ; echo
```

### Alertmanager

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-alertmanager 9093:9093 &
```

---

## 🔥 Load & Stress Testing

### Trigger a CPU spike with many requests

Deploy a busybox pod so you can `curl` the application:

```bash
kubectl run curl-test --image=radial/busyboxplus:curl -i --tty --rm
```

or

```bash
kubectl run curl-test \
  --image=curlimages/curl:latest \
  -i --tty --rm -- sh
```

Curl the application endpoint (the external load balancer service endpoint) in a loop:

```bash
for i in $(seq 1 10000)
do
  curl ae4aee0715edc46b988c6ce67121bf57-1459479566.eu-west-3.elb.amazonaws.com > test.txt
done
```

### Create CPU stress directly

```bash
kubectl delete pod cpu-test
kubectl run cpu-test --image=containerstack/cpustress -- --cpu 4 --timeout 60s --metrics-brief
```

---

## 🧵 Deploy Redis Exporter

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update

helm install redis-exporter prometheus-community/prometheus-redis-exporter -f redis-values.yaml
```

---

<p align="center">Made with ❤️ for learning Kubernetes observability.</p>
