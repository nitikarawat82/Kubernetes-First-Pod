# 🚀 Kubernetes for Beginners: Create Your First Pod-Run Your First Live Application on Kubernetes
If you are starting your Kubernetes journey and don't know where to begin, you can start from this project.

In this hands-on project, we will go step by step from setting up Kubernetes on an AWS EC2 instance to creating our first Pod and running a live Nginx application inside it.

We will use Docker, Kind, kubectl, Kubernetes, and YAML to understand the basic Kubernetes workflow.

This project demonstrates how to set up a Kubernetes cluster using **Kind**, create a **Namespace**, deploy an **Nginx Pod**, and access the Nginx application running inside the Pod.

> **Note:** Nginx is used as our sample web application. Technically, Nginx is a web server that runs inside a container.
> Let's get started!


---

## 🏗️ Architecture

```text
EC2 Instance
     ↓
Docker
     ↓
Kind
     ↓
Kubernetes Cluster
     ↓
Control Plane + Worker Nodes
     ↓
Namespace
     ↓
Pod
     ↓
Nginx Container
     ↓
Nginx Web Server
```

---


## ☸️ What is Kubernetes?

**Kubernetes is a container orchestration platform used to deploy, manage, and scale containerized applications.**

In simple terms, Docker helps us run containers, while Kubernetes helps us **manage those containers and applications**.

* * *

## 🏗️ Kubernetes Architecture — Quick Overview

Kubernetes follows a **Control Plane and Worker Node** architecture.

```text
             Kubernetes Cluster
                    │
          ┌─────────┴─────────┐
          │                   │
     Control Plane        Worker Nodes
          │                /    |    \
          │              Node  Node  Node
          │                │
          │               Pods
          │                │
          │            Containers
```

### Control Plane

The **Control Plane manages the Kubernetes cluster**. It receives our commands through `kubectl` and decides what should happen in the cluster.

Important components include:

*   **API Server** → Receives and processes requests from `kubectl`.
    
*   **Scheduler** → Decides which worker node should run a new Pod.
    
*   **Controller Manager** → Makes sure the desired state of resources is maintained.
    
*   **etcd** → Stores the cluster's configuration and state.
    

### Worker Nodes

**Worker Nodes are the machines where our applications actually run.**

They run components such as:

*   **Kubelet** → Communicates with the Control Plane and manages Pods on the node.
    
*   **Container Runtime** → Runs the containers.
    
*   **Pods** → Run our application containers.
    

### Simple Flow

```text
kubectl
   ↓
API Server
   ↓
Scheduler
   ↓
Worker Node
   ↓
Pod
   ↓
Container
```

In simple terms:

> **Control Plane manages the cluster, while Worker Nodes run the applications.**

* * *

## 📦 What is a Pod?

A **Pod is the smallest deployable unit in Kubernetes**.

A **Pod is the place where our application runs in Kubernetes**.

A Pod contains one or more **containers**, and the application runs inside these containers.

So, you can simply think of a **Pod as a small environment that runs and manages our application containers**.

The basic relationship looks like this:

```text
Kubernetes
     ↓
    Pod
     ↓
 Container
     ↓
Application
```

For example:

```text
Pod
 ↓
Nginx Container
 ↓
Nginx Application
```

Kubernetes manages the **Pod**, rather than directly managing the container.

* * *

## 1️⃣ Create an EC2 Instance

Create an **Ubuntu EC2 instance** on AWS.

We will use this EC2 instance as our environment for installing Docker, Kind, kubectl, and running our Kubernetes cluster.

---

## 2️⃣ Install Docker, Kind and kubectl

Install the required tools:

* Docker → Runs the Kind Kubernetes nodes as containers
* Kind → Creates a Kubernetes cluster
* kubectl → Used to communicate with and manage the Kubernetes cluster

You can use the installation script provided in this repository.

After installation, verify:

```bash
docker --version
kind --version
kubectl version --client
```

---

## 3️⃣ Verify Docker and Fix Permissions

