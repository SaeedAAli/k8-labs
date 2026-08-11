# Kubernetes Infrastructure & Architecture

A comprehensive reference for Kubernetes (K8s) infrastructure, architecture, core components, networking, storage, workloads, security, configuration, scheduling, observability, autoscaling, cloud integration, and common deployment workflows.

---

## Table of Contents

1. [What is Kubernetes?](#what-is-kubernetes)
2. [Kubernetes Cluster](#kubernetes-cluster)
3. [High-Level Architecture](#high-level-architecture)
4. [Control Plane](#control-plane)
5. [Worker Nodes](#worker-nodes)
6. [Pods](#pods)
7. [Containers and Container Runtimes](#containers-and-container-runtimes)
8. [Kubernetes API](#kubernetes-api)
9. [Kubernetes Objects](#kubernetes-objects)
10. [Workloads](#workloads)
11. [Services and Networking](#services-and-networking)
12. [Ingress](#ingress)
13. [DNS](#dns)
14. [Storage](#storage)
15. [Configuration and Secrets](#configuration-and-secrets)
16. [Scheduling](#scheduling)
17. [Labels, Selectors and Annotations](#labels-selectors-and-annotations)
18. [Namespaces](#namespaces)
19. [RBAC and Security](#rbac-and-security)
20. [Resource Management](#resource-management)
21. [Health Checks](#health-checks)
22. [Autoscaling](#autoscaling)
23. [High Availability](#high-availability)
24. [Cloud Provider Integration](#cloud-provider-integration)
25. [AWS EKS Example](#aws-eks-example)
26. [CI/CD with Kubernetes](#cicd-with-kubernetes)
27. [Observability](#observability)
28. [Kubernetes Lifecycle](#kubernetes-lifecycle)
29. [Common kubectl Commands](#common-kubectl-commands)
30. [Example Application Architecture](#example-application-architecture)
31. [Important Relationships to Remember](#important-relationships-to-remember)
32. [Summary](#summary)

---

# What is Kubernetes?

[Kubernetes](https://kubernetes.io/) is an open-source container orchestration platform used to deploy, manage, scale, and maintain containerised applications.

Instead of manually running containers across multiple machines, Kubernetes provides a control system that continuously works to keep the application in the state that has been declared by the user.

Kubernetes can:

- Deploy containers
- Restart failed containers
- Replace failed Pods
- Scale applications
- Perform rolling updates
- Roll back deployments
- Provide service discovery
- Load-balance traffic
- Manage configuration
- Manage secrets
- Attach persistent storage
- Schedule workloads across nodes
- Enforce access control
- Integrate with cloud providers
- Expose applications to internal and external networks

---

# Kubernetes Cluster

A Kubernetes **cluster** is the complete Kubernetes environment.

At a high level:

```text
KUBERNETES CLUSTER
│
├── CONTROL PLANE
│   ├── kube-apiserver
│   ├── etcd
│   ├── kube-scheduler
│   ├── kube-controller-manager
│   └── cloud-controller-manager (when applicable)
│
└── WORKER NODES
    ├── Worker Node 1
    │   ├── kubelet
    │   ├── kube-proxy
    │   ├── Container Runtime
    │   └── Pods
    │
    ├── Worker Node 2
    │   ├── kubelet
    │   ├── kube-proxy
    │   ├── Container Runtime
    │   └── Pods
    │
    └── Worker Node N
        ├── kubelet
        ├── kube-proxy
        ├── Container Runtime
        └── Pods
```

The cluster is therefore **not just the worker nodes**.

It consists of the Kubernetes control plane and the compute infrastructure on which workloads run.

---

# High-Level Architecture

```text
                              USERS / DEVELOPERS
                                      │
                                      │
                                  kubectl
                                      │
                                      ▼
                              ┌───────────────┐
                              │  API SERVER   │
                              │ kube-apiserver│
                              └───────┬───────┘
                                      │
                 ┌────────────────────┼────────────────────┐
                 │                    │                    │
                 ▼                    ▼                    ▼
              ┌──────┐          ┌───────────┐      ┌──────────────┐
              │ etcd │          │ Scheduler │      │ Controllers  │
              └──────┘          └───────────┘      └──────────────┘
                 │                    │                    │
                 └────────────────────┼────────────────────┘
                                      │
                                      ▼
                              CONTROL PLANE
                                      │
                         Kubernetes API / State
                                      │
                ┌─────────────────────┼─────────────────────┐
                │                     │                     │
                ▼                     ▼                     ▼
          WORKER NODE 1         WORKER NODE 2         WORKER NODE N
                │                     │                     │
        ┌───────┼────────┐    ┌───────┼────────┐    ┌───────┼────────┐
        │       │        │    │       │        │    │       │        │
     kubelet kube-proxy Runtime kubelet kube-proxy Runtime kubelet kube-proxy Runtime
        │       │        │    │       │        │    │       │        │
        └───────┴────────┘    └───────┴────────┘    └───────┴────────┘
                │                     │                     │
             ┌──┴──┐               ┌──┴──┐               ┌──┴──┐
             │Pods │               │Pods │               │Pods │
             └─────┘               └─────┘               └─────┘
```

---

# Control Plane

The **control plane** is responsible for managing the Kubernetes cluster.

It makes decisions about the cluster and continuously attempts to make the actual state match the desired state.

The main control-plane components are:

```text
CONTROL PLANE
│
├── kube-apiserver
├── etcd
├── kube-scheduler
├── kube-controller-manager
└── cloud-controller-manager
```

---

## kube-apiserver

The **API server** is the central entry point into Kubernetes.

Almost all Kubernetes operations go through the API server.

```text
kubectl
   │
   ▼
API Server
   │
   ├── Authentication
   ├── Authorisation
   ├── Admission
   └── API processing
```

Responsibilities include:

- Exposing the Kubernetes API
- Authenticating requests
- Authorising requests
- Validating API objects
- Processing admission controls
- Reading/writing cluster state
- Acting as the communication hub between Kubernetes components

Example:

```bash
kubectl get pods
```

The request is sent to the API server.

---

## etcd

`etcd` is the distributed key-value datastore used by Kubernetes.

It stores the cluster's persistent state.

Examples of information stored include:

- Nodes
- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- Namespaces
- Roles
- Cluster configuration

Conceptually:

```text
Kubernetes API Server
        │
        ▼
       etcd
        │
        └── Kubernetes cluster state
```

`etcd` is critical to the control plane.

If the control plane is highly available, etcd is normally deployed in a highly available configuration.

---

## kube-scheduler

The scheduler decides **which node should run a newly created Pod**.

Example:

```text
New Pod
   │
   ▼
API Server
   │
   ▼
Scheduler
   │
   ├── CPU available?
   ├── Memory available?
   ├── Node constraints?
   ├── Taints/tolerations?
   ├── Affinity rules?
   └── Other scheduling requirements?
          │
          ▼
     Selected Node
```

The scheduler does not itself run the container.

It selects the appropriate node.

---

## kube-controller-manager

The controller manager runs Kubernetes controllers.

Controllers continuously compare:

```text
DESIRED STATE
      vs
CURRENT STATE
```

Example:

```text
Desired:
3 Pods

Current:
2 Pods

Controller:
Create another Pod
```

Common controllers include:

- Deployment controller
- ReplicaSet controller
- Node controller
- Job controller
- EndpointSlice controller
- Namespace controller
- ServiceAccount controller

---

## cloud-controller-manager

The cloud controller manager allows Kubernetes to interact with a cloud provider.

For example:

```text
Kubernetes
    │
    ▼
Cloud Controller Manager
    │
    ▼
AWS / Azure / GCP
```

Depending on the environment, it can integrate Kubernetes with cloud resources such as:

- Load balancers
- Routes
- Nodes
- Cloud volumes
- Network resources

---

# Worker Nodes

A **worker node** is a machine that provides compute resources for Kubernetes workloads.

A node can be:

- A virtual machine
- A physical machine
- A cloud instance
- Another supported compute resource

A typical worker node contains:

```text
WORKER NODE
│
├── kubelet
├── kube-proxy
├── Container Runtime
│
└── Pods
    ├── Container
    ├── Container
    └── ...
```

---

# kubelet

The **kubelet** is the Kubernetes agent running on each worker node.

Its primary responsibility is to ensure that the containers described by Pod specifications are running and healthy.

Conceptually:

```text
API Server
    │
    │ Pod specification
    ▼
 kubelet
    │
    ▼
Container Runtime
    │
    ▼
Container
```

The kubelet:

- Registers the node
- Watches for Pod specifications
- Communicates with the container runtime
- Reports Pod/node status
- Runs health checks
- Ensures assigned Pods are running

### kubelet vs kubectl

These are different:

| Component | Purpose |
|---|---|
| `kubectl` | CLI used by humans/automation to communicate with Kubernetes |
| `kubelet` | Agent running on worker nodes that manages Pods |

```text
Developer
   │
kubectl
   │
   ▼
API Server
   │
   ▼
kubelet
   │
   ▼
Pod
```

---

# kube-proxy

`kube-proxy` is a node-level networking component associated with Kubernetes Service networking.

It helps implement network rules that allow traffic to reach Services and their backend Pods.

Conceptually:

```text
Client
  │
  ▼
Service
  │
  ▼
kube-proxy / node networking
  │
  ├── Pod A
  ├── Pod B
  └── Pod C
```

Modern Kubernetes networking implementations can use different mechanisms and dataplanes, so kube-proxy is not the only possible way networking can be implemented.

---

# Containers and Container Runtimes

Kubernetes does not directly execute containers itself.

A **container runtime** is responsible for running containers.

Common Kubernetes-compatible runtimes include:

- containerd
- CRI-O

The relationship is:

```text
Kubernetes
     │
     ▼
   kubelet
     │
     ▼
Container Runtime
     │
     ├── containerd
     └── CRI-O
     │
     ▼
 Container
```

---

## Docker vs Kubernetes

Docker and Kubernetes solve different problems.

### Docker

Primarily provides:

- Container image building
- Container packaging
- Local container execution
- Container development workflow

### Kubernetes

Primarily provides:

- Container orchestration
- Scheduling
- Scaling
- Service discovery
- Self-healing
- Rolling updates
- Networking
- Configuration
- Storage orchestration

A common workflow is:

```text
Dockerfile
    │
    ▼
Docker build
    │
    ▼
Container Image
    │
    ▼
Container Registry
    │
    ▼
Kubernetes
    │
    ▼
Container Runtime
    │
    ▼
Container
```

---

# Pods

A **Pod** is the smallest deployable unit in Kubernetes.

A Pod contains one or more containers that share certain resources, including networking and attached storage.

Most applications use one main container per Pod.

```text
Pod
│
├── Container
│
└── Shared resources
    ├── Network namespace
    └── Volumes
```

Multiple containers in one Pod can be useful for patterns such as sidecars:

```text
Pod
├── Application Container
└── Sidecar Container
```

Pods are generally **ephemeral**. Applications should normally be managed through higher-level workload resources rather than manually creating individual Pods.

---

# Kubernetes API

Kubernetes is fundamentally API-driven.

Objects are represented through Kubernetes APIs.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
```

The workflow is:

```text
YAML
  │
  ▼
kubectl apply
  │
  ▼
API Server
  │
  ▼
Validation / Admission
  │
  ▼
etcd
  │
  ▼
Controllers
  │
  ▼
Scheduler
  │
  ▼
Worker Node
  │
  ▼
Pod
```

---

# Kubernetes Objects

Kubernetes objects represent the desired state of resources.

Common objects include:

```text
Workloads
├── Pod
├── Deployment
├── ReplicaSet
├── StatefulSet
├── DaemonSet
├── Job
└── CronJob

Networking
├── Service
├── Ingress
└── EndpointSlice

Configuration
├── ConfigMap
└── Secret

Storage
├── PersistentVolume
├── PersistentVolumeClaim
└── StorageClass

Security
├── ServiceAccount
├── Role
├── RoleBinding
├── ClusterRole
└── ClusterRoleBinding

Cluster
├── Node
└── Namespace
```

---

# Workloads

## Deployment

A Deployment manages stateless applications.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

Architecture:

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ├── Pod
    ├── Pod
    └── Pod
```

---

## ReplicaSet

A ReplicaSet maintains a desired number of identical Pods.

```text
Desired replicas = 3

ReplicaSet
├── Pod 1
├── Pod 2
└── Pod 3
```

If one Pod disappears:

```text
3 Pods
 │
 └── Pod fails
       │
       ▼
2 Pods
       │
       ▼
ReplicaSet creates replacement
       │
       ▼
3 Pods
```

Deployments normally manage ReplicaSets for you.

---

## StatefulSet

StatefulSets are designed for workloads that require stable identity and/or persistent storage.

Examples:

- Databases
- Distributed systems
- Stateful applications

Example identity:

```text
database-0
database-1
database-2
```

Unlike ordinary Deployment Pods, these identities are stable.

---

## DaemonSet

A DaemonSet ensures a Pod runs on selected nodes, commonly one Pod per node.

Examples:

- Logging agents
- Monitoring agents
- Security agents
- Node-level networking components

```text
Node 1 ── Agent
Node 2 ── Agent
Node 3 ── Agent
```

---

## Job

A Job runs a task until it successfully completes.

```text
Job
 │
 ▼
Pod
 │
 ▼
Task completes
```

Example:

```text
Database migration
Batch processing
One-time script
```

---

## CronJob

A CronJob creates Jobs according to a schedule.

```text
CronJob
   │
   ├── Job ── Pod
   │
   ├── Job ── Pod
   │
   └── Job ── Pod
```

Example use case:

```text
Run database backup every night
```

---

# Services and Networking

Pods are ephemeral.

Their IP addresses can change when Pods are recreated.

A **Service** provides a stable networking endpoint for a group of Pods.

```text
                 Service
                    │
          ┌─────────┼─────────┐
          │         │         │
        Pod A      Pod B      Pod C
```

The Service selects Pods using labels.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

---

## ClusterIP

Default Service type.

Used for internal cluster communication.

```text
Pod A
  │
  ▼
ClusterIP Service
  │
  ├── Pod B
  └── Pod C
```

---

## NodePort

Exposes a Service through a port on each node.

```text
Internet
   │
   ▼
Node IP:NodePort
   │
   ▼
Service
   │
   ▼
Pods
```

---

## LoadBalancer

Requests a load balancer from a supported cloud/infrastructure integration.

```text
Internet
    │
    ▼
Cloud Load Balancer
    │
    ▼
Kubernetes Service
    │
    ▼
Pods
```

---

# Ingress

Ingress provides HTTP/HTTPS routing into Kubernetes Services.

A common architecture is:

```text
Internet
   │
   ▼
DNS
   │
   ▼
Load Balancer
   │
   ▼
Ingress Controller
   │
   ├── /api  ──► API Service
   │
   └── /     ──► Frontend Service
```

Important distinction:

- **Ingress** is the Kubernetes API object/routing configuration.
- **Ingress Controller** is the software that implements the routing.

Examples of ingress controllers include:

- NGINX Ingress Controller
- Traefik
- HAProxy
- Cloud-provider-specific controllers

---

# DNS

Kubernetes normally provides internal DNS through a cluster DNS implementation such as CoreDNS.

A Service can be accessed using a DNS name such as:

```text
service-name.namespace.svc.cluster.local
```

Example:

```text
frontend.default.svc.cluster.local
```

Conceptually:

```text
Pod
 │
 │ DNS query
 ▼
CoreDNS
 │
 ▼
Service
 │
 ▼
Backend Pods
```

---

# Storage

Containers and Pods are generally ephemeral, so persistent applications need persistent storage.

Kubernetes provides abstractions for storage.

```text
Pod
 │
 ▼
PersistentVolumeClaim
 │
 ▼
PersistentVolume
 │
 ▼
Physical / Cloud Storage
```

---

## PersistentVolume (PV)

A PersistentVolume represents storage available to the cluster.

The storage may come from:

- Cloud disks
- Network storage
- Local storage
- Other storage systems

---

## PersistentVolumeClaim (PVC)

A PVC is a request for storage made by a workload.

```text
Application
    │
    ▼
PVC
    │
    ▼
PV
    │
    ▼
Storage
```

---

## StorageClass

A StorageClass defines how storage can be dynamically provisioned.

```text
PVC
 │
 ▼
StorageClass
 │
 ▼
Dynamic Provisioning
 │
 ▼
Persistent Storage
```

---

# Configuration and Secrets

## ConfigMap

A ConfigMap stores non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
```

---

## Secret

A Secret is designed for sensitive configuration such as:

- Passwords
- API keys
- Tokens
- Certificates

Example:

```text
Pod
 │
 ├── ConfigMap
 │
 └── Secret
```

Important: Kubernetes Secrets are not automatically equivalent to a full secrets-management system. Encryption at rest and access controls should be configured appropriately.

---

# Scheduling

Kubernetes scheduling determines where Pods run.

Factors can include:

- CPU
- Memory
- Resource requests
- Resource limits
- Node selectors
- Affinity
- Anti-affinity
- Taints
- Tolerations
- Topology constraints
- Node availability

Example:

```text
Pod
 │
 ▼
Scheduler
 │
 ├── Node 1 ❌ insufficient CPU
 ├── Node 2 ❌ taint mismatch
 └── Node 3 ✅ suitable
             │
             ▼
          Pod runs
```

---

# Labels, Selectors and Annotations

## Labels

Labels identify and organise Kubernetes objects.

```yaml
labels:
  app: frontend
  environment: production
```

---

## Selectors

Selectors find resources using labels.

Example:

```text
Service
 selector:
   app=frontend
       │
       ├── Pod A
       ├── Pod B
       └── Pod C
```

---

## Annotations

Annotations store additional metadata that can be consumed by Kubernetes components and other systems.

They are not intended to be used as the primary mechanism for selecting objects.

---

# Namespaces

Namespaces logically separate resources within a cluster.

Example:

```text
Cluster
│
├── development
│   ├── frontend
│   └── backend
│
├── staging
│   ├── frontend
│   └── backend
│
└── production
    ├── frontend
    └── backend
```

Namespaces can help with:

- Organisation
- Access control
- Resource quotas
- Environment separation

Namespaces are logical isolation, not automatically a complete security boundary.

---

# RBAC and Security

Kubernetes uses **Role-Based Access Control (RBAC)** to control access to resources.

Main concepts:

```text
User / ServiceAccount
        │
        ▼
      Role
        │
        ▼
   RoleBinding
        │
        ▼
 Kubernetes Resources
```

---

## Role

Defines permissions within a namespace.

Example permissions:

```text
get Pods
list Pods
create Deployments
update Services
```

---

## ClusterRole

Defines permissions at cluster scope or reusable permissions that can be bound in namespaces.

---

## RoleBinding

Connects a Role to a user, group, or ServiceAccount.

---

## ServiceAccount

Provides an identity for workloads running inside Kubernetes.

```text
Pod
 │
 ▼
ServiceAccount
 │
 ▼
RBAC permissions
```

---

# Resource Management

Kubernetes can define CPU and memory requirements.

## Requests

Requests indicate the resources a container needs for scheduling purposes.

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
```

## Limits

Limits define maximum resource usage for a container.

```yaml
resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
```

Conceptually:

```text
Pod
 │
 ├── CPU Request
 ├── CPU Limit
 ├── Memory Request
 └── Memory Limit
```

---

# Health Checks

Kubernetes supports probes for determining application health.

## Liveness Probe

Determines whether the application should be restarted.

```text
Application
    │
Liveness Probe
    │
    ├── Healthy → continue
    └── Unhealthy → restart container
```

---

## Readiness Probe

Determines whether the application is ready to receive traffic.

```text
Pod
 │
Readiness Probe
 │
 ├── Ready → Service can route traffic
 └── Not ready → remove from traffic
```

---

## Startup Probe

Useful for applications that take a long time to start.

```text
Container starts
      │
      ▼
Startup Probe
      │
      ▼
Application becomes ready
      │
      ▼
Readiness/Liveness checks
```

---

# Autoscaling

## Horizontal Pod Autoscaler (HPA)

HPA adjusts the number of Pod replicas.

```text
CPU / Memory / Custom Metrics
             │
             ▼
            HPA
             │
       ┌─────┴─────┐
       │           │
     Scale Up    Scale Down
       │           │
       ▼           ▼
 More Pods      Fewer Pods
```

Example:

```text
Low traffic
   │
   ▼
2 Pods

Traffic increases
   │
   ▼
5 Pods

Traffic decreases
   │
   ▼
2 Pods
```

---

## Vertical Pod Autoscaler (VPA)

VPA can adjust resource requests/limits based on observed usage.

---

## Cluster Autoscaler

The Cluster Autoscaler can add or remove nodes when workloads cannot be scheduled or nodes are underutilised, depending on the cluster environment.

```text
Pods need more capacity
        │
        ▼
Cluster Autoscaler
        │
        ▼
New Worker Node
        │
        ▼
Pods scheduled
```

---

# High Availability

Production Kubernetes clusters should avoid having a single point of failure.

A highly available architecture can look like:

```text
                 LOAD BALANCER
                      │
          ┌───────────┼───────────┐
          │           │           │
       API Server  API Server  API Server
          │           │           │
          └───────────┼───────────┘
                      │
                    etcd
                 (HA cluster)
                      │
          ┌───────────┼───────────┐
          │           │           │
        Node 1      Node 2      Node 3
          │           │           │
         Pods        Pods        Pods
```

High availability can involve:

- Multiple control-plane instances
- Highly available etcd
- Multiple worker nodes
- Multiple replicas
- Pod anti-affinity
- Multiple availability zones
- Load balancing
- Automatic recovery

---

# Cloud Provider Integration

Kubernetes can run:

- On-premises
- Locally
- In virtual machines
- In public clouds
- In managed Kubernetes services

Major cloud providers offer managed Kubernetes services.

```text
AWS
└── Amazon EKS

Microsoft Azure
└── Azure Kubernetes Service (AKS)

Google Cloud
└── Google Kubernetes Engine (GKE)
```

Managed Kubernetes typically reduces the amount of control-plane infrastructure the customer has to operate.

---

# AWS EKS Example

Amazon EKS provides managed Kubernetes.

A simplified architecture:

```text
                         AWS CLOUD
                             │
                 ┌───────────┴───────────┐
                 │                       │
          EKS Control Plane          AWS Services
                 │                       │
        ┌────────┼────────┐        ┌─────┼─────────┐
        │        │        │        │     │         │
    API Server  etcd  Scheduler   ECR   ALB       EBS
        │
        │
        ▼
   Worker Compute
        │
   ┌────┴───────────────┐
   │                    │
Worker Node 1       Worker Node 2
   │                    │
kubelet              kubelet
   │                    │
Runtime              Runtime
   │                    │
Pods                 Pods
```

AWS services commonly used alongside EKS include:

- ECR — container image registry
- VPC — networking
- ALB/NLB — load balancing
- EBS — block storage
- EFS — shared file storage
- IAM — identity and permissions
- CloudWatch — monitoring/logging
- Route 53 — DNS
- ACM — TLS certificates

---

# Azure AKS Example

Azure Kubernetes Service follows a similar model:

```text
Azure
 │
 └── AKS
      │
      ├── Managed Control Plane
      │
      └── Node Pools
           │
           ├── Node
           │    ├── kubelet
           │    ├── Runtime
           │    └── Pods
           │
           └── Node
                ├── kubelet
                ├── Runtime
                └── Pods
```

Azure services commonly integrated with AKS include:

- Azure Container Registry
- Azure Load Balancer
- Azure Application Gateway
- Azure Virtual Network
- Azure Managed Disks
- Azure Files
- Microsoft Entra ID
- Azure Monitor

---

# CI/CD with Kubernetes

A common deployment pipeline looks like:

```text
Developer
    │
    ▼
Git Repository
    │
    ▼
CI/CD Pipeline
    │
    ├── Test
    ├── Build
    ├── Security Scan
    └── Build Container Image
             │
             ▼
       Container Registry
             │
             ▼
       Kubernetes Deployment
             │
             ▼
          API Server
             │
             ▼
          Deployment
             │
             ▼
         ReplicaSet
             │
             ▼
            Pods
```

Common tools:

- GitHub Actions
- GitLab CI/CD
- Jenkins
- Azure DevOps
- Argo CD
- Flux

---

# Observability

Kubernetes environments commonly use three major observability areas.

## Metrics

Used to measure system performance.

Examples:

- CPU usage
- Memory usage
- Request rate
- Error rate
- Latency

Common tools:

- Prometheus
- Grafana
- CloudWatch
- Azure Monitor

---

## Logs

Used to investigate application and infrastructure events.

Example:

```text
Application
    │
    ▼
Container stdout/stderr
    │
    ▼
Logging Agent
    │
    ▼
Log Platform
```

Examples:

- Fluent Bit
- Fluentd
- Elasticsearch
- OpenSearch
- Loki
- CloudWatch Logs

---

## Tracing

Used to follow requests through distributed systems.

Common tools:

- OpenTelemetry
- Jaeger
- Grafana Tempo

---

# Kubernetes Lifecycle

A typical application deployment lifecycle:

```text
1. Developer writes application
          │
          ▼
2. Dockerfile created
          │
          ▼
3. Container image built
          │
          ▼
4. Image pushed to registry
          │
          ▼
5. Kubernetes manifest created
          │
          ▼
6. kubectl apply
          │
          ▼
7. API Server receives request
          │
          ▼
8. Object stored in etcd
          │
          ▼
9. Controller processes desired state
          │
          ▼
10. Scheduler selects node
          │
          ▼
11. kubelet receives Pod assignment
          │
          ▼
12. Container runtime pulls image
          │
          ▼
13. Container starts
          │
          ▼
14. Readiness checks pass
          │
          ▼
15. Service routes traffic
```

---

# Common kubectl Commands

## Cluster information

```bash
kubectl cluster-info
kubectl get nodes
kubectl get namespaces
kubectl version
```

## Pods

```bash
kubectl get pods
kubectl get pods -A
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/sh
```

## Deployments

```bash
kubectl get deployments
kubectl describe deployment <deployment-name>
kubectl apply -f deployment.yaml
kubectl delete -f deployment.yaml
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name>
```

## Services

```bash
kubectl get services
kubectl describe service <service-name>
```

## Configuration

```bash
kubectl get configmaps
kubectl get secrets
```

## Nodes

```bash
kubectl get nodes
kubectl describe node <node-name>
```

## Debugging

```bash
kubectl get events
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
```

---

# Example Application Architecture

A typical production-style web application could look like:

```text
                         INTERNET
                            │
                            ▼
                           DNS
                            │
                            ▼
                    Cloud Load Balancer
                            │
                            ▼
                    Ingress Controller
                            │
             ┌──────────────┴──────────────┐
             │                             │
             ▼                             ▼
       Frontend Service              API Service
             │                             │
       ┌─────┼─────┐                 ┌─────┼─────┐
       │     │     │                 │     │     │
      Pod   Pod   Pod               Pod   Pod   Pod
       │                             │
       │                             ▼
       │                         Database
       │                             │
       │                         Persistent
       │                          Storage
       │
       └──────────── Kubernetes Cluster ────────────┘
```

A more complete cloud architecture:

```text
                              INTERNET
                                  │
                                  ▼
                                DNS
                                  │
                                  ▼
                         Cloud Load Balancer
                                  │
                                  ▼
                         Ingress Controller
                                  │
                     ┌────────────┴────────────┐
                     │                         │
                     ▼                         ▼
              Frontend Service           API Service
                     │                         │
                ┌────┴────┐               ┌────┴────┐
                │         │               │         │
               Pod       Pod             Pod       Pod
                │                         │
                │                         ▼
                │                    Database
                │                         │
                │                         ▼
                │                    Persistent
                │                     Storage
                │
                └───────────┐
                            │
                       Kubernetes
                          Cluster
                            │
                    ┌───────┴────────┐
                    │                │
                 Node 1           Node 2
                    │                │
                  Pods             Pods
```

---

# Important Relationships to Remember

## Cluster → Nodes → Pods → Containers

```text
Cluster
  │
  ├── Control Plane
  │
  └── Worker Nodes
       │
       └── Pods
            │
            └── Containers
```

This is one of the most important Kubernetes relationships.

---

## kubectl → API Server → Kubernetes

```text
kubectl
   │
   ▼
API Server
   │
   ▼
Kubernetes Resources
```

`kubectl` does **not** directly control Pods.

It communicates with the API server.

---

## kubelet → Container Runtime → Containers

```text
kubelet
   │
   ▼
Container Runtime
   │
   ▼
Container
```

The kubelet is the node agent.

The container runtime actually runs the containers.

---

## Deployment → ReplicaSet → Pods

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
Pods
```

---

## Service → Pods

```text
Service
   │
   ├── Pod
   ├── Pod
   └── Pod
```

A Service provides a stable endpoint for a changing set of Pods.

---

## Ingress → Service → Pods

```text
Internet
   │
   ▼
Ingress
   │
   ▼
Service
   │
   ▼
Pods
```

---

## PVC → PV → Storage

```text
Pod
 │
 ▼
PVC
 │
 ▼
PV
 │
 ▼
Storage
```

---

## Scheduler → Node

```text
Pod requiring scheduling
          │
          ▼
      Scheduler
          │
          ▼
      Worker Node
```

---

# Complete Kubernetes Infrastructure Diagram

```text
                                      USERS
                                        │
                                        │ HTTPS
                                        ▼
                                      DNS
                                        │
                                        ▼
                              CLOUD LOAD BALANCER
                                        │
                                        ▼
                              INGRESS CONTROLLER
                                        │
                              ┌─────────┴─────────┐
                              │                   │
                              ▼                   ▼
                       FRONTEND SERVICE      API SERVICE
                              │                   │
                         ┌────┴────┐        ┌────┴────┐
                         │         │        │         │
                        Pod       Pod      Pod       Pod
                         │         │        │         │
                         └────┬────┘        └────┬────┘
                              │                   │
                              │                   ▼
                              │               DATABASE
                              │                   │
                              │                   ▼
                              │              PVC / PV
                              │                   │
                              │                   ▼
                              │             CLOUD STORAGE
                              │
              ╔═══════════════╧════════════════════════════════╗
              ║              KUBERNETES CLUSTER                ║
              ║                                                 ║
              ║  ┌───────────────────────────────────────────┐  ║
              ║  │              CONTROL PLANE                │  ║
              ║  │                                           │  ║
              ║  │  API Server ─────── etcd                  │  ║
              ║  │      │                                    │  ║
              ║  │      ├──────── Scheduler                  │  ║
              ║  │      │                                    │  ║
              ║  │      ├──────── Controller Manager         │  ║
              ║  │      │                                    │  ║
              ║  │      └──────── Cloud Controller Manager   │  ║
              ║  └────────────────────┬──────────────────────┘  ║
              ║                       │                         ║
              ║                       │ Kubernetes API          ║
              ║          ┌────────────┼────────────┐            ║
              ║          │            │            │            ║
              ║          ▼            ▼            ▼            ║
              ║     ┌─────────┐ ┌─────────┐ ┌─────────┐        ║
              ║     │  NODE 1 │ │  NODE 2 │ │  NODE N │        ║
              ║     │         │ │         │ │         │        ║
              ║     │ kubelet │ │ kubelet │ │ kubelet │        ║
              ║     │ kube-   │ │ kube-   │ │ kube-   │        ║
              ║     │ proxy   │ │ proxy   │ │ proxy   │        ║
              ║     │ runtime │ │ runtime │ │ runtime │        ║
              ║     │         │ │         │ │         │        ║
              ║     │  Pods   │ │  Pods   │ │  Pods   │        ║
              ║     └─────────┘ └─────────┘ └─────────┘        ║
              ╚═════════════════════════════════════════════════╝
                                        │
                                        │
                         CLOUD PROVIDER INTEGRATION
                                        │
                         ┌──────────────┼──────────────┐
                         │              │              │
                        AWS           AZURE          GCP
                         │              │              │
                        EKS            AKS            GKE
                         │              │              │
                  ┌──────┼──────┐ ┌────┼─────┐ ┌─────┼─────┐
                  │      │      │ │    │     │ │     │     │
                 ECR     ALB    EBS ACR  LB  Disk Registry ...
```

---

# The Kubernetes Mental Model

When learning Kubernetes, think about the platform in layers:

```text
                    APPLICATION
                         │
                         ▼
                       PODS
                         │
                         ▼
                 WORKLOAD RESOURCES
          Deployment / StatefulSet / DaemonSet
                         │
                         ▼
                  SERVICES / INGRESS
                         │
                         ▼
                   NETWORKING
                         │
                         ▼
                  WORKER NODES
                         │
             ┌───────────┼───────────┐
             │           │           │
          kubelet    kube-proxy   Runtime
             │                       │
             └───────────┬───────────┘
                         │
                         ▼
                  CONTAINER RUNTIME
                         │
                         ▼
                    CONTAINERS
```

Above the nodes:

```text
                 CONTROL PLANE
                      │
        ┌─────────────┼─────────────┐
        │             │             │
    API Server      etcd       Controllers
        │             │             │
        └─────────────┼─────────────┘
                      │
                  Scheduler
                      │
                      ▼
                Worker Nodes
```

---

# Kubernetes vs Docker vs Cloud

These technologies should not be viewed as direct replacements for one another.

```text
CLOUD PROVIDER
AWS / Azure / GCP
       │
       ▼
KUBERNETES
EKS / AKS / GKE
       │
       ▼
WORKER NODES
VMs / other compute
       │
       ▼
CONTAINER RUNTIME
containerd / CRI-O
       │
       ▼
CONTAINERS
       │
       ▼
APPLICATION
```

Docker can fit into the image-build portion:

```text
Application
    │
    ▼
Dockerfile
    │
    ▼
Docker
    │
    ▼
Container Image
    │
    ▼
Registry
    │
    ▼
Kubernetes
```

---

# Summary

The most important Kubernetes concepts can be reduced to these relationships:

```text
Kubernetes Cluster
│
├── Control Plane
│   ├── API Server
│   ├── etcd
│   ├── Scheduler
│   ├── Controller Manager
│   └── Cloud Controller Manager
│
└── Worker Nodes
    │
    ├── kubelet
    ├── kube-proxy
    ├── Container Runtime
    │
    └── Pods
        │
        └── Containers
```

And for application delivery:

```text
Developer
   │
   ▼
Git
   │
   ▼
CI/CD
   │
   ▼
Container Image
   │
   ▼
Registry
   │
   ▼
Kubernetes API
   │
   ▼
Deployment
   │
   ▼
ReplicaSet
   │
   ▼
Pods
   │
   ▼
Service
   │
   ▼
Ingress / Load Balancer
   │
   ▼
Users
```

Kubernetes therefore acts as the orchestration layer between your **containerised applications** and the **compute/networking/storage infrastructure** on which those applications run.
