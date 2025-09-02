# 01 - Basic Kubernetes: Your First Steps

## 🎯 The Story
Let's start your Kubernetes journey! You'll deploy your first pod, see how deployments manage your apps, and understand how Kubernetes keeps everything running.

---

## 📖 Chapter 1: Your First Pod

**The Mission**: Deploy a simple web server and explore it

```bash
# Create your first pod
kubectl create -f pods/pod.yaml --save-config --record

# Watch it come to life
kubectl get pods --watch
# Press Ctrl+C when it's running

# See where it's running and its IP
kubectl get pods -o wide

# Let's explore our creation
kubectl describe pod nginx-pod

# Jump inside and look around
kubectl exec -it nginx-pod -- /bin/bash
# Run these inside the pod:
ps aux                    # See what's running
curl localhost           # Test the web server
exit                     # Leave the pod

# Check what the pod is saying
kubectl logs nginx-pod
```

**What happened?** You created a single container running NGINX. But what if it dies?

```bash
# Let's kill it and see what happens
kubectl delete pod nginx-pod

# Check if it comes back (spoiler: it won't)
kubectl get pods
```

**Lesson**: Pods don't restart themselves. We need something smarter...

---

## 📖 Chapter 2: Enter Deployments - The Smart Manager

**The Mission**: Deploy an app that can heal itself and scale

```bash
# Deploy a frontend app with a smart manager
kubectl create -f deployments/dp.yaml --save-config --record

# Watch the deployment orchestrate pod creation
kubectl rollout status deployment/frontend-dp

# See your managed pods and deployments
kubectl get deployments,pods

# Test the self-healing magic
kubectl delete pod <pick-any-pod-name>

# Watch Kubernetes bring it back to life
kubectl get pods --watch
# Press Ctrl+C when you see the replacement

# Scale up for more traffic
kubectl scale deployment frontend-dp --replicas=5

# Watch the scaling happen
kubectl get pods --watch
# Press Ctrl+C when all are running

# Scale back down
kubectl scale deployment frontend-dp --replicas=2
kubectl get pods --watch
```

**What happened?** The deployment ensures you always have the right number of healthy pods running.

---

## 📖 Chapter 3: Understanding the Control Loop

**The Mission**: See how Kubernetes continuously monitors and fixes things

```bash
# Check the current state
kubectl get all

# See the deployment's management structure
kubectl describe deployment frontend-dp

# Let's break something and watch it get fixed
kubectl delete pod <pick-any-pod-name>

# Immediately check the replacement process
kubectl get pods
kubectl get events --sort-by=.metadata.creationTimestamp

# Try to access one of your pods
kubectl get pods -o wide
kubectl exec -it <any-pod-name> -- /bin/bash
# Inside the pod:
hostname                  # See the pod's identity
env | grep HOSTNAME      # Environment variables
ip addr show             # Network interface
exit
```

**What happened?** Kubernetes constantly monitors desired vs actual state and fixes any drift.

---

## 📖 Chapter 4: Working with ReplicaSets (Optional Deep Dive)

**The Mission**: Understand the layer between deployments and pods

```bash
# See the ReplicaSet created by your deployment
kubectl get deployments,replicasets,pods

# Look at its details
kubectl describe rs <replicaset-name>

# Create a standalone ReplicaSet
kubectl create -f replicasets/rs1.yaml --save-config --record

# Watch it create pods
kubectl get pods --show-labels

# Test its healing ability
kubectl delete pod <pick-any-pod-from-replicaset>
kubectl get pods --watch
```

**What happened?** ReplicaSets ensure a specific number of identical pods are always running.

---

## 🧹 Cleanup Time

```bash
# Clean up everything we created
kubectl delete -f deployments/dp.yaml
kubectl delete -f replicasets/rs1.yaml

# Verify everything is gone
kubectl get all
```

---

## 🎓 What You Learned

1. **Pods** are the smallest unit, but they don't restart themselves
2. **Deployments** manage pods and provide self-healing and scaling
3. **ReplicaSets** ensure the right number of pod replicas
4. **kubectl exec** lets you explore inside containers
5. **kubectl get --watch** shows real-time changes
6. **Kubernetes control loop** constantly monitors and fixes your infrastructure

## 🚀 Next Step

Ready for databases? Head to [02-database-apps](../02-database-apps/README.md) to learn how to run stateful applications!