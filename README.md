# Kubernetes Learning Journey 🚀

This repository contains practical exercises and examples for learning Kubernetes fundamentals. Each section focuses on specific Kubernetes concepts with hands-on YAML configurations and kubectl commands.

## 📁 Project Structure

```
k8s-learning/
├── 01-basic-kubernetes/       # Basic Pods, Deployments & ReplicaSets
├── 02-database-apps/          # Database Pods & Web Applications  
├── 03-frontend-services/      # Frontend Applications with Services
├── 04-voting-microservices/   # Complex Multi-Tier Voting Application
└── README.md                  # This file
```

## 🛠️ Essential kubectl Commands

### Creating Resources

```bash
# Create resources with tracking and save config
kubectl create -f <file.yaml> --save-config --record

# Create from directory
kubectl create -f /path/to/directory/ --save-config --record

# Create specific resource types
kubectl create deployment <name> --image=<image> --save-config --record
kubectl create service nodeport <name> --tcp=80:80 --save-config --record
```

### 🚀 Efficient Multi-Resource Commands

```bash
# Get multiple resource types at once (save time!)
kubectl get deployments,services,pods
kubectl get pods,services,endpoints
kubectl get all,configmaps,secrets

# Monitor multiple resources simultaneously
kubectl get deployments,pods --watch

# Get detailed info with multiple columns
kubectl get pods,services -o wide

# Describe multiple related resources
kubectl describe deployments,replicasets <name>
```

### Monitoring Deployments

```bash
# Watch deployment rollout status
kubectl rollout status deployment/<deployment-name>

# Watch deployment creation in real-time
kubectl get deployments --watch

# Get deployment history
kubectl rollout history deployment/<deployment-name>

# Monitor pods creation
kubectl get pods --watch

# Monitor all resources
kubectl get all --watch
```

### Accessing Containers

```bash
# Execute commands in a container
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- /bin/sh

# Execute in specific container (multi-container pod)
kubectl exec -it <pod-name> -c <container-name> -- /bin/bash

# Run temporary debugging pod
kubectl run debug --image=busybox -it --rm -- /bin/sh
```

### Finding Database IPs and Network Information

```bash
# Get pod IP addresses
kubectl get pods -o wide

# Describe pod to see detailed network info
kubectl describe pod <pod-name>

# Get service endpoints
kubectl get endpoints

# Get service details with cluster IP
kubectl get services -o wide

# Get cluster DNS information
kubectl exec -it <pod-name> -- nslookup kubernetes.default

# Test connectivity to database
kubectl exec -it <pod-name> -- ping <db-pod-ip>
kubectl exec -it <pod-name> -- telnet <service-name> <port>
```

### Resource Management

```bash
# Get all resources  
kubectl get all
kubectl get all --all-namespaces

# Get multiple resource types at once
kubectl get deployments,services,pods
kubectl get pods,services,endpoints

# Describe resources for detailed info
kubectl describe <resource-type> <resource-name>
kubectl describe pod <pod-name>
kubectl describe service <service-name>

# View resource YAML
kubectl get <resource-type> <resource-name> -o yaml

# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # Multi-container pods
kubectl logs -f <pod-name>  # Follow logs

# Port forwarding for local access
kubectl port-forward pod/<pod-name> <local-port>:<container-port>
kubectl port-forward service/<service-name> <local-port>:<service-port>
```

### Debugging and Troubleshooting

```bash
# Check cluster info
kubectl cluster-info

# Get node information
kubectl get nodes -o wide
kubectl describe node <node-name>

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Get resource usage
kubectl top nodes
kubectl top pods

# Debug failed pods
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous  # Previous container logs
```

### Working with Namespaces

```bash
# List namespaces
kubectl get namespaces

# Create namespace
kubectl create namespace <namespace-name>

# Set default namespace context
kubectl config set-context --current --namespace=<namespace-name>

# Get resources in specific namespace
kubectl get all -n <namespace-name>
kubectl get pods,services -n <namespace-name>

# Compare resources across namespaces
kubectl get pods --all-namespaces
kubectl get services,endpoints --all-namespaces
```

### Scaling and Updates

```bash
# Scale deployments
kubectl scale deployment <deployment-name> --replicas=<number>

# Check scaling progress
kubectl get deployments,pods --watch

# Update image
kubectl set image deployment/<deployment-name> <container-name>=<new-image>

# Monitor rollout
kubectl rollout status deployment/<deployment-name>

# Rollback deployment
kubectl rollout undo deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name> --to-revision=<revision-number>

# Check rollout history
kubectl rollout history deployment/<deployment-name>
```

### Cleanup Commands

```bash
# Delete resources
kubectl delete -f <file.yaml>
kubectl delete pod <pod-name>
kubectl delete service <service-name>
kubectl delete deployment <deployment-name>

# Delete all resources in namespace
kubectl delete all --all -n <namespace-name>

# Force delete stuck resources
kubectl delete pod <pod-name> --grace-period=0 --force
```

## 🎯 Learning Sections

### [01 - Basic Kubernetes](./01-basic-kubernetes/README.md)
- **Focus**: Core Kubernetes resources
- **Story**: Deploy your first pod, scale with deployments, understand ReplicaSets
- **Key Skills**: Resource creation, basic scaling, container access

### [02 - Database Apps](./02-database-apps/README.md)  
- **Focus**: Stateful applications
- **Story**: Deploy MySQL database, connect applications, manage data
- **Key Skills**: Environment variables, database connectivity, debugging

### [03 - Frontend Services](./03-frontend-services/README.md)
- **Focus**: External access and networking
- **Story**: Deploy frontend apps, expose them to the world via services
- **Key Skills**: Service discovery, external access, load balancing

### [04 - Voting Microservices](./04-voting-microservices/README.md)
- **Focus**: Complete microservices architecture
- **Story**: Build a full voting app with multiple connected services
- **Key Skills**: Microservices, namespaces, inter-service communication

## 🚀 Quick Start

1. **Clone and navigate to the repository**
   ```bash
   cd "k8s (geek university)"
   ```

2. **Start with Basic Kubernetes**
   ```bash
   cd 01-basic-kubernetes
   kubectl create -f pods/pod.yaml --save-config --record
   ```

3. **Monitor your first pod**
   ```bash
   kubectl get pods --watch
   ```

4. **Access your pod**
   ```bash
   kubectl exec -it nginx-pod -- /bin/bash
   ```

## 📚 Additional Resources

- [Official Kubernetes Documentation](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/)

## 🏷️ Best Practices

- Always use `--save-config --record` when creating resources
- Use combined commands like `kubectl get deployments,pods` for efficiency
- Monitor deployment rollouts with `kubectl rollout status`
- Use `kubectl describe` for debugging issues  
- Access containers with `kubectl exec -it` for troubleshooting
- Check pod IPs and service endpoints for network debugging
- Use `kubectl get all` to see the complete picture quickly
- Use namespaces to organize complex applications
- Clean up resources after learning exercises

## ⚡ Pro Tips

- **Combine resource types**: `kubectl get deployments,services,pods` instead of 3 separate commands
- **Watch multiple resources**: `kubectl get deployments,pods --watch` to see scaling in real-time
- **Use shortcuts**: `kubectl get deploy,svc,pods` (deploy=deployments, svc=services)
- **Get wide output**: `kubectl get pods,services -o wide` for more detailed info

---

**Happy Learning!** 🎓 Start with basic concepts in `01-basic-kubernetes` and progress through each section to master Kubernetes fundamentals.
