# Local 3-Node Kubernetes Cluster & Application Deployment

A hands-on DevOps project demonstrating how to containerize a web application and deploy it onto a multi-node Kubernetes cluster locally using KinD (Kubernetes in Docker).

This setup mirrors production Amazon EKS workflows—including multi-node pod scheduling, internal service load balancing, and container orchestration—without incurring cloud infrastructure costs.

# Cluster Architecture

                     +-------------------------------------------------------+
                                     |
                     |  (Browser: localhost:8080 <-> kubectl port-forward)   |
                     +---------------------------+---------------------------+
                                                 |
                                                 v
+-------------------------------------------------------------------------------------------------+
| KinD Cluster: "eks-cluster"                                                                         |
|                                                                                                 |
|   +---------------------------------+                                                           |
|   |   Control-Plane Node            |                                                           |
|   |   (eks-cluster-control-plane)      |                                                           |
|   |   - API Server & Etcd           |                                                           |
|   +---------------------------------+                                                           |
|                                                                                                 |
|   +-----------------------------------------------------------------------------------------+   |
|   | Kubernetes Service: "hello-kubernetes-service" (ClusterIP / Port 80)                    |   |
|   +--------------------------------------------+--------------------------------------------+   |
|                                                |                                                |
|                        +-----------------------+-----------------------+                        |
|                        |                                               |                        |
|                        v                                               v                        |
|   +----------------------------------------+      +----------------------------------------+   |
|   | Worker Node 1: "eks-cluster-worker"      |      | Worker Node 2: "eks-cluster-worker2"     |   |
|   |                                        |      |                                        |   |
|   |  [ Pod 1 ]         [ Pod 2 ]           |      |  [ Pod 3 ]         [ Pod 4 ]           |   |
|   |  hello-k8s         hello-k8s           |      |  hello-k8s         hello-k8s           |   |
|   +----------------------------------------+      +----------------------------------------+   |
+-------------------------------------------------------------------------------------------------+


# Tech Stack

Orchestration: Kubernetes

Containerization: Docker Desktop / Engine

CLI Tools: kubectl, kind, git

Sample Application: hello-kubernetes (Node.js web app)

OS Environment: Linux / Ubuntu (WSL2)



 # Step-by-Step Implementation:

1. Provision the 3-Node Kubernetes Cluster

We use KinD to spin up a local Kubernetes cluster containing 1 Control Plane Node and 2 Worker Nodes:

kind create cluster --name eks-demo --config kind-config.yaml


Verify that all three nodes are healthy and running:

kubectl get nodes


2. Containerize the Application

Build the Docker container image locally from the application source directory:

cd hello-kubernetes/src/app
docker build -t hello-kubernetes:v1 .


3. Load the Image into Cluster Nodes

Transfer the local Docker image directly into the KinD node containers (bypassing the need for a remote cloud registry like Amazon ECR):

kind load docker-image hello-kubernetes:v1 --name eks-cluster


4. Deploy Kubernetes Manifests

Apply the Deployment and Service configurations:

kubectl apply -f k8s-manifests/


Verify that 4 pod replicas are distributed across both worker nodes (eks-cluster-worker and eks-cluster-worker2):

kubectl get pods -o wide


5. Access the Application & Test Load Balancing

Forward a port from your workstation to the Kubernetes internal service:

kubectl port-forward svc/hello-kubernetes-service 8080:80


Open your web browser and navigate to:
http://localhost:8080

Note: Refreshing the page dynamically updates the Pod name displayed on screen, demonstrating active internal round-robin load balancing across all 4 replicas.

# Key Concepts Demonstrated:

Multi-Node Cluster Management: Configured control-plane and worker node topology locally using declarative YAML specs.

Local Registry Ingestion: Successfully transferred local Docker images into KinD nodes using kind load.

High Availability & Replication: Scaled pod replicas across multiple worker nodes to prevent single-point-of-failure scenarios.

Service Load Balancing: Used Kubernetes ClusterIP services to abstract pod networking and distribute incoming web requests.

Declarative Infrastructure: Defined all Kubernetes resources (Deployment, Service, Cluster) in version-controlled YAML files.

# Cleanup:

To delete the cluster and release all associated CPU/RAM resources:

kind delete cluster --name eks-demo

