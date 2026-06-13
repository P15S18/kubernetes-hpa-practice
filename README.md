# Kubernetes Horizontal Pod Autoscaler (HPA) Implementation

This project demonstrates Kubernetes Horizontal Pod Autoscaling (HPA) using CPU utilization metrics.

## The implementation includes:

* Kubernetes Deployment
* Resource Requests and Limits
* Kubernetes Service
* Horizontal Pod Autoscaler (HPA)
* Metrics Server Installation
* CPU Load Generation
* Automatic Pod Scaling Validation

---

# Project Structure

```bash
kubernetes-hpa-lab/
│
├── deployment.yaml
├── service.yaml
├── hpa.yaml
└── README.md
```

---

# Architecture Overview

```bash
                User Traffic
                      │
                      ▼
                Kubernetes Service
                      │
                      ▼
                Deployment
                      │
             Multiple Pod Replicas
                      │
                      ▼
               Metrics Server
                      │
                      ▼
        Horizontal Pod Autoscaler
                      │
                      ▼
          Scale Up / Scale Down
```

---

# 1. Deployment Configuration

Create `deployment.yaml` file with resource quota.

The deployment creates Nginx pods with CPU requests and limits.

## Resource Configuration

```yaml
resources:
  requests:
    cpu: "100m"
  limits:
    cpu: "200m"
```

## Why Resource Requests Are Required

HPA calculates CPU utilization using:

```text
CPU Usage / CPU Request × 100
```

Example:

```text
CPU Request = 100m
Current Usage = 50m

Utilization = 50%
```

Without CPU requests, HPA cannot calculate utilization and scaling will not occur.

---

# 2. Create service.yaml file

---

# 3. Create hpa.yaml file with maxReplicas and minReplicas with metrics

```yaml
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

# 4. Apply all the resources

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
```

---

# 5. Verify everything

```bash
kubectl get pods
kubectl get svc
kubectl get hpa
```

---

# 6. Ensure Metrics Server is working

```bash
kubectl top pods
```

👉 If this fails → HPA will NOT work.

---

# 7. If it fails then install Metrics Server first

## Step 1: Install Metrics Server

```bash
kubectl apply -f metrice-server.yaml
```

or

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## Step 2: Verify installation

```bash
kubectl get deployment -n kube-system
kubectl get pods -n kube-system
```

Now you should see:

```text
metrics-server
```

## Step 3: Now apply patch (important)

```bash
kubectl patch deployment metrics-server -n kube-system \
  --type='json' \
  -p='[
    {
      "op": "add",
      "path": "/spec/template/spec/containers/0/args/-",
      "value": "--kubelet-insecure-tls"
    }
  ]'
```

## Step 4: Wait until pod is running

```bash
kubectl get pods -n kube-system | grep metrics
```

Wait for:

```text
metrics-server-xxxxx   Running
```

---

# 8. Generate Load

## Full Flow (Correct)

### Terminal 1 (Load Generator)

```bash
kubectl run -i --tty load-generator --rm --image=busybox -- /bin/sh
while true; do wget -q -O- http://nginx-service; done
```

### Terminal 2 (Watch HPA)

```bash
kubectl get hpa -w
```

### Terminal 3 (Watch Pods)

```bash
kubectl get pods -w
```

## Watch Scaling Live

### Terminal 1

```bash
kubectl get hpa -w
```

### Terminal 2

```bash
kubectl get pods -w
```

# Expected Result

```bash
Initial pods: 2
CPU increases due to load

HPA scales:
2 → 3 → 5 → 8 → ...
```

---

# Real Load Testing using Apache JMeter

Instead of simple wget, use JMeter for realistic traffic.

## Setup

Run JMeter (local or container)

```bash
docker run -it --rm justb4/jmeter -n -t test-plan.jmx -l result.jtl
```

## Simple Test Plan

* Thread Group → Users: 100
* Loop Count: Infinite
* HTTP Request → http://nginx-service

## This generates:

* concurrent users
* real HTTP load
* sustained CPU usage → triggers HPA

## Why JMeter?

* Simulates real users
* Better than curl/wget
* Used in performance testing teams

Be ready for these:

### Why HPA not scaling?

* metrics missing
* CPU requests not set
* load too low

### Why scaling delayed?

* stabilization window
* metrics lag

---
