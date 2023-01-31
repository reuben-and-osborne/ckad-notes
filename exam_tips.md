# Building Container Images
1. Images are files that include all of the software needed to run a container.
2. A Dockerfile defines the contents of an image.
3. The `docker build` command builds an image using a Dockerfile.

# Jobs and CronJobs
1. A Job is designed to run a containerised task successfully to completion.
2. CronJobs run Jobs periodically according to a schedule. 
3. The *restartPolicy* for a Job or CronJob Pod must be *OnFailure* or *Never*.
4. Use *activeDeadlineSeconds* in the Job spec to terminate the Job if it runs too long.
 
# Multi-Container Pods
1. A sidecar contianer performs some task that helps the main container.
2. An ambassador container proxies network traffic to and/or from the main container.
3. An adapter container transforms the main containers output. 

# Init Containers
1. Init containers run to completion before the main containers startup. 
2. Add init containers using the *initContainers* field of the Pod spec. 

# Volumes: 
1. The *volumes* field in the Pod spec defines details about volumes used in the Pod.
2. the *volumesMounts* field in the container spec mounts a volume to a specific container at a specific location. 
3. *hostPath* volumes mount data from a specific location on the host (k8s node).
4. *hostPath* volume types:
   - *Directory* - Mounts an existing directory on the host.
   - *DirectoryOrCreate* - Mounts a directory on the host, and creates it if it doesn't exist.
   - *File* - Mounts an existing single file on the host.
   - *FileOrCreate* - Mounts a file on the host, and creates it if it doesn't exist.
5. *emptyDir* volumes provide temporary storage that uses the host file system, and are removed if the Pod is deleted.
   
# Persistent Volume: 
1. A PersistentVolume defines a storage resource. 
2. A PersistentVolumeClaim defines a request to consume a storage resource.
3. PersistentVolumeClaims automatically bind to a PersistentVolume that meet their criteria. 
4. Mount a PersistentVolumeClaim to a container like a regular volume. 

# Deployments:
1. A Deployment actively manges a desired state for a set of replica Pods.
2. The Pods templates provides the Pod configuration that the Deployment will use to create new Pods.
3. The replica field sets the number of replicas. You can scale up or down by changing this value.

# Rollout Update:
1. A rolling update gradually rolls out changes to a Deployment's Pod templates by gradually replacing replicas with new ones.
2. Use `kubectl rollout status` to check the status of a rolling update.
3. Roll back to the latest rolling update with: `kubectl rollout undo`. 

# Deployment Straregies
1. Use multiple Deplyoments to set up blue/green environments in Kubernetes
2. Use labels and selectors on Services to direct user traffic to different Pods.
3. A simple way to set up a canary environment in Kubernetes is to use a Service that selects Pods from 2 different Deployments. Vary the number of replicas to direct fewer users to the Canary environment.

# Helm
1. Helm is a package management tool for Kubernetes applications.
2. Helm Charts are packages that contain all of the resource definitions needed to get an application up and running in a cluster.
3. A Helm Repository is a collection of Charts and a source for browsing and downloading them.

# API Deprecation
1. API deprecation is the process of announcing changes to an API early, giving users time to update their code and/or tools.
2. Kubernetes removes support for deprecated APIs that are in GA (general availability) only after 12 months or 3 Kubernetes releases, whichever is longer.

# Probes and Health Checks:
1. Liveness probes check if a container is healthy so that it can be restarted if it is not.
2. Readiness probes check whether a container is fully started up and ready to be used.
3. Probes can run a command inside the container, make an HTTP request, or attempt a TCP socket connection to determine container status. 
   
# Monitoring:
1. The Kubernetes Metrics API provides metric data about container performance.
2. You can view Pod metrics using `kubectl top pod`. 

# Logging:
1. Standard/error output for containers is stored in the container log.
2. You can view the container log using `kubectl logs`
3. For multi-container Pods, use the `-c` to specify which container's logs you want to view.

# Debugging:
1. Use `kubectl get pods` to check the status of all Pods in a Namespace. Use the `--all-namespace` flag if you don't knoe what Namespace to look in.
2. Use kubectl describe to get detailed information about Kubernetes objects.
3. Use `kubectl logs` to retrieve container logs.
4. Check cluster-level logs if you still cannot located any relevant information.

# Custom Resources (CRD)
1. Custom Resources are extensions of the Kubernetes API.
2. A CustomResourceDefinition defines a custom resource.

# ServiceAccounts:
1. ServiceAccounts allow processes within containers to authenticate with the Kubernetes API server.
2. You can set the Pod's ServiceAccount with serviceAccountName in the Pod spec.
3. The Pod's ServiceAccount token is automatically mounted to the Pod's containers.

# Kubernetes Auth:
1. Normal users usually authenticate using client certificates, while ServiceAccounts usually use tokens. 
2. Authorization for both normal users and ServiceAccounts can be managed using role-based access control (RBAC).
3. Roles and ClusterRoles define a specific set of permissions
4. RoleBindings and ClusterRoleBindings tie Roles or ClusterRoles to users/serviceAccounts.
# Addmission Control:
1. Admission controlles intercept requests to the Kubernetes API and can be used to validate and/or modify them.
2. You can enable addmission controllers using the `--enable-admission-plugins` flag or kube-apiserver.
# Compute Resource Management
1. A resource request informs the cluster of the expected resoruce usage for a container. It is used to select a Node that has enough resources available to run the pod. 
2. A resource limit sets an upper limit on how many resources a container can use. IF the container process attempts to go above this limit, the container process will be terminated.
3. A ResourceQuota limits the amount of resoruces that can be used within a specific Namespace. If a user attempts to create or modify objects in that Namespace such that the quota would be exceeded, the request will be denied. 

# Application Configuration
1. A ConfigMap stores configuration data that can be passed to containers. 
2. A Secret is designed to store sensitive configuration data such as passwords or API keys.
3. Data from both ConfigMaps and Secrets can be passed into containers using either a volume mount or environment variable.

# Security Context
1. A container's security context allows you to control advanced security related settings for the container.
2. Set the container's user ID (UID) and group ID (GID) with `securityContext.runAsUser` and `securityContext.runAsGroup`.
3. Enable or disable privilege escalation with `securityContext.allowPrivilegeEscalation`.
4. Make the container root filesystem read-only with `ssecurityContext.readOnlyRootFilesystem`.

# Network Policies
1. If a Pod is not selected by any NetworkPolicies, the Pod is non-isolated, and all traffic is allowed.
2. If a Pod is selected by any NetworkPolicy, traffic will be blocked unless it is allowed by at least 1 NetworkPolicy that selects the Pod. 
3. If you combine a namespaceSelector and a podSelector within the same rule, the traffic must meet both the Pod- and Namespace- related conditions in order to be allowed.
4. Even if a NetworkPolicy allows outgoing traffic from the source Pod, NetworkPolicies could still block the same traffic when it is incoming to the destination Pod. 

# Services
1. Services allow you to expose an application running in multiple Pods.
2. ClusterIP Services expose the Pods to other applications within the cluster.
3. NodePort Services expose the Pods externally using a port that listens on every node in the cluster.

# Ingress
1. An Ingress manages external access to Kubernetes applications.
2. An Ingress routes to 1 or more Kubernetes Services.
3. You need an Ingress controller to implement the Ingress functionality. Which controller you use determines the specifics of how the Ingress will work.
