# Application Environment, Configuration, and Security Intro
## Using CRDs (Custom Resource Definitons)
### What is a Custom Resource?
A **Custom resource** is an extension to the Kubernetes API. It allows you to define your own custom Kubernetes object types and iteract with them, just as you would with other Kubernetes objects.
- A CustomResourceDefinition (CRD) defines a custom resource.
- You can create custom controllers that act upon custom resources.
`beehives.yml`
```
apiVersion: apiextension.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: beehives.acloud.guru
spec:
  group: acloud.guru
  names:
    plural: beehives
    singular: beehive
    kind:BeeHive
    shortNames:
    - hive
  scopes: Namescoped
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Scehma:
        type: object
        properties
          spec:
            type: object
            properties
              supers:
                type: integer
              bees:
                type: intger
```
`test-beehive.yaml`
```
apiVersion: acloud.guru/v1
kind: BeeHive
metadata:
  name: test-beehive
spec:
  supers: 3
  bees: 60000
```
---
## Using ServiceAccounts
### What is a service account?
A ServiceAccount allows process within containers to authicate with the Kubernetes API server. They can be assigned permissions via role-based access control, just like regular accounts.
### Tokens
A **token** is the credential used to access the API using the ServiceAccount.
The token for a Pod's ServiceAccount is automatically mounted to each container.
### ServiceAccount Example
`my-sa.yaml`
```
apiVeresion: v1
kind: ServiceAccount
metadata:
  name: my-sa
automountServiceAccountToken: true
```
`sa-pod.yaml`
```
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
spec:
  serviceAccountName: my-sa
  containers:
  - name: busybox
    image: radial/busybox:curl
    command: ['sh','-c','while true; do curl -s --header "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt https://kubernetes/api/v1/namespaces/default/pods; sleep 5; done']
```
---

## Understanding Kubernetes Auth
- **Authentication**: Is the user who they claim to be?
- **Authentication**: Does the user have permissions to do this?

### Users vs ServiceAccounts
| Users                                                               | ServiceAccounts                                                 |
| ------------------------------------------------------------------- | --------------------------------------------------------------- |
| Not represented by a Kubernetes object.                             | Represented by a Kubernetes object.                             |
| Usually authenticates using a client certificate.                   | Usually authenticates using an API token.                       |
| Usually used by human users or tools running outside the container. | Usually used by automated processes running within the cluster. |

- Both can receive authorization using role-based access control (RBAC)
### What is Role-Based Access Control (RBAC) ?
**Role-based access control** is a system for managing authorization. It allows you to define what users, or groups of users, are allowed to do within  the Kubernetes cluster.
`list-pods-role.yaml`
```
apiVersion: rbca.authorization.k8s.io/v1
kind: Role
metadata:
  name: list-pods-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
```
Now we need to attach this role to accounts. 
`list-pods-role-binding.yaml`
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: list-pods-rb
subjects:
- kind: ServiceAccount
  name: my-sa
  namespace: default
roleRef:
  kind: Role
  name: list-pods-role
  apiGroup: rbac.authorization.k8s.io
```
---

## Exploring Admission Control
### What is an Admission Controller?
**Admission controllers** intercept requests to the Kubernetes API after authentication and authorization, but before anu objects are persisted. They can be used to validate, deny, or even modify the request.
  
---
## Managing Compute Resource Usage
### Resource Requests vs Limits
| Requests                                                                                | Limits                                                                                                                   |
| --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Provides Kubernetes with an idea of how many resources a container is expected to use.  | Provides an upper limit on how many resources a container is allowed to use.                                             |
| The cluster uses this information to select a node that has enough resources available. | The cluster uses this information to terminate the container process if it attempts to use more than the allowed amount. |

#### Example
`simple-pod.yaml` using a request and a limit.
```
apiVersion: v1
kind: Pod
metadata:
  name: resources-pod
  namespace: resources-test
