# Chapter 4: Common `kubectl` Commands
The `kubectl` command-line utility is a tool that use to create objects and interact with the Kubernetes API.

## Namespaces
Kubernetes uses `namespaces` to organize objects in the cluster; each namespace as a folder that holds a set of objects.
```bash
# kubectl --namespace
kubectl --namespace=mystuff

# short-hand: -n
# interact with all namespaces: --all-namespaces
```

## Contexts
```bash
# Create a context with a different default namespace
kubectl config set-context my-context --namespace=mystuff
kubectl config use-context my-context 
```
Contexts can also be used to manage different clusters or different users for authenticating to those clusters using the `--users` or `--clusters` flags with the `set-context` command.

## Viewing Kubernetes API Objects
```bash
# Get a listing of all resources in the current namespace
kubectl get <resource-name>

# Get a specific resource
kubectl get <resource-name> <obj-name> [--no-headers]
```
To get more information, add following flags:
1. -o wide flag
2. -o json flag
3. -o yaml flag

Extracting specific fields from the object
```bash
kubectl get pods my-nginx -o jsonpath --template={.status.conditions} 
```

View multiple objects of different types
```bash
kubectl get pods,services
```
Use the `describe` command to get more detailed information about a particular object
```bash
kubectl describe <resource-name> <obj-name>
```

Use the `explain` command to see a list of supported fields for each supported type of k8s object
```bash
kubectl explain pods
```
Add the `--watch` flag to any `kubectl get` command to continuously monitor the state of particular resource.

## Creating, Updating, and Destroying Kubernetes Object
Objects in the Kubernetes API are represented as JSON or YAML files. These files are either returned by the server in response to a query or posted to the server as part of an API request. You can use these YAML or JSON files to create, update, or delete objects on the Kubernetes server.

```bash
# Use kubectl to create, modify objects in Kubernetes
kubectl apply -f obj.yaml
```
The `apply` command also records the history of previous configurations in an annotation within the object. You can manipulate these records with the
* `edit-last-applied`
* `set-last-applied`
* `view-last-applied`
```bash
kubectl apply -f myobj.yaml view-last-applied
```
When you want to delete an object
```bash
kubectl delete -f obj.yaml
kubectl delete <resource-name> <obj-name>
```

## Labeling and Annotating Objects
Labels and annotations are tags for your objects; you can update the labels and annotations on any Kubernetes object using the `label` and `annotate` commands.
```bash
kubectl label pods bar color=red --overwire
kubectl label pods bar color- # Remove the color label
```

## Debugging Commands
`kubectl` also makes a number of commands available for debugging your containers.
```bash
# See the logs for a running container
# -f continuosly stream the bogs back to the terminal without exiting
# -c choose the container to view
kubectl logs <pod-name>

# Execute a command in a running container
kubectl exec -it <pod-name> -- bash

# Send input the running process
kubectl attact -it <pod-name>

# Copy files to and from a container
kubectl cp <pod-name>:</path/to/remote/file> </path/to/local/file>

# Access Pod via the network: forward network traffic from the local machine to the Pod
kubectl port-forward <pod-name> 8080:80
kubectl port-forward services/<service-name> 8080:80

# View k8s events
# --watch stream events
# -A see events in all namespace
kubectl get events

# See how your cluster is using resources: CPU, memory, percentage of available resources
kubectl top nodes

# Show all Pods and their resouce usage
# --all-namespaces
kubectl top pds
```

## Cluster Management
The `kubectl` tool can also be used to manage the cluster itself.
* `Cordon` a node: prevent future Pods from being scheduled onto that machine.
* `Drain` a node: remove any Pods that are currently running on that machine.
Example: removing a physical machine for repairs or upgrades. Once the machine is repaired, you can use `kubectl uncordon` to re-enable Pods scheduling onto the node.
## Command Autocompletion
kubectl supports integration with your shell to enable tab completion for both commands and resources. Depending on your environment, you may need to install the bash-completion package before you activate command autocompletion.


## Alternative Ways of Viewing Your Cluster
In addition to `kubectl`, there are other tools for interacting with your Kubernetes cluster:
* Visual Studio Code
* IntelliJ
* Eclipse

Most oth managed Kubernetes service feature a graphical interface to k8s intergrated into their web-based user experience. K8s in the public cloud alse integrates with sophisticated monitoring tools, that can help you gain insights into how your applications are running.
