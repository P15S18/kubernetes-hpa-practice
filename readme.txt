# Kubernetes Horizontal Pod Autoscaler (HPA) Implementation

This project demonstrates Kubernetes Horizontal Pod Autoscaling (HPA) using CPU utilization metrics.

# The implementation includes:

Kubernetes Deployment
Resource Requests and Limits
Kubernetes Service
Horizontal Pod Autoscaler (HPA)
Metrics Server Installation
CPU Load Generation
Automatic Pod Scaling Validation

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

# 1 Deployment Configuration
create deployment.yaml file with resource quota 

The deployment creates Nginx pods with CPU requests and limits.

# Resource Configuration

```hcl
resources:
  requests:
    cpu: "100m"
  limits:
    cpu: "200m"
```

Why Resource Requests Are Required

HPA calculates CPU utilization using:

CPU Usage / CPU Request × 100

Example:

CPU Request = 100m
Current Usage = 50m

Utilization = 50%

Without CPU requests, HPA cannot calculate utilization and scaling will not occur.

---

# 2. create service.yaml file 

---

# 3. Create hpa.yaml file with maxReplicas and minReplicas with matrics
  
  ```hcl
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

```hcl
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
```

---

# 5. verify every things.

```hcl
kubectl get pods
kubectl get svc
kubectl get hpa
```

---

# 6. Ensure Metrics Server is working

```hcl
kubectl top pods
```

👉 If this fails → HPA will NOT work

---

# 7. if it fails then install metrics server first

## Step 1: Install Metrics Server

```hcl
kubectl apply -f metrice-server.yaml
```
or
```hcl
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## Step 2: Verify installation
```hcl
kubectl get deployment -n kube-system
kubectl get pods -n kube-system
```

 Now you should see:

## metrics-server

#S tep 3: Now apply patch (important)

```hcl
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

# Step 4: Wait until pod is running

```
kubectl get pods -n kube-system | grep metrics
```

## Wait for:

metrics-server-xxxxx   Running 


# Step 4: Wait until pod is running

```hcl
kubectl get pods -n kube-system | grep metrics
```

Wait for:

metrics-server-xxxxx   Running

---

# 8 Generate Load 
🧪 Full Flow (Correct)

## Terminal 1 (Load Generator)

```hcl
kubectl run -i --tty load-generator --rm --image=busybox -- /bin/sh
while true; do wget -q -O- http://nginx-service & done
```

## Terminal 2 (Watch HPA)

```hcl
kubectl get hpa -w
```

## Terminal 3 (Watch pods)

```hcl
kubectl get pods -w
```

## Watch scaling liveTerminal 1:

```hcl
kubectl get hpa -w
```

## Terminal 2:

```hcl
kubectl get pods -w
```

#  🎯 Expected Result

```bash
Initial pods: 2
CPU increases due to load
HPA scales:
2 → 3 → 5 → 8 → ...
```

---

####################################################################
###################################################################
🚀 1. Real Load Testing using Apache JMeter

Instead of simple wget, use JMeter for realistic traffic.

## Setup
Run JMeter (local or container)
docker run -it --rm justb4/jmeter -n -t test-plan.jmx -l result.jtl

## Simple Test Plan
Thread Group → Users: 100
Loop Count: Infinite
HTTP Request → http://nginx-service

## This generates:

concurrent users
real HTTP load
sustained CPU usage → triggers HPA
🎯 Why JMeter?
Simulates real users
Better than curl/wget
Used in performance testing teams


Be ready for these:

❓ Why HPA not scaling?
metrics missing
CPU requests not set
load too low
❓ Why scaling delayed?
stabilization window
metrics lag

---