spec:
  containers:
  - name: busybox
    image: busybox:stable
    command: ['sh', '-c', 'while true; do sleep 10; done']
    resources:
      requests:
        memory: 64Mi
        cpu: 250m
      limits:
        memory: 128Mi
        cpu: 500m
```

### What is a Resource Quota? 
A **Resource Quota** is a Kubernetes object that sets limits on the resources within a Namespace. If creating or modifying a resource would go above this limit, the request will be denied. 

#### Example
`resources-test-quota.yaml`
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: resources-test-quota
  namespace: resources-test
spec:
  hard:
    requests.memory: 128Mi
    requests.cpu: 500m
    limits.memory: 256Mi
    limits.cpu: "1"
```
---
## Configuring Applications with ConfigMaps and Secrets

### What is a ConfigMap?
A **ConfigMap** is an object that stores configuration data for applications in the form of key-value pairs. This data can then be passed to containers at runtime.

### What is a Secret?
A **Secret** is similar to a ConfigMap. It stores data that can be passed to container, but it is designed to store sensitive data such as passwords or API tokens. 

### 2 ways to pass ConfigMaps and Secret data to containers...

| Volume Mounts                                              | Environment Variables                                                       |
| ---------------------------------------------------------- | --------------------------------------------------------------------------- |
| Data appears in the container's file system at runtime.    | Data appears as environment variables visible to the container at runetime. |
| Each top-level key in the config data becomes a file name. | You can specify keys and variable names for each piece of data.             |

#### Examples
`configmap.yaml`
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  message: Hello World!
  app.cfg: |
    key1=value1
    key2=value2
```
`cm-pod.yaml` which reads $MESSAGE using env variable and app.cfg using a volume.
```
apiVersion: v1
kind: Pod
metadata:
  name: cm-pod
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox:stable
    command: ["sh", "-c", "echo $MESSAGE; cat /config/app.cfg"]
    env:
    - name: MESSAGE
      valueFrom:
        configMapKeyRef:
          name: my-configmap
          key: message
    volumeMounts:
    - name: config
      mountPath: /config
      readOnly: true
  volumes:
  - name: my-configmap
    configMap:
      name: my-configmap
      items:
      - key: app.cfg
        path: app.cfg
```
### Using Secrets
We must base64 encode our sensitive data...
```
echo Secret Stuff! | base64
> U2VjcmV0IFNOdWZmIQo=
``` 
`my-secret.yaml`
```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  sensitive.data: U2VjcmV0IFNOdWZmIQo=
  passwords.txt: <another base64>
```
using the secrets... `secret-pod.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox:stable
    command: ["sh", "-c", "echo $SENSITIVE_STUFF; cat /config/passwords.txt]
    env:
    - name: SENSITIVE_STUFF
      valueFrom:
        configMapKeyRef:
          name: my-secret
          key: sensitive.data
    volumeMounts:
    - name: secret-config
      mountPath: /config
      readOnly: true
  volumes:
  - name: secret-configmap
    secret:
      secretName: my-secret
      items:
      - key: passwords.txt
        path: passwords.txt
```
---
## Configuring SecurityContext for Containers
### What is a SecurityContext?
A **security context** is part of the Pod and Contianer spec. It allows you to control advanced security-related settings for contianers. 

### Examples
**These are a few things you can control using security context.**
1. **User ID and Group ID**: You can customize the user ID (UID) and/or the group ID (GID) used by the container process. 
2. **Allow Privilege Escalation**: You can enable or disable privilege esacaltion for the container process.
3. **Read-Only Root Filesystem**: You can set the container's root filesystem to read-only. 

`security-context-pod.yaml`
```
apiVersion: v1
kind: Pod
metadata:
  name: securitycontext-pod
spec:
  containers:
  - name: busybox
    image: busybox:stable
    command: ['sh', '-c', 'while true; do echo Running...' sleep 5; done']
  securityContext:
    runAsUser:3000
    runAsGroup: 4000
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
```
```
kubectl exec securitycontext-pod -- id
> uid=3000 gid=4000
```