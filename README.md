# K8s Security, Storage & Networking Lab

## Overview
A hands-on Kubernetes administration lab for CKA preparation. The application is intentionally simple; the focus is Kubernetes administration, security, networking, storage, and troubleshooting.

Project namespace: `security-app`

## Architecture

```text
Frontend (nginx)
      |
Backend Service
      |
Backend (hashicorp/http-echo)

PostgreSQL Service
      |
PostgreSQL Pod
```

The backend is intentionally an echo service and does not implement real database integration. PostgreSQL is used as a Kubernetes networking, authentication, secrets, and storage target.

## Structure

```text
.
├── manifests/
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── namespace.yaml
│   ├── postgres-deployment.yaml
│   └── postgres-service.yaml
├── networking/
├── security/
│   ├── authentication/
│   │   ├── cluster-service-account.yaml
│   │   └── frontend-service-accoutn.yaml
│   ├── certificates/
│   ├── network-policy/
│   ├── rbac/
│   │   ├── cluster-pod-reader.yaml
│   │   ├── cluster-rolebinding.yaml
│   │   ├── cluster-rolebinding.yaml.yaml
│   │   ├── frontend-rolebinding.yaml
│   │   └── frontend-role.yaml
│   └── security-context/
├── storage/
├── tests/
└── README.md
```

`cluster-rolebinding.yaml.yaml` is a duplicate/mistaken filename and should be cleaned up later.

## Completed Work

### Application
- `security-app` namespace
- Frontend: `nginx:alpine`
- Backend: `hashicorp/http-echo:1.0`, port `5678`
- PostgreSQL workload and `postgres-service:5432`
- Kubernetes DNS/Service connectivity testing

### Service networking
We practiced:

```text
Client -> Service -> Selector -> Endpoints -> Pod IP -> Container Port
```

The backend Service uses `5678 -> 5678`. Testing `http://backend-service` failed because curl defaults to port 80; `http://backend-service:5678` worked.

### ServiceAccounts
Created and tested:
- `frontend-sa`
- `cluster-reader-sa`

We used `kubectl auth can-i` and verified allowed and denied operations.

### RBAC
Implemented:
- Role
- RoleBinding
- ClusterRole
- ClusterRole referenced by RoleBinding

Important distinction:

```text
ClusterRole + RoleBinding
    = permissions limited to the RoleBinding namespace

ClusterRole + ClusterRoleBinding
    = cluster-wide permissions for applicable resources
```

A previous mistake created `cluster-pod-reader` as `Role` instead of `ClusterRole`. It was corrected.

### KubeConfig
Practiced:
- clusters
- users
- contexts
- current context
- namespace in a context
- manual kubeconfig editing
- a context using the limited ServiceAccount

This also demonstrated that the limited ServiceAccount context cannot modify RBAC resources, so the admin `minikube` context is required for cluster configuration changes.

## Certificates — Next Lab

Certificates have NOT yet been fully implemented.

They belong here:

```text
security/certificates/
```

Planned flow:

```text
Private Key
    ↓
CSR
    ↓
Kubernetes CertificateSigningRequest
    ↓
Approval
    ↓
Client Certificate
    ↓
KubeConfig User
    ↓
Context
    ↓
kubectl
```

Planned user:

```text
dev-user
```

The certificate lab will demonstrate client-certificate authentication separately from ServiceAccount token authentication.

Do not commit private keys or sensitive generated credentials.

## Remaining Security Work

1. Client certificate authentication
2. ClusterRoleBinding
3. Image Security
4. Security Context
5. Secrets for PostgreSQL credentials
6. NetworkPolicies

## Remaining Storage Work

- emptyDir
- PersistentVolume
- PersistentVolumeClaim
- StorageClass
- dynamic provisioning
- access modes
- reclaim policies
- storage troubleshooting
- PostgreSQL persistence

## Remaining Networking Work

- Pod networking
- Services
- DNS/service discovery
- Endpoints
- NetworkPolicies
- Ingress
- networking troubleshooting

## Learning Method

Every topic follows:

```text
Study -> Understand -> Implement -> Break -> Troubleshoot -> Explain
```

The goal is CKA-level operational understanding, not command memorization.

## Final Goal

One integrated project covering:

```text
Authentication
  -> KubeConfig
  -> Authorization / RBAC
  -> Certificates
  -> Security Context
  -> Image Security
  -> NetworkPolicy
  -> Services / DNS
  -> Storage
  -> Ingress / Networking
  -> Troubleshooting
```
