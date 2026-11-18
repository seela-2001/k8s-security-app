# Kubernetes Security, Storage & Networking Lab

A hands-on Kubernetes project built as part of my **Certified Kubernetes Administrator (CKA)** preparation, following Mumshad Mannambeth's Kubernetes course.

The project focuses on understanding Kubernetes concepts through practical implementation, troubleshooting, and verification — not just writing YAML manifests.

## Project Overview

This lab uses a simple multi-tier application to explore how Kubernetes manages workloads, networking, persistent storage, and access control.

The main components are:

- **Frontend:** NGINX-based application.
- **Backend:** HTTP Echo service.
- **Database:** PostgreSQL.
- **Namespace:** `security-app`.
- **Cluster:** Minikube.

The application is intentionally lightweight. The focus is on Kubernetes administration and infrastructure concepts rather than application development.

## Architecture

```text
                    Kubernetes Cluster
                    Namespace: security-app

                         Frontend
                        (NGINX Pods)
                             |
                             | HTTP
                             v
                     backend-service
                             |
                             v
                        Backend Pods
                       (HTTP Echo)
                             |
                             | PostgreSQL
                             v
                    postgres-service
                             |
                             v
                       PostgreSQL
                      (StatefulSet)
                             |
                             v
                    Persistent Storage
                      PVC / PV
                    StorageClass
```

**Note:** PostgreSQL is deployed as a separate workload for storage and networking exercises. The backend is not configured as an application-level PostgreSQL client.

## Technologies & Kubernetes Concepts

| Area | Concepts Practiced |
|---|---|
| Workloads | Pods, Deployments, StatefulSets, ReplicaSets |
| Networking | Services, ClusterIP, Pod-to-Pod communication, DNS, Service Discovery |
| Network Security | NetworkPolicies, Ingress and Egress rules |
| Storage | PersistentVolumes, PersistentVolumeClaims, StorageClasses, Dynamic Provisioning |
| Authentication | ServiceAccounts, Kubernetes authentication, client certificates |
| Authorization | RBAC, Roles, ClusterRoles, RoleBindings, ClusterRoleBindings |
| Configuration | ConfigMaps and Secrets |
| Troubleshooting | Pod failures, CrashLoopBackOff, service connectivity, storage and networking issues |

## Repository Structure

```text
k8s-security-app/
│
├── manifests/
│   ├── namespace.yaml
│   ├── frontend-deployment.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── postgres-deployment.yaml
│   ├── postgres-statefullsets.yaml
│   └── postgres-service.yaml
│
├── networking/
│
├── security/
│   ├── authentication/
│   │   ├── cluster-service-account.yaml
│   │   └── frontend-service-accoutn.yaml
│   │
│   ├── certificates/
│   │
│   ├── network-policy/
│   │   ├── frontend-network-policy.yaml
│   │   ├── backend-network-policy.yaml
│   │   └── postgres-network-policy.yaml
│   │
│   ├── rbac/
│   │   ├── dev-user-role.yaml
│   │   ├── dev-user-rolebinding.yaml
│   │   ├── dev-user-csr.yaml
│   │   ├── cluster-pod-reader.yaml
│   │   ├── cluster-rolebinding.yaml
│   │   ├── frontend-role.yaml
│   │   └── frontend-rolebinding.yaml
│   │
│   └── security-context/
│
├── storage/
│   ├── postgres-pv.yaml
│   ├── postgres-pvc.yaml
│   └── storage-class.yaml
│
├── screenshots/
│
├── tests/
└── README.md
```

## Implementation Highlights

### 1. Kubernetes Networking

Implemented Kubernetes Services to provide stable network access to application workloads.

Practiced:

- Service selectors and target ports.
- Pod IP addresses and service endpoints.
- Internal communication between application components.
- DNS-based service discovery.
- Endpoint and EndpointSlice inspection.
- NetworkPolicy ingress and egress rules.

Useful commands:

```bash
kubectl get pods -n security-app -o wide
kubectl get svc -n security-app
kubectl get endpointslices -n security-app
kubectl get networkpolicies -n security-app
```

### 2. Persistent Storage