Check whether Docker is working:

```bash
docker ps
```

If you get a permission error, add your user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Refresh the current session:

```bash
newgrp docker
```

Then verify again:

```bash
docker ps
```

---

## 4️⃣ Create the Kubernetes Cluster Configuration

Create a file named:

```text
cluster.yml
```

This file defines how our Kind Kubernetes cluster should be created.

Our configuration contains:

* 1 Control Plane node
* 3 Worker nodes
* Port mappings for HTTP and HTTPS

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
    image: kindest/node:v1.35.1

  - role: worker
    image: kindest/node:v1.35.1

  - role: worker
    image: kindest/node:v1.35.1

  - role: worker
    image: kindest/node:v1.35.1
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
```

The complete `cluster.yml` file is available in this repository.

---

## 5️⃣ Create the Kubernetes Cluster

Use the configuration file to create the cluster:

```bash
kind create cluster --name my-cluster-1 --config=cluster.yml
```

Verify the cluster nodes:

```bash
kubectl get nodes
```

You should see **1 control-plane node and 3 worker nodes** with `Ready` status.

---

## 6️⃣ Create a Namespace

A **Namespace** helps organize and separate resources inside a Kubernetes cluster.

Create the `nginx` namespace:

```bash
kubectl create namespace nginx
```

Verify:

```bash
kubectl get namespaces
```

---

## 7️⃣ Create the Pod Configuration

Create a file named:

```text
pod.yml
```

This YAML file defines the configuration of our Pod.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: my-first-pod
  namespace: nginx

spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

The configuration tells Kubernetes to:

* Create a Pod named `my-first-pod`
* Create it inside the `nginx` namespace
* Run an Nginx container
* Use the `nginx:latest` image
* Configure the container to use port `80`

---

## 8️⃣ Create the Pod

Apply the YAML configuration:

```bash
kubectl apply -f pod.yml
```

You should see:

```text
pod/my-first-pod created
```

---

## 9️⃣ Verify the Pod

Check the Pod status:

```bash
kubectl get pods -n nginx
```

Expected output:

```text
NAME            READY   STATUS    RESTARTS   AGE
my-first-pod    1/1     Running   0          10s
```

If the status is `Running`, our Pod is successfully running.

---

## 🔟 Access the Nginx Application

To verify that Nginx is actually serving the application, we can temporarily forward a port from the EC2 instance to the Pod:

```bash
kubectl port-forward pod/my-first-pod 8080:80 -n nginx
```

This connects **EC2 port 8080** to **port 80 inside the Nginx Pod**.

You can test it from the EC2 instance:

```bash
curl http://localhost:8080
```

If Nginx is running correctly, you will receive the Nginx welcome page HTML.

To stop port forwarding:

```text
Ctrl + C
```

> Stopping port forwarding does not delete or stop the Pod. It only closes the temporary connection.

---

## 📚 What We Learned

Through this project, we learned:

* Kubernetes cluster creation using Kind
* Control Plane and Worker Nodes
* Kubernetes Namespaces
* Pod configuration using YAML
* Running an Nginx container inside a Pod
* Managing Kubernetes resources using `kubectl`
* Verifying Pods
* Accessing a Pod using port forwarding

### 🔜 What's Next?

After understanding Pods, the next Kubernetes concepts to explore are:

```text
Pod
 ↓
Deployment
 ↓
ReplicaSet
 ↓
Service
 ↓
StatefulSet
 ↓
DaemonSet
 ↓
Job
 ↓
CronJob
```

---

## 📖 Detailed Tutorial

This README provides the practical steps to run the project.

For a **detailed explanation of each step, Kubernetes architecture, YAML configuration, and the concepts behind this practical**, refer to my detailed blog:👉 [My Blog](https://techwithnitika.hashnode.dev/kubernetes-for-beginners-create-your-first-pod-run-your-first-live-application-on-kubernetes)

👉 **Kubernetes for Beginners: Create Your First Pod — Run Your First Live Application on Kubernetes**
