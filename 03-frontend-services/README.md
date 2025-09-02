# 03 - Frontend Services: Exposing Apps to the World

## 🎯 The Story
You've deployed pods and databases, but they're trapped inside the cluster! Time to learn Services - the bridges that connect your apps to the outside world and to each other.

---

## 📖 Chapter 1: Deploy a Frontend Application

**The Mission**: Create a scalable frontend deployment

```bash
# Deploy the frontend application
kubectl create -f deployments/frontend-dp.yaml --save-config --record

# Watch the deployment create multiple pods
kubectl rollout status deployment/frontend-dp

# See your frontend army
kubectl get pods -o wide
kubectl get deployments

# Try to access one pod directly (spoiler: this is painful)
kubectl exec -it <any-frontend-pod> -- curl localhost
```

**What happened?** You have 6 frontend pods, but they're isolated islands with changing IP addresses.

---

## 📖 Chapter 2: The Service Solution

**The Mission**: Create a Service to provide stable access to your frontend

```bash
# Create a NodePort service to expose your frontend
kubectl create -f services/frontend-svc.yaml --save-config --record

# See what we created
kubectl get services
kubectl get services -o wide

# Get detailed service information
kubectl describe service frontend-svc

# Check the endpoints (should list all your pod IPs)
kubectl get endpoints frontend-svc
```

**What happened?** The Service created a stable IP and name that routes to your pods.

---

## 📖 Chapter 3: Testing External Access

**The Mission**: Access your app from outside the cluster

```bash
# Get your node IP (if using minikube/kind)
kubectl get nodes -o wide

# The service is exposed on port 30080 (NodePort)
# Test from your local machine:
curl http://<node-ip>:30080

# If using minikube, get the URL directly:
minikube service frontend-svc --url

# Test the service from inside the cluster
kubectl run test-pod --image=busybox -it --rm -- /bin/sh

# Inside the test pod:
wget -O- http://frontend-svc        # Service name works!
wget -O- http://frontend-svc:80     # Full service:port format
exit
```

**What happened?** Services provide both internal DNS names and external access points.

---

## 📖 Chapter 4: Load Balancing in Action

**The Mission**: See how Services distribute traffic across pods

```bash
# Scale your frontend to more replicas
kubectl scale deployment frontend-dp --replicas=3

# Watch the scaling
kubectl get pods --watch
# Press Ctrl+C when done

# Check that service endpoints updated
kubectl get endpoints frontend-svc

# Test load balancing with multiple requests
for i in {1..10}; do
  kubectl exec -it <any-pod> -- curl -s http://frontend-svc | grep "Welcome to nginx"
done
```

**What happened?** The Service automatically updated to include new pods and balances traffic across all healthy ones.

---

## 📖 Chapter 5: Service Discovery Deep Dive

**The Mission**: Understand how pods find each other using Services

```bash
# Create a client pod to explore service discovery
kubectl run client --image=busybox -it --rm -- /bin/sh

# Inside the client pod, explore DNS:
nslookup frontend-svc                    # Service DNS resolution
nslookup frontend-svc.default.svc.cluster.local  # Full DNS name

# Check what DNS gives us
dig frontend-svc

# Test connectivity to the service
nc -zv frontend-svc 80

# See all available services
nslookup kubernetes

# Exit the client
exit
```

**What happened?** Kubernetes provides built-in DNS that lets pods find services by name.

---

## 📖 Chapter 6: Service Types and Networking

**The Mission**: Understand different ways to expose services

```bash
# Look at our current service details
kubectl get service frontend-svc -o yaml

# See the different service types available
kubectl explain service.spec.type

# Check how NodePort works
kubectl get service frontend-svc
# Note the three ports: port, targetPort, nodePort

# See the service from the pod's perspective
kubectl exec -it <frontend-pod> -- netstat -tlnp

# Check iptables rules (if you're curious about the magic)
kubectl get nodes
kubectl describe node <node-name> | grep -A 10 "Non-terminated Pods"
```

**What happened?** Services use multiple networking layers to route traffic correctly.

---

## 📖 Chapter 7: Breaking Things to Learn

**The Mission**: See what happens when pods fail behind a service

```bash
# Kill a few pods and watch the service adapt
kubectl delete pod <pick-a-frontend-pod>

# Check that service still works
curl http://<node-ip>:30080

# Check endpoints updated
kubectl get endpoints frontend-svc

# Scale to zero and see what happens
kubectl scale deployment frontend-dp --replicas=0
kubectl get endpoints frontend-svc

# Try to access the service (should fail)
curl http://<node-ip>:30080

# Bring pods back
kubectl scale deployment frontend-dp --replicas=3
kubectl get endpoints frontend-svc --watch
```

**What happened?** Services automatically track healthy pods and remove failed ones from rotation.

---

## 🧹 Cleanup Time

```bash
# Clean up everything
kubectl delete -f services/frontend-svc.yaml
kubectl delete -f deployments/frontend-dp.yaml

# Verify cleanup
kubectl get all
```

---

## 🎓 What You Learned

1. **Services** provide stable networking for dynamic pods
2. **NodePort** exposes services on every node at a specific port
3. **Service Discovery** uses DNS to connect pods by name
4. **Load Balancing** distributes traffic across healthy pods automatically
5. **Endpoints** track which pods are available for a service
6. **Service Types** (ClusterIP, NodePort, LoadBalancer) provide different access patterns

## 🔑 Key Concepts

- **ClusterIP**: Default, internal access only
- **NodePort**: Exposes on all nodes at a static port (30000-32767)
- **LoadBalancer**: Cloud provider creates external load balancer
- **Service DNS**: `<service-name>.<namespace>.svc.cluster.local`

## 🚀 Next Step

Ready for a real-world application? Head to [04-voting-microservices](../04-voting-microservices/README.md) to build a complete multi-tier voting app with multiple services working together!