Explored Kubernetes persistent storage using PostgreSQL as a stateful workload.

Practiced:

- PersistentVolumes (PV).
- PersistentVolumeClaims (PVC).
- StorageClasses.
- Static and dynamic provisioning.
- Persistent storage integration with Pods.
- StatefulSet storage using volume claim templates.

The goal was to understand how storage is provisioned, claimed, and mounted into stateful workloads.

### 3. Authentication & Authorization

Implemented Kubernetes access-control scenarios using ServiceAccounts and RBAC.

Practiced:

- Creating and assigning ServiceAccounts.
- Creating Roles and ClusterRoles.
- Binding permissions with RoleBindings and ClusterRoleBindings.
- Generating and submitting CertificateSigningRequests.
- Testing user permissions with `kubectl auth can-i`.
- Configuring and using a separate Kubernetes user context.

Example:

```bash
kubectl auth can-i get pods -n security-app
kubectl auth can-i delete pods -n security-app
kubectl config get-contexts
```

### 4. Network Policies

Created NetworkPolicies to define permitted communication between application components.

The intended traffic flow:

| Source | Destination | Port | Expected Access |
|---|---|---:|---|
| Frontend | Backend | 5678 | Allowed |
| Backend | PostgreSQL | 5432 | Allowed |
| Frontend | PostgreSQL | 5432 | Denied |
| Frontend / Backend | CoreDNS | 53 | DNS traffic allowed |

**Important:** NetworkPolicy enforcement depends on the cluster's CNI plugin. Applying a NetworkPolicy resource alone does not guarantee that traffic is restricted. Enforcement must be verified in a compatible cluster.

## Troubleshooting & Validation

A major part of this project was investigating Kubernetes behavior when things did not work as expected.

Examples include:

- Investigating why Services had no endpoints.
- Debugging Pods in `CrashLoopBackOff`.
- Inspecting container logs and Pod events.
- Troubleshooting PostgreSQL persistent data and WAL recovery errors.
- Understanding PVC access modes and why multiple PostgreSQL instances should not share the same data directory.
- Investigating NetworkPolicy enforcement and CNI compatibility.
- Verifying connectivity using `wget`, `nc`, and `kubectl exec`.

The troubleshooting process followed:

```text
Identify the issue
       |
       v
Inspect Kubernetes resources
       |
       v
Check logs and events
       |
       v
Find the root cause
       |
       v
Apply a fix
       |
       v
Verify the result
```

## Screenshots & Proof of Implementation

The `screenshots/` directory contains evidence from practical exercises, including:

- Frontend connectivity tests.
- DNS and service discovery.
- RBAC allowed and denied operations.
- ServiceAccount configuration.
- Cross-namespace access tests.
- Network connectivity tests.

## Running the Lab

### Prerequisites

- Docker
- Minikube
- kubectl
- Basic Linux command-line knowledge

### Start the Cluster

```bash
minikube start
```

### Create the Namespace

```bash
kubectl apply -f manifests/namespace.yaml
```

### Deploy the Workloads

```bash
kubectl apply -f manifests/
```

Apply the required storage resources and security manifests according to their dependencies and the selected PostgreSQL storage configuration.

### Verify the Resources

```bash
kubectl get all -n security-app
kubectl get pv
kubectl get pvc -n security-app
kubectl get storageclass
kubectl get networkpolicies -n security-app
```

## Key Takeaways

Through this project, I gained practical experience with:

- Understanding how Kubernetes networking connects Pods and Services.
- Working with DNS and service discovery.
- Managing persistent storage for stateful workloads.
- Understanding the difference between Deployments and StatefulSets.
- Implementing authentication and authorization with RBAC.
- Restricting workload communication using NetworkPolicies.
- Troubleshooting real Kubernetes issues using logs, events, and resource inspection.

## Learning Resources

- [Certified Kubernetes Administrator (CKA) – Mumshad Mannambeth](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)
- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Kubernetes Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [Kubernetes Storage](https://kubernetes.io/docs/concepts/storage/)

---

**This project is part of my hands-on journey toward becoming a Kubernetes and DevOps Engineer.**
