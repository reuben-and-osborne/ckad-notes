# Services and Networking

## Kubernetes Networking
The Kubernetes cluster uses a *virtual network* to allows Pods to communitate with one another seamlessly, even if they are on different nodes.

## What is a NetworkPolicy?
- A **NetworkPolicy** is a Kubernetes object that allows you to restrict network traffic to and from Pods within the cluster network.
- Use NetworkPolicies to block unnecessary or unexpected network traffic and make your applications more secure.

| Non-Isolated Pods                                                               | Isolated Pods                                                                            |
| ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Any Pod that is not selected by any NetworkPolicies                             | Any Pod that is selected by at least 1 NetworkPolicy                                     |
| Open to all incoming and outgoing network traffic (NetworkPolicies don't apply) | Only open to traffic that is explicitly allowed by a NetworkPolicy that selects the Pod. |

### Example 
`deny-ingress.yaml` deny all ingress traffic to the pods in the test namespace
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-test-default-deny-ingress
  namespace: test
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

`test-allow.yaml` allow inbound traffic on pods that have the label app = test-app. Allow traffic from pods that have the label `label2 = value2`, and are in a namespace that has the label `label1 = value1`.
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-test-client-allow
  namespace: test
spec:
  podSelector:
    matchLabels:
      app: test-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespasceSelector:
      matchLabels:
        labe1: value1 
      podSelector:
        matchLabels:
          label2: value2
    ports:
    - protocol: TCP
      ports: 80
```
---
## Exploring Services
### What is a Service? 
- A Service allows you to expose an application running across multiple Pods to the network.
- Clients can communicate with the Service, and traffic will be automatically routed to one of the underlying Pods.

#### ClusterIP
A ClusterIP Service exposes the application within the cluster network where it can be accessed by other Pods.

#### NodePort
A NodePort Service exposes the application externally by listening on an external port on each cluster node/

### Example
#### ClusterIP
`clusterip-service.yaml` this is a service that maps port 8080 to port 80 on all pods in the cluster with the label `app = service-server`
```
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  selector:
    app: service-server
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
```
`curl <pod ip>:8080`

#### NodePort
`nodeport-service.yaml` allows access from outside the cluster on port 30080.
```
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: service-server
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
    nodePort: 30080
```
`curl localhost:30080`

---
## Exposing Applications with Ingress
### What is an Ingress?
- An **Ingress** is a Kubernetes object that manages access to Services from outside the cluster.
- For example, you can use an Ingress to provide your end users with access to your web application running Kubernetes. 

#### Ingress Routing
- An Ingress routes traffic to a Service, which then routes the traffice to a Pod.
- You need an Ingress controller to actually implement the Ingress functionality. 

### Exmaple
`test-service.yml`
```
apiVersion: v1
kind: Service
metadata:
  name: test-service
spec:
  type: ClusterIP
  selector:
    app: test
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```
`test-ingress.yml`. ingressClassName tells the Ingress what IngressController to use. This example is fronted by nginx, to serve traffic. 
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: ingresstest.acloud.guru
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: test-service
            port:
              number: 80
```
Now we must install an ingress controller, we can do so using helm:
```
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo updates
kubectl create namespace nginx-ingress
helm install nginx-ingress nginx-stable/nginx-ingress -n nginx-ingress
```