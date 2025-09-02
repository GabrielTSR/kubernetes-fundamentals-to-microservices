# 02 - Database Apps: Running Stateful Applications

## 🎯 The Story
Time to get serious! You'll deploy a MySQL database, connect to it from other pods, and learn how applications talk to each other in Kubernetes.

---

## 📖 Chapter 1: Deploying Your Database

**The Mission**: Get MySQL running in your cluster

```bash
# Deploy MySQL with environment configuration
kubectl create -f pods/mysql.yaml --save-config --record

# Watch it start up (databases take a moment)
kubectl get pods --watch
# Press Ctrl+C when it's running

# Get the database details
kubectl get pods -o wide
kubectl describe pod mysql-pod

# Check if MySQL is actually running
kubectl logs mysql-pod

# See the environment variables we configured
kubectl exec -it mysql-pod -- env | grep MYSQL
```

**What happened?** You deployed MySQL with a root password and created a database called "geek".

---

## 📖 Chapter 2: Connecting to Your Database

**The Mission**: Access MySQL and explore the database

```bash
# Jump into the MySQL pod
kubectl exec -it mysql-pod -- /bin/bash

# Connect to MySQL (password is 'password')
mysql -u root -p

# Explore what we have
SHOW DATABASES;
USE geek;
SHOW TABLES;

# Create a test table and add some data
CREATE TABLE users (id INT, name VARCHAR(50));
INSERT INTO users VALUES (1, 'Alice'), (2, 'Bob');
SELECT * FROM users;

# Exit MySQL
EXIT;

# Exit the pod
exit
```

**What happened?** You created data inside your database. But how do other apps connect to it?

---

## 📖 Chapter 3: Pod-to-Pod Communication

**The Mission**: Connect to MySQL from another pod

```bash
# First, get your MySQL pod's IP address
kubectl get pod mysql-pod -o wide
# Note the IP address (something like 10.244.0.X)

# Create a temporary client pod to test connectivity
kubectl run mysql-client --image=mysql:latest -it --rm -- /bin/bash

# Inside the client pod, connect to MySQL using the IP
mysql -h <mysql-pod-ip> -u root -p
# Password: password

# Check our data is still there
USE geek;
SELECT * FROM users;
EXIT;

# Test network connectivity
ping <mysql-pod-ip>
nc -zv <mysql-pod-ip> 3306

# Exit (this pod will be automatically deleted)
exit
```

**What happened?** Pods can talk to each other using their IP addresses, but these IPs change when pods restart!

---

## 📖 Chapter 4: Database Debugging and Monitoring

**The Mission**: Learn to troubleshoot database issues

```bash
# Check if MySQL is responding
kubectl exec -it mysql-pod -- mysql -u root -p -e "SELECT 1"

# Monitor MySQL processes
kubectl exec -it mysql-pod -- ps aux | grep mysql

# Check MySQL is listening on the right port
kubectl exec -it mysql-pod -- netstat -tlnp | grep 3306

# Monitor resource usage
kubectl top pod mysql-pod

# Check MySQL configuration
kubectl exec -it mysql-pod -- cat /etc/mysql/my.cnf

# Check available disk space
kubectl exec -it mysql-pod -- df -h

# See real-time logs
kubectl logs mysql-pod -f
# Press Ctrl+C to stop following logs
```

**What happened?** You learned to diagnose database health and performance.

---

## 📖 Chapter 5: Understanding Pod Networking

**The Mission**: Explore how pod networking works

```bash
# Create a debug pod with network tools
kubectl run debug --image=busybox -it --rm -- /bin/sh

# Inside the debug pod, explore networking
nslookup kubernetes.default    # DNS works!
wget -O- http://kubernetes.default/api/v1/namespaces

# Try to reach MySQL (use the IP you noted earlier)
nc -zv <mysql-pod-ip> 3306
ping <mysql-pod-ip>

# Check your own network details
ip addr show
route -n

# Exit the debug pod
exit
```

**What happened?** Every pod gets its own IP and can reach other pods directly.

---

## 📖 Chapter 6: Data Persistence Reality Check

**The Mission**: Understand what happens to data when pods restart

```bash
# Add more data to your database
kubectl exec -it mysql-pod -- mysql -u root -p

# Inside MySQL:
USE geek;
INSERT INTO users VALUES (3, 'Charlie'), (4, 'Diana');
SELECT * FROM users;
EXIT;

# Now delete the pod (simulating a crash)
kubectl delete pod mysql-pod

# Check what happened
kubectl get pods

# Try to access the data (spoiler alert: it's gone!)
kubectl create -f pods/mysql.yaml --save-config --record
kubectl exec -it mysql-pod -- mysql -u root -p

# Inside MySQL:
USE geek;
SHOW TABLES;    # Empty! Data is lost
EXIT;
```

**What happened?** Without persistent storage, data disappears when pods are deleted!

---

## 🧹 Cleanup Time

```bash
# Clean up the MySQL pod
kubectl delete -f pods/mysql.yaml

# Verify it's gone
kubectl get pods
```

---

## 🎓 What You Learned

1. **Environment Variables** configure applications in pods
2. **Pod IPs** allow direct communication between containers
3. **kubectl exec** is essential for debugging applications
4. **Database connectivity** works the same as traditional networking
5. **Data persistence** requires special storage (volumes) that we haven't covered yet
6. **Network troubleshooting** uses standard tools like ping, nc, and telnet

## ⚠️ Important Lessons

- Pod IPs change when pods restart - don't hardcode them!
- Data in pods is ephemeral - it disappears when pods die
- Always test connectivity and troubleshoot with kubectl exec

## 🚀 Next Step

Ready to solve the connectivity problem? Head to [03-frontend-services](../03-frontend-services/README.md) to learn how Services provide stable networking!