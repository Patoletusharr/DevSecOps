# 05 - Kubernetes (Kind + Docker Desktop + Git Bash)

## Overview

This guide covers the Kubernetes fundamentals using a local Kind (Kubernetes in Docker) cluster on Windows. By the end of this lab, you will understand how Kubernetes workloads communicate, how RBAC controls access, and how Network Policies restrict traffic between Pods.

**Environment**
- Docker Desktop
- Kind
- kubectl
- Git Bash

---

# Step 1 - Verify Installation

Verify Docker, kubectl and Kind.

```bash
docker version
kubectl version --client
kind version
```

Create a Kubernetes cluster.

```bash
kind create cluster --name k8-demo
```

Verify the cluster.

```bash
kubectl cluster-info
kubectl get nodes
```

---

# Step 2 - Namespaces

Namespaces logically separate workloads inside a Kubernetes cluster.

Create a namespace.

```bash
kubectl create namespace payments
```

List namespaces.

```bash
kubectl get ns
```

---

# Step 3 - Pods

Create application Pods inside the namespace.

Example:

```bash
kubectl run frontend --image=nginx -n payments
kubectl run backend --image=nginx -n payments
```

List Pods.

```bash
kubectl get pods -n payments
kubectl get pods -A
kubectl get pods -A -o wide
```

Describe a Pod.

```bash
kubectl describe pod backend -n payments
```

Delete a Pod.

```bash
kubectl delete pod backend -n payments
```

---

# Step 4 - Deployments

Deployments provide self-healing and scaling for Pods.

Create a Deployment.

```bash
kubectl create deployment backend --image=nginx -n payments
```

Scale the Deployment.

```bash
kubectl scale deployment backend --replicas=3 -n payments
```

View Deployments.

```bash
kubectl get deploy -A
```

---

# Step 5 - Services

Services provide a stable endpoint for Pods.

Expose the backend Deployment.

```bash
kubectl expose deployment backend \
  --port=80 \
  --target-port=80 \
  --name=backend-svc \
  -n payments
```

Verify the Service.

```bash
kubectl get svc -n payments
```

Verify Service Endpoints.

```bash
kubectl get endpoints backend-svc -n payments
```

Expected Output:

```
backend-svc   10.x.x.x:80
```

---

# Step 6 - Testing Kubernetes DNS

Service names are only resolvable **inside the Kubernetes cluster**.

The following command **will not work** from Windows or Git Bash.

```bash
curl http://backend-svc
```

Expected Error

```
Could not resolve host: backend-svc
```

Create a temporary Pod for testing.

```bash
kubectl run curl \
--image=curlimages/curl \
-it \
--rm \
--restart=Never \
-n payments \
-- sh
```

Inside the Pod run:

```bash
curl http://backend-svc
```

or

```bash
wget -qO- backend-svc
```

Expected Output

```
Welcome to nginx!
```

This confirms that:

- Kubernetes DNS is working.
- Service discovery is working.
- Traffic is reaching the backend Pod.

---

# Step 7 - Service Accounts

Create a Service Account.

```bash
kubectl create serviceaccount payments-user -n payments
```

Verify.

```bash
kubectl get sa -n payments
```

---

# Step 8 - RBAC (Role Based Access Control)

Create a Role that allows reading Pods.

```bash
kubectl apply -f role.yaml
```

Create a RoleBinding to assign the Role to the Service Account.

```bash
kubectl apply -f rolebinding.yaml
```

Verify.

```bash
kubectl get role -n payments
kubectl get rolebinding -n payments
```

Check permissions.

```bash
kubectl auth can-i list pods \
--as=system:serviceaccount:payments:payments-user \
-n payments
```

Expected Output

```
yes
```

Try an unauthorized action.

```bash
kubectl auth can-i delete pods \
--as=system:serviceaccount:payments:payments-user \
-n payments
```

Expected Output

```
no
```

This demonstrates that RBAC allows only the permissions granted by the Role.

---

# Step 9 - Network Policies

Network Policies control communication between Pods.

Apply the policy.

```bash
kubectl apply -f networkpolicy.yaml
```

Verify.

```bash
kubectl get networkpolicy -n payments
```

Describe the policy.

```bash
kubectl describe networkpolicy allow-specific-traffic -n payments
```

---

# Step 10 - Testing Network Policies

Deploy a frontend Pod that is allowed to communicate.

```bash
kubectl run frontend \
--image=curlimages/curl \
--labels="role=frontend" \
-it \
--rm \
--restart=Never \
-n payments \
-- sh
```

Inside the Pod test connectivity.

```bash
curl http://backend-svc
```

Expected Result

```
Connection Successful
```

Now deploy another Pod without the required label.

```bash
kubectl run attacker \
--image=curlimages/curl \
-it \
--rm \
--restart=Never \
-n payments \
-- sh
```

Test the connection.

```bash
curl http://backend-svc
```

Expected Result

```
Connection timed out
```

or

```
Connection refused
```

This verifies that the Network Policy only allows traffic from Pods matching the configured labels.

> **Note:** Network Policies are enforced only when your Kubernetes cluster uses a CNI plugin that supports them. If you're using a default Kind cluster without a NetworkPolicy-capable CNI, the policy object will be created but traffic may still be allowed.

---

# Useful Commands

View all resources.

```bash
kubectl get all -A
```

View Pods.

```bash
kubectl get pods -A
```

View Services.

```bash
kubectl get svc -A
```

View Deployments.

```bash
kubectl get deploy -A
```

View Events.

```bash
kubectl get events -A
```

View Logs.

```bash
kubectl logs <pod-name>
```

Open a shell inside a Pod.

```bash
kubectl exec -it <pod-name> -- sh
```

Delete resources.

```bash
kubectl delete -f filename.yaml
```

Delete the Kind cluster.

```bash
kind delete cluster --name k8-demo
```

---

# Git Bash vs PowerShell

Git Bash supports Linux here-documents.

```bash
kubectl apply -f - <<EOF
...
EOF
```

PowerShell equivalent.

```powershell
@"
...
"@ | kubectl apply -f -
```

---
