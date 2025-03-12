# Kubernetes Core Concepts

## Table of Contents
1. [Introduction](#introduction)
2. [Cluster Architecture](#cluster-architecture)
3. [Pods](#pods)
4. [Multi-Container Pods](#multi-container-pods)
5. [Replication Controllers & ReplicaSets](#replication-controllers--replicasets)
6. [Deployments](#deployments)
7. [Rolling Updates & Rollbacks](#rolling-updates--rollbacks)
8. [Namespaces](#namespaces)
9. [Services](#services)
10. [Docker vs Containerd](#docker-vs-containerd)

---

## Introduction
Kubernetes is an open-source container orchestration platform that automates application deployment, scaling, and management.

---

## Cluster Architecture
Kubernetes has a client-server architecture consisting of:

### Master Node
- Manages, plans, schedules, and monitors the cluster.

### Worker Nodes
- Host applications as containers.

### Master Node Components:
- **ETCD Cluster**
- **Kube API Server**
- **Kube Controller Manager**
- **Kube Scheduler**

### Worker Node Components:
- **Kubelet**
- **Kube Proxy**
- **Container Runtime Engine**

### ETCD
#### What is ETCD?
- ETCD is a distributed, reliable key-value store that is:
  - Simple
  - Secure
  - Fast

#### Key-Value Store vs Traditional Database
- Traditional relational databases store data in tables with predefined schemas.
- Key-value stores use simple key-value pairs.

Example key-value data:
```sh
Key: name
Value: John Doe
Key: age
Value: 45
```

#### Installing ETCD
```sh
curl -L https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz -o etcd-v3.3.11-linux-amd64.tar.gz
tar xzvf etcd-v3.3.11-linux-amd64.tar.gz
./etcd
```

#### Operating ETCD
```sh
./etcdctl set key1 value1
./etcdctl get key1  # Returns: value1
```

#### ETCD in Kubernetes
- Stores all cluster data: Nodes, Pods, Configs, Secrets, etc.
- In HA environment, multiple ETCD instances run in a cluster.

```sh
kubectl exec etcd-master -n kube-system etcdctl get / --prefix --keys-only
```

---

## Pods
- A **Pod** is the smallest deployable unit in Kubernetes.
- It can contain one or more containers that share storage, networking, and lifecycle.
- Example command to create a pod:
  ```sh
  kubectl run nginx --image=nginx
  ```

---

## Multi-Container Pods
- Pods can have multiple containers working together.
- These containers share storage and network and can communicate via localhost.
- Example commands:
  ```sh
  docker run python-app
  docker run helper --link app1
  ```

---

## Replication Controllers & ReplicaSets
- **Replication Controllers** ensure that a specified number of Pod replicas are running at any time.
- **ReplicaSets** are the newer version of Replication Controllers.
- Example YAML for a ReplicaSet:
  ```yaml
  apiVersion: apps/v1
  kind: ReplicaSet
  metadata:
    name: myapp-replicaset
  spec:
    replicas: 3
    selector:
      matchLabels:
        type: front-end
    template:
      metadata:
        labels:
          type: front-end
      spec:
        containers:
        - name: nginx-container
          image: nginx
  ```
  - Apply the configuration:
    ```sh
    kubectl apply -f replicaset-definition.yml
    ```

---

## Deployments
- A **Deployment** provides declarative updates to Pods and ReplicaSets.
- Example YAML for a Deployment:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: myapp-deployment
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: myapp
    template:
      metadata:
        labels:
          app: myapp
      spec:
        containers:
        - name: nginx-container
          image: nginx
  ```
  - Deploy using:
    ```sh
    kubectl apply -f deployment-definition.yml
    ```

---

## Rolling Updates & Rollbacks
- Rolling updates allow updating a deployment without downtime.
- Rollback to a previous version if needed.
- Commands:
  ```sh
  kubectl rollout status deployment/myapp-deployment
  kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
  kubectl rollout undo deployment/myapp-deployment
  ```

---

## Namespaces
- Namespaces are used to isolate groups of resources within a cluster.
- Useful commands:
  ```sh
  kubectl create namespace dev
  kubectl get pods --namespace=dev
  kubectl config set-context $(kubectl config current-context) --namespace=dev
  ```

---

## Services

### Introduction
- Kubernetes **Services** enable communication between different parts of an application and external users.
- They provide a stable IP and DNS name to a set of pods.

### Types of Services

#### 1. ClusterIP (Default)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
```sh
kubectl apply -f clusterip-service.yaml
```

#### 2. NodePort
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
      nodePort: 30007
```
```sh
kubectl apply -f nodeport-service.yaml
```

#### 3. LoadBalancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
```sh
kubectl apply -f loadbalancer-service.yaml
```

#### 4. ExternalName
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-external-service
spec:
  type: ExternalName
  externalName: example.com
```
```sh
kubectl apply -f externalname-service.yaml
```

### Discovering Services
```sh
kubectl get services
kubectl describe service my-service
kubectl get service my-service -o jsonpath='{.spec.clusterIP}'
```

### Ingress vs Service
| Feature       | Service                     | Ingress                 |
|--------------|----------------------------|-------------------------|
| Purpose      | Exposes a set of pods       | Manages external access |
| Load Balancer | Optional                    | Yes (for HTTP/HTTPS)    |
| Routing      | Basic (IP-based)            | Advanced (path-based)   |

---

## Docker vs Containerd

### Docker
- Docker is a containerization platform that simplifies container management.
- It includes features like CLI, API, volume management, and security.

#### CLI Commands
```sh
$ docker run --name redis redis:alpine
$ docker run --name webserver -p 80:80 -d nginx
```

### Containerd
- A lightweight container runtime optimized for Kubernetes.

#### CLI - ctr
```sh
$ ctr images pull docker.io/library/redis:alpine
$ ctr run docker.io/library/redis:alpine redis
```

### nerdctl
- Provides Docker-like commands for containerd.

```sh
$ nerdctl run --name redis redis:alpine
$ nerdctl run --name webserver -p 80:80 -d nginx
```

### crictl
- CLI tool for debugging CRI-compatible runtimes.
```sh
$ crictl pull busybox
$ crictl images
$ crictl ps -a
$ crictl exec -i -t <container-id> ls
```

### Docker vs crictl
| Feature | Docker | crictl |
|---------|--------|--------|
| Purpose | General-purpose container management | Debugging and inspecting containers |
| Works with | Docker engine | Any CRI-compatible runtime |

---

