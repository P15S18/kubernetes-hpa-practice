---
#1. need to create deployment.yaml file with resource quta
          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "200m"

---
# 2. create service.yaml file 

---

# 3. Create hpa.yaml file with maxReplicas and minReplicas with matrics
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
---

#4. Apply all the resources
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml

---

#5. verify every things.
kubectl get pods
kubectl get svc
kubectl get hpa

---

#6. Ensure Metrics Server is working
kubectl top pods

---

👉 If this fails → HPA will NOT work

#7. if it fails then install metrics server first
Step 1: Install Metrics Server
kubectl apply -f metrice-server.yaml

or

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

#Step 2: Verify installation
kubectl get deployment -n kube-system
kubectl get pods -n kube-system


👉 Now you should see:

metrics-server

#Step 3: Now apply patch (important)
kubectl patch deployment metrics-server -n kube-system \
  --type='json' \
  -p='[
    {
      "op": "add",
      "path": "/spec/template/spec/containers/0/args/-",
      "value": "--kubelet-insecure-tls"
    }
  ]'


#Step 4: Wait until pod is running
kubectl get pods -n kube-system | grep metrics


Wait for:

metrics-server-xxxxx   Running 


#Step 4: Wait until pod is running
kubectl get pods -n kube-system | grep metrics


Wait for:

metrics-server-xxxxx   Running

---

# 8 Generate Load 
🧪 Full Flow (Correct)
Terminal 1 (Load Generator)
kubectl run -i --tty load-generator --rm --image=busybox -- /bin/sh


Then:

while true; do wget -q -O- http://nginx-service & done

Terminal 2 (Watch HPA)
kubectl get hpa -w

Terminal 3 (Watch pods)
kubectl get pods -w




Step 5: Watch scaling liveTerminal 1:
kubectl get hpa -w

Terminal 2:
kubectl get pods -w

🎯 Expected Result
Initial pods: 2
CPU increases due to load
HPA scales:
2 → 3 → 5 → 8 → ...

####################################################################
###################################################################
🚀 1. Real Load Testing using Apache JMeter

Instead of simple wget, use JMeter for realistic traffic.

✅ Setup
Run JMeter (local or container)
docker run -it --rm justb4/jmeter -n -t test-plan.jmx -l result.jtl

✅ Simple Test Plan
Thread Group → Users: 100
Loop Count: Infinite
HTTP Request → http://nginx-service

👉 This generates:

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

