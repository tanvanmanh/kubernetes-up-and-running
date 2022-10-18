# Chapter 5: Pods
## Pods in Kubernetes
* A Pod is a collection containers and volumes running in the same execution environment.
* Pods, not container, are the smallest deployable artifact in a Kubernetes cluster.
* Each container within a Pod runs in its own cgroup, but they share a number of Linux namespaces.
* Application running in the same Pod share the same IP address and port space, and can communicate using native inteprocess communication channels. However, applications in different Pods are isolated from each other; they have different IP address, hostnames, and more.
* Containers in different Pods running on the same node might as well be on different servers.

## Thinking with Pods
In general, the right question to ask yourself when designing Pods is “Will these containers work correctly if they land on different machines?” If the answer is no, a Pod is the correct grouping for the containers. If the answer is yes, using multiple Pods is probably the correct solution.

## The Pod Manifest
Pods are described in a Pod `manifest`, which is just a text-file representation of the k8s API object.
Kubernetes strongly believes in `declarative configuration`, which means that you write down the desired state of the world in a configuration file and then submit that configuration to a service that takes actions to ensure the desired state becomes the actual state.

> Declarative configuration has numerous advantages
> * Enabling code review configuration
> * Documenting the current state of the system for distributed teams.
> * It is the basis for all the self-healing behaviors in k8s that keep applications running without user action.

Pod deploying workflow:
> 1. The k8s API server accepts and processes Pod manifests before storing them in persistent storage (`etcd`)
> 2. The scheduler uses the k8s API to find Pods that haven't been scheduled to a node.
> 3. Then, it places the Pods onto nodes depending on the resources and other constaints expressed in the Pod manifests.
> 4. The k8s scheduler tries to ensure that Pods from the same application are distributed onto different machines for reliability in the presence of such failures.

### Creating a Pod
The simplest way to create a Pod is via the imperative `kubectl run` command.
```bash
kubectl run kuard --generator=run-pod/v1 --image=gcr.io/kuar-demo/kuard-amd64:blue

# See the status of Pods
kubectl get pods

# Delete Pods
kubectl delete pods/kuard
```
### Creating a Pod Manifest
You can write Pod manifests using YAML or JSON, but YAML is generally preferred because it is slightly more human-editable and support comments.
Pod manifests include a couple of key fields and attributes: namely, a `metadata` section for describing the Pod and its labels, a `spec` section for describing volumes, and a list of containers that will run in the Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

## Running Pods
Use the `kubectl apply` command to launch a single instance of kuard
```bash
kubectl apply -f kuard-pod.yaml
```
The Pod manifest will be submitted to the Kubernetes API server. The Kubernetes system will then schedule that Pod run on a healthy node in the cluster, where the `kubelet` daemon will monitor it.

### Listing Pods
```bash
# List all Pods running in the cluster
# -o wide print out slightly more information
# -o json print out the complete objects in JSON
# -o yaml print out the complete objects in YAML
kubectl get pods
```

### Pod Details
Use the `kubectl describe` command to find out more information about a Pod
```bash
kubectl describe pods kuard
```

### Deleting a Pod
```bash
# Delete a Pod by name
kubectl delete pods/kuard

# Delete a Pod by the same file that used to create it
kubectl delete -f kuard-pod.yaml
```
When a Pod is deleted, it is not immediately killed. All Pods have a termination grace period (default 30s). The grace period is important for reliability because it allows the Pod to finish any active requests that it may be in the middle of processing before it terminated.

> When you delete a Pod, any data stored in the containers associated with that Pod will be deleted as well.

## Accessing Your Pod
You're going to want to access the Pod for a variety of reasons:
* Load the web service that is running in the Pod
* Want to view its logs to debug a problem.
* Execute other commands inside the Pod

### Getting More Information with Logs
The `kubectl logs` command downloads the current logs from the running instance:
```bash
# -f flag cause the logs to stream continuously
# --previous flag will get logs from a previous instance of the container.
kubectl logs kuard
```

### Copying Files to and from Containers

## Health Checks
When you run your application as a container in Kubernetes, it is automatically kept alive for you using a process health check. This health check simply ensures that the main process of your application is always running. If it isn’t, Kubernetes restarts it.
However, in most cases, a simple process check is insufficient. To address this, Kubernetes introduced health checks for application liveness. Since these liveness health checks are application-specific, you have to define them in your Pod manifest.

