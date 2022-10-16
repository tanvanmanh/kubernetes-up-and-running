# Introduction
This chapter will talk about deploying a k8s cluster. We have 2 approaches:
* Cloud-based Kubernetes services
* Locally Kubernetes

# Installing Kubernetes on a Public Cloud Provider
With cloud-based k8s services approach, we will only focus on installing k8s with GKE (Google Cloud Platform) in this chapter. Other public cloud provider (AWS, Microsoft Azure), we will only read and watch demos on the book.
## Installing Kubernetes with GKE
```bash
# Set a default zone:
gcloud config set compute/zone us-west1-a

# Create a cluster
gcloud container clusters create kuar-cluster --num-nodes=3

# When the cluster is ready, get credentials for the cluster using
gcloud container clusters get-credentials kuar-cluster
```

We can practice deploying GKE by this lab: [Deploying Google Kubernetes Engine](https://www.cloudskillsboost.google/course_sessions/1579969/labs/332280). Or we can learn about GKE through this free courese [Architecting with Google Kubernetes Engine: Foundations](https://www.cloudskillsboost.google/course_templates/127).

# Installing Kubernetes Locally Using minikube
We can install a simple single-node cluster using `minikube`(or `Docker Desktop`) if we don't want to pay for cloud resources.

# Running Kubernetes in Docker
A different approach to running a k8s cluster, uses Docker containers to simulate multitple k8s nodes instead of running everything in a virtual machine. The `kind` (Kubernetes IN Docker) project provides a great experience for launching and managing test clusters in Docker.

Install [kind](https://oreil.ly/EOgJn) on your machine, and then creating a cluster as:
```bash
kind create cluster --wait 5m
export KUBECONFIG="$(kind get kubeconfig-path)"
kubectl cluster-info
kind delete cluster
```

# The Kubernetes Client
The official k8s client is `kubectl`: a command-line tool for interacting with k8s API:
* used to manage most k8s objects
* used to explore and verify the overall health of the cluster
## Checking Cluster Status
```
# Check the version
kubectl version

# Get a simple diagnostic for the cluster
kubectl get componentstatuses
```

## Listing Kubernetes Nodes
```bash
# List out all of the nodes in cluster
kubectl get nodes
# Get more information about a specific node
kubectl describe nodes kube1
```

# Cluster Components
## Kubernetes Proxy
k8s proxy is responsible for routing network traffic to load-balanced services in the k8s cluster. k8s proxy must be present on every node in the cluster.
## Kubernetes DNS
Kubernetes also runs a DNS server, which provides naming and discovery for the services that are defined in the cluster. This DNS server also runs as a replicated service on the cluster. Depending on the size of your cluster, you may see one or more DNS servers running in your cluster.

```bash
# 
kubectl get deployments --namespace=kube-system core-dns

#
kubectl get services --namespace=kube-system core-dns

```
## Kubernetes UI
* Most of the cloud providers integrate such a visualization into the GUI for their cloud.
* Use extensions for development environments like VS Code to see the state of your cluster at a glance.