# Application Observability and Maintenance
## Understanding the API Deprecation Policy
### Kubernetes API
 The **Kubernetes API** is the primary interface for Kubernetes. Running a command like `kubectl get pods` talks to the API on the control plane. 
---
 ## Deprecation
 **Deprecation** is the process of providing advanced warning of changes to an API, giving consumers of that API time to update their code and/or processes.

 ### K8s Deprecation Policy Highlights
 1. **apiVersion** - The apiVersion field on a Kubernetes object indicates which version of the API it is designed to be compatible with.
 2. **Deprecation Window** - Once deprecated, GA (general availability) released API versions are supported for 12 months or 3 releases (whichever is longer).
 3. Migration Guide - Check the Deprecated aPI Migration Guide for details on API changes with each version of Kubernetes. 
---
## Implementing Probes and Health Checks
### What is a Probe?
Probes are part of the container spec in Kubernetes. They allow you to customize how Kubernetes detects the state of a container.

### Types of Probes:
1. **Liveness** - Liveness probes check whether a container is healthy so that it can be restarted if it becomes unheathly.
2. **Readiness** - Readiness probes determine when a container is fully started up and ready to receive user traffic. 
3. **Startup** - Startup probes check contianer health during startup for slow-starting containers.

### Liveness Example
Check whether our busy box container can respond to a simple echo command.
```
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - name: busybox
    image: busybox:stable
    command: ['sh', '-c', 'while true; do sleep 10; done']
    livenessProbe:
      exec:
        command: ['echo', 'health check!']
      initialDelaySeconds: 5
      periodSeconds: 5
```

### Readiness Example:
Check whether our nginx container is ready by making a http GET request. 
```
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.20.1
    ports:
    - containerPort: 80
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 5
```
---
## Monitoring Kubernetes Applications
### What is monitoring? 
Monitoring is the process of gathering data (metrics) about the performance of your containerized applications.

**Steps to Access Data with the Metrics API:**
- **Install Metrics Server**: The is a component that gathers the metric data so that we can access it. Without Metrics Server, the Metrics API will not provide any data.
- **Access metric data**: The `kubectl top` command can be used to view metric data once it is gathered by the Metrics server.

---
## Accessing Container Logs
### Container Logging
Kubernetes stores the stdout/stderr console output for each container in the container log.
You can then view the log with the `kubectl logs <Pod name>` command.

### Multi-Container logging
For pods with multiple containers running, use the `-c` flag to specify the container. `kubectl logs <pod name> -c <container name>`

---
## Debugging in Kubernetes
### The Debugging Process:
1. Locate the problem. Identify which component(s) may not be working.
2. Gather information about the broken component to help determine what is wrong.
3. Make changes to fix the issue.

### Kubernetes Debugging Tips:
1. Check Object Status: Check the basic status of objects. This can be quick way to locate a broken Pod in the cluster.
2. Check Object Configuration: Use `kubectl describe` and/or view the full object manifest to understand the relevant object(s) in more detail. 
3. Check Logs: Check container logs, and even cluster-level logs for more insight into what is going on. 


- Kube API server logs: `sudo cat /var/log/containers/kube-apiserver-k8s-control_kube_system_kube-apiserver-<uid>.log`
- Kubelet logs: `sudo journalctl -u kubelet`
---