### Liveness Probe
Liveness probes are defined per container, which means each container inside a Pod is health checked separately.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:blue
    name: kuard
    livenessProbe:
      httpGet:
        path: /healthy
        port: 8080
      initialDelaySeconds: 5 # will not be called until 5s aflter all the containers in Pod are created
      timeoutSeconds: 1 # respond within 1s (200 < status < 400)
      periodSeconds: 10 # will call the probe every 10s
      failureThreshold: 3 # the container will fail and restart after 3 consecutive probes fail
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```
Create a Pod using this manifest and then port-forward to that Pod:
```bash
kubectl apply -f kuard-pod-health.yaml
kubectl port-forward kuard 8090:8080
```
### Readiness Probe
Kubernetes makes a distinction between `liveness` and `readiness`. `Liveness` determines if an application is running properly. Containers that fail `liveness` checks are restarted. `Readiness` describes when a container is ready to serve user requests. Containers that fail `readiness` checks are removed from service load balancers. `Readiness` probes are configured similarly to liveness probes.

### Startup Probe
Startup probes have recently been introduced to Kubernetes as an alternative way of managing slow-starting containers. When a Pod is started, the startup probe is run before any other probing of the Pod is started.
The startup probe proceeds until it either times out (in which case the Pod is restarted) or it succeeds, at which time the liveness probe takes over.
Startup probes enable you to poll slowly for a slow-starting container while also enabling a responsive liveness check once the slow-starting container has initialized.

### Advanced Probe Configuration
Probes in Kubernetes have a number of advanced options:
* How long to wait after Pod startup to start probing
* How many failures should be considered a true failure
* How many successes are necessary to reset a failure count

### Other Types of Health Checks
* `tcpSocket` health checks that open a TCP socket: useful for non-HTTP applications, such as databases or other non-HTTP based APIs.
* `exec` probes: execute a script or program in the context of the container. `exec` scripts are often useful for custom application validation logic that doesn't fit neatly into an HTTP call.

## Resource Management
Kubernetes allows users to specify two different resource mtrics:
1. Resource `requests` specify the minumum amount of a resource required to run the application
2. Resource `limits` specify the maximum amount of a resource that an application can consume.

### Resource Requests: Minimum Required Resources
When a Pod requests the resources required to run its containers, Kubernetes guarantees that these resources are available to the Pod. The most commonly requested resources are CPU and memory, but Kubernetes supports other resource types as well, such as GPUs.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

> Resources are requested per container, not per Pod. The total resources requested by the Pod is the sum of all resources requested by all containers in the Pod.

Requests are used when scheduling Pods to nodes. The Kubernetes scheduler will ensure that the sum of all requests of all Pods on a node does not exceed the capacity of the node. Therefore, a Pod is guaranteed to have at least the requested resources
when running on the node.

Since resource requests guarantee resource availability to a Pod, they are critical to ensuring that containers have sufficient resources in high-load situations.

### Capping Resourse Usage with Limits
Set a maximum on a its resource usage via resource `limits`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```
When you establish limits on a container, the kernel is configured to ensure that consumption cannot exceed these limits.

## Persisting Data with Volumes
When a Pod is deleted or a container restarts, any and all data in the container’s filesystem is also deleted. 
### Using Volumes with Pods
To add a volume to a Pod manifest, there are two new stanzas to add to our configuration.
1. A new `spec.volumes` section: this array defines all of the volumes that may be accessed by containers in the Pod manifest.
2. The `volumeMounts` array in ther container definition: this array defines the volumes that are mounted into a particular container and the path where each volume should be mounted.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  volumes:
    - name: "kuard-data"
      hostPath:
        path: "/var/lib/kuard"
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      volumeMounts:
        - mountPath: "/data"
          name: "kuard-data"
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

### Different Ways of Using Volumes with Pods
#### Communication/synchronization
`emptyDir`: is scoped to the Pod's lifespan, but it can be shared between two containers, forming the basis for communication between our Git sync and web serving containers.
#### Cache
An application may use a volume that is valuable for performance, but not required for correct operation of the application.
For example, perhaps the application keeps prerendered thumbnails of larger images. Of course, they can be reconstructed from the original images, but that makes serving the thumbnails more expensive. You want such a cache to survive a container restart due to a health-check failure, and thus emptyDir works well for the cache use case as well.
#### Persistent data
Sometimes you will use a volume for truly persistent data—data that is independent of the lifespan of a particular Pod, and should move between nodes in the cluster if a node fails or a Pod moves to a different machine. 
To achieve this, Kubernetes supports a wide variety of remote network storage volumes, including widely supported protocols like NFS and iSCSI as well as cloud provider network storage like Amazon Elastic Block Store, Azure File and Azure Disk, and Google’s Persistent Disk.
#### Mounting the host filesystem
Other applications don’t actually need a persistent volume, but they do need some access to the underlying host filesystem. For example, they may need access to the /dev filesystem to perform raw block-level access to a device on the system.
For these cases, Kubernetes supports the `hostPath` volume, which can mount arbitrary locations on the worker node into the container.

