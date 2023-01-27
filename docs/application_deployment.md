## What is a deployment? 
A **deployment** defines a desired state for a set of replica Pods. Kubernetes constatly works to maintain that desired state by creating, deleting and replacing those Pods. 

A deployment manages multiple replica Pods using a Pod templated. The **Pod template** is the shared configuration used by the Deployment to create new replicas. 

### Example
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
### To **scale** a deployment:
1. Use scale:
```
kubectl scale deployment/nginx-deployment --replicas=4
```
2: Edit deployment:
```
kubectl edit deployment nginx-deployment
```

## Rolling Update
A **rolling update** allows you to change a Deployment's Pod template, gradually replacing replicas with zero downtime.

**rollout status**:
Check the status of a rollout using
```
kubectl rollout status
```

**undo rollout**: 
Undo to the previous deployment.
```
kubectl rollout undo <deployment>
```

## Deploying with Blue/Green and Canary Strategies 

What is a **Deployment Strategy**?
A Deployment strategy is a method of rolling out new code that is used to achieve some benefit, such as increasing reliability and minimizing risk.

### Blue/Green Deployment. 
- 2 identical production environments, called blue and green.
- new code is rolled out to the second env. 
- This env is confirmed to be stable and working before traffic is directed there.

### Canary Deployment
- again uses 2 environments
- a portion of the user base is directed to the new code in order to expose any issues
- then the changes are rolled out to everyone else

## Helm 
**Helm** is a package managment tool for applications that run in Kubernetes. It allows you to easily install software in your cluster, alongside the necessary Kubernetes configuration. 

### Helm Charts
- A **Helm Chart** is a Helm software packages. It contains all the Kubernetes resource definitions needed to get the application up and running in the cluster.

###
- A **Helm Repository** is a collection of available Charts. You can use it to browse and download Charts vefore installing them in your cluster.  