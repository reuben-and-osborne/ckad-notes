# Application Environment, Configuration, and Security Intro
---
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
---