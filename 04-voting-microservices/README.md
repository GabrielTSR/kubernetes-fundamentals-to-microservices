# 04 - Voting Microservices: Complete Application Architecture

## 🎯 The Story
Time for the real deal! You'll deploy a complete voting application with 5 microservices, databases, message queues, and multiple namespaces. This is how real applications work in production.

---

## 📖 Chapter 1: Understanding the Architecture

**The Mission**: Explore what we're building

```
                    🗳️  Voting Application Architecture 🗳️

┌─────────────┐                                    ┌─────────────┐
│    Vote     │                                    │   Result    │
│   (Web UI)  │                                    │   (Web UI)  │
│  Port 80    │                                    │  Port 80    │
└─────────────┘                                    └─────────────┘
       │                                                    ▲
       │ 1. User votes                          4. Read results │
       ▼                                                    │
┌─────────────┐    2. Store vote    ┌─────────────┐    3. Read data
│    Redis    │◄──────────────────► │   Worker    │──────────────┐
│   (Queue)   │                     │ (Processor) │              │
│  Port 6379  │                     │             │              ▼
└─────────────┘                     └─────────────┘     ┌─────────────┐
                                                        │ PostgreSQL  │
                                                        │ (Database)  │
                                                        │ Port 5432   │
                                                        └─────────────┘

📊 Data Flow:
1. Users vote on the Vote web interface
2. Votes are queued in Redis
3. Worker processes votes from Redis → PostgreSQL
4. Result web interface displays data from PostgreSQL
```

**What we're building**: A complete microservices voting application with separate frontend, queue, processor, and database components.

💡 **Security Note**: This application uses Kubernetes Secrets to store the PostgreSQL password securely, following best practices for sensitive data.

---

## 📖 Chapter 2: Setting Up the Namespace

**The Mission**: Organize everything in its own space

```bash
# Create the vote namespace
kubectl create -f namespaces/vote.yaml --save-config --record

# Create PostgreSQL secret (stores database password safely)
kubectl create -f secrets/postgres-secret.yaml

# See all namespaces
kubectl get namespaces

# Set this as our working namespace
kubectl config set-context --current --namespace=vote

# Verify we're in the right namespace
kubectl config view --minify | grep namespace
```

**What happened?** Namespaces provide isolation and organization for related resources.

---

## 📖 Chapter 3: Deploy the Data Layer

**The Mission**: Set up Redis (queue) and PostgreSQL (database)

```bash
# Deploy Redis first
kubectl create -f deployments/redis.yaml --save-config --record
kubectl create -f services/redis.yaml --save-config --record

# Deploy PostgreSQL database
kubectl create -f deployments/db.yaml --save-config --record
kubectl create -f services/db.yaml --save-config --record

# Watch everything come up
kubectl get pods --watch
# Press Ctrl+C when both are running

# Check our data services
kubectl get services
kubectl get pods -o wide
```

**What happened?** You deployed the backend data services that will store votes and queue processing jobs.

---

## 📖 Chapter 4: Deploy the Application Services

**The Mission**: Deploy the vote frontend, worker, and results services

```bash
# Deploy the voting web interface
kubectl create -f deployments/vote.yaml --save-config --record
kubectl create -f services/vote.yaml --save-config --record

# Deploy the result web interface
kubectl create -f deployments/result.yaml --save-config --record
kubectl create -f services/result.yaml --save-config --record

# Deploy the worker (processes votes from Redis to PostgreSQL)
kubectl create -f deployments/worker.yaml --save-config --record

# Watch the application come together
kubectl get pods --watch
# Press Ctrl+C when all are running

# See the complete architecture
kubectl get all

# Or get specific components together
kubectl get deployments,services,pods -o wide
```

**What happened?** You deployed a complete microservices architecture with 5 different components.

---

## 📖 Chapter 5: Testing the Voting Application

**The Mission**: Use the application and watch data flow

```bash
# Get the voting interface URL
kubectl get service vote
# Note: It's exposed on NodePort 30000

# Get your node IP
kubectl get nodes -o wide

# Open the voting app in your browser
# http://<node-ip>:30000

# While voting, monitor the worker logs
kubectl logs -f deployment/worker

# Check what's happening in Redis
kubectl exec -it deployment/redis -- redis-cli
# Inside Redis:
KEYS *                    # See vote keys
LLEN db                   # Check queue length
EXIT

# Check the PostgreSQL database
kubectl exec -it deployment/db -- psql -U postgres
# Inside PostgreSQL:
\l                        # List databases
\c postgres               # Connect to postgres db
\dt                       # List tables
SELECT * FROM votes;      # See the votes!
\q                        # Quit
```

**What happened?** Votes flow from web UI → Redis → Worker → PostgreSQL → Results web UI.

---

## 📖 Chapter 6: Service Communication Deep Dive

