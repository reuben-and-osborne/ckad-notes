# Application Design and Build
---
## Building Container Images
### What is a container image?
An image is a lightweight, standalone file that contains the software and executables needed to run a container.
The image can then be used to run multiple containers. 

### Building an image
```
docker build -t tag:version .
docker run --rm --name name -d -p 8080:80 tag:version
```
### Saving image locally
```
docker save -o <path to .tar file> tag:version
```
---
## Running Jobs and CronJons
Jobs are designed to run a containerised task successfully to completion. 
CronJobs run jobs periodically according to a schedule. 
```
kubectl get jobs
```
### Job Example
```
apiVersion: batch/v1
kind: Job
metadata: 
  name: my-job
spec:
  template:
    spec:
      containers:
      - name: print
        image: busybox:stable
        command: ["echo", "This is a test!"]
      restartPolicy: Never
    backoffLimit: 4
    activeDeadlineSeconds: 10
```
```
kubectl apply -f my-job.yml
```
### CronJob Example
```
apiVersion: batch/v1
kind: CronJob
metadata: 
  name: my-cronjob
spec:
  schedule: "*/1 * * * *" 
  jobTemplate:
    spec:
      containers:
      - name: print
        image: busybox:stable
        command: ["echo", "This is a test!"]
      restartPolicy: Never
    backoffLimit: 4
    activeDeadlineSeconds: 10
```
```
kubectl apply -f my-cronjob.yml
```

- The restartPolicy for a Job or a CronJob must be OnFailure or Never.
- Use activeDeadlineSeconds in the Job spec to terminate the Job if it runs too long.

---

## Building Multi-Container Pods

Multi-container pods are Pods that contain multiple containers that work together.

### Sidecar
A sidecar container performs a task to assist the main container.

### Ambassador 
An Ambassador container proxies network traffic to and/or from the main container.

### Adapter
An Adapter container transforms the main container's output in some way.

### When should I use Multi-Container Pods?
Only use multi-container pods when the containers need to be tightly coupled, sharing resources such as network and storage volumes. 

---

## Init Containers

An init container is a container that runs to complete a task before a Pod's main container starts up.
- Separate Image - perform start up tasks using software the main container doesn't have or need.
- Delay startup - delay main container until certain conditions are met.
- Security - 'sensitive startups' like consuming secrets. 
---
## Volumes 
A volume provides external storage for containers outside the container file system. 

### volumes
- defined in the pod spec
- where and how data is stored

### volumeMounts
- defined in the container spec
- attaches the volume to a specific container
- defines the path where the volume data will appear at runtime

### Volume Types 
**Volume type** determines where and how data storage is handled.

**hostPath** - specific location directly on the host file system, on the Kubernetes node where the Pod is running. 
*emptyDir* - automatically managed location on the host file system. Data is deleted if the pod is deleted.

**persistentVolume**
- Abstract volume storage away from Pods and treat storage like a consumable resource. 
- **PersistentVolume**
  - defines an abstract storage resource ready to be consumed by pods
  - defines details about the type and amount of storage provided 
- **PersistentVolumeClaim**
  - defines a request for a storage, including details on the type of storage needed.
  - automatically binds to an available Persistentvolume that meets the provided requirements.
  - Mounted in a Pod like any volume. 

1. Create a Persistent Volume
2. Make a Persistent Volume Claim 
3. Use volumes and volumeMounts in the Pod spec
   
```
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```








