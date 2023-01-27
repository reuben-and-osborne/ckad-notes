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

# Probes:
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