**The Mission**: Understand how services communicate

```bash
# Check how services find each other
kubectl exec -it deployment/vote -- env | grep -i db
kubectl exec -it deployment/vote -- env | grep -i redis

# Test Redis connectivity from vote service
kubectl exec -it deployment/vote -- nc -zv redis 6379

# Test database connectivity from worker
kubectl exec -it deployment/worker -- nc -zv db 5432

# See the service discovery in action
kubectl exec -it deployment/vote -- nslookup redis
kubectl exec -it deployment/worker -- nslookup db

# Check endpoints for each service
kubectl get endpoints
```

**What happened?** Services use environment variables and DNS to find each other across the microservices architecture.

---

## 📖 Chapter 7: Scaling the Application

**The Mission**: Scale different parts of the application based on load

```bash
# Scale the vote frontend for more users
kubectl scale deployment vote --replicas=3

# Scale the worker for faster vote processing
kubectl scale deployment worker --replicas=2

# Watch the scaling of deployments and pods together
kubectl get deployments,pods --watch
# Press Ctrl+C when done

# Test that voting still works with multiple instances
# Go back to http://<node-ip>:30000 and vote more

# Check that all workers are processing
kubectl logs deployment/worker --tail=10

# Scale back down
kubectl scale deployment vote --replicas=1
kubectl scale deployment worker --replicas=1
```

**What happened?** Different microservices can be scaled independently based on their load characteristics.

---

## 📖 Chapter 8: Troubleshooting Microservices

**The Mission**: Learn to debug when things go wrong

```bash
# Check the health of all services
kubectl get pods
kubectl get services
kubectl get endpoints

# Look for problems in logs
kubectl logs deployment/vote --tail=20
kubectl logs deployment/worker --tail=20
kubectl logs deployment/result --tail=20

# Test connectivity between services
kubectl exec -it deployment/vote -- wget -O- http://result

# Check resource usage across the application
kubectl top pods

# Simulate a failure and recovery
kubectl delete pod -l app=worker
kubectl get pods --watch
# Press Ctrl+C when replacement is running

# Verify the application still works
```

**What happened?** Microservices require monitoring multiple components and understanding their dependencies.

---

## 📖 Chapter 9: Understanding Namespaces in Practice

**The Mission**: See how namespaces provide isolation

```bash
# Check what's in our vote namespace
kubectl get all -n vote

# Compare with the default namespace
kubectl get all -n default

# Try to access vote service from default namespace
kubectl run test --image=busybox -n default -it --rm -- /bin/sh
# Inside the test pod:
nc -zv vote.vote.svc.cluster.local 5000    # Full DNS name required!
exit

# Switch back to vote namespace
kubectl config set-context --current --namespace=vote
```

**What happened?** Namespaces provide resource isolation and require full DNS names for cross-namespace communication.

---

## 🧹 Cleanup Time

```bash
# Clean up the entire voting application
kubectl delete -f services/
kubectl delete -f deployments/
kubectl delete -f secrets/
kubectl delete -f namespaces/

# Switch back to default namespace
kubectl config set-context --current --namespace=default

# Verify cleanup
kubectl get all -n vote
kubectl get namespaces

# Or check everything at once
kubectl get all,namespaces
```

---

## 🎓 What You Learned

1. **Microservices Architecture** - How multiple services work together
2. **Service Communication** - Services find each other via DNS and environment variables
3. **Data Flow** - Understanding how data moves through a complex application
4. **Namespaces** - Organizing and isolating related resources
5. **Independent Scaling** - Different services scale based on their needs
6. **Debugging Distributed Systems** - Troubleshooting across multiple components
7. **Production Patterns** - Real-world application deployment strategies

## 🔑 Key Architecture Patterns

- **Frontend Services**: vote, result (user-facing web interfaces)
- **Backend Services**: redis, db (data storage)
- **Worker Services**: worker (background processing)
- **Service Mesh**: All services communicate via Kubernetes DNS
- **Horizontal Scaling**: Each service scales independently

## 🏆 Congratulations!

You've completed the Kubernetes learning journey! You now understand:
- ✅ Basic Kubernetes resources (Pods, Deployments, Services)
- ✅ Database integration and stateful applications
- ✅ Service networking and external access
- ✅ Complete microservices architecture
- ✅ Namespace organization and isolation
- ✅ Scaling and troubleshooting distributed applications

## 🚀 Next Steps

Ready for production? Consider learning about:
- **Persistent Volumes** for data that survives pod restarts
- **ConfigMaps and Secrets** for secure configuration management
- **Ingress Controllers** for advanced routing and SSL termination
- **Helm Charts** for packaging and deploying complex applications
- **Monitoring and Logging** with Prometheus and Grafana
- **Security Policies** and RBAC for production deployments
