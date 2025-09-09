# Cluster Configuration

Purpose: This repository contains Argo CD platform App-of-Apps configuration for managing infrastructure components across multiple environments.

## Overview

This repository implements an Argo CD "platform" App-of-Apps pattern to manage:

- **nginx ingress controller** - Load balancing and ingress management
- **cert-manager** - Automatic TLS certificate management with ACME DNS-01
- **ExternalDNS** - Automatic DNS record management with OpenStack Designate
- **metrics-server** - Cluster metrics collection
- **KEDA** - Kubernetes Event-driven Autoscaling
- **RBAC** - Role-based access control
- **NetworkPolicies** - Network security policies
- **Resource Quotas** - Resource limits and quotas
- **Argo Projects** - Argo CD project governance

## Architecture

```
apps/
├── platform/                 # Platform App-of-Apps
│   ├── platform-app-of-apps.yaml
│   └── apps/                 # Individual platform applications
└── projects/                 # Argo CD Projects

manifests/                    # Kubernetes manifests managed by Kustomize
├── rbac/
├── network-policies/
├── quotas/
├── cert-manager/
└── namespaces/

overlays/                     # Environment-specific configurations
├── dev/                      # Development environment
└── prod/                     # Production environment
```

## Features

- **Automated sync** - All applications sync automatically with self-heal enabled
- **Environment overlays** - Separate configurations for dev/prod via Kustomize
- **Security** - Network policies with default deny, RBAC, and resource quotas
- **High availability** - Production configurations include multi-replica deployments
- **Monitoring** - Prometheus metrics and ServiceMonitors where supported

## Deployment

### Prerequisites

- Kubernetes cluster with Argo CD installed
- Proper RBAC permissions for Argo CD
- OpenStack Designate credentials (for ExternalDNS)
- TSIG secrets (for cert-manager DNS-01)

### Installation

1. **Deploy Argo CD Project:**
   ```bash
   kubectl apply -f apps/projects/platform-project.yaml
   ```

2. **Deploy Platform App-of-Apps:**
   ```bash
   kubectl apply -f apps/platform/platform-app-of-apps.yaml
   ```

3. **Update secrets with actual credentials:**
   - Update `manifests/cert-manager/external-dns-secret.yaml` with real OpenStack credentials
   - Update `manifests/cert-manager/cluster-issuers.yaml` with real email and DNS server

### Environment Deployment

For environment-specific deployments:

```bash
# Development
kustomize build overlays/dev | kubectl apply -f -

# Production  
kustomize build overlays/prod | kubectl apply -f -
```

## Configuration

### Network Policies

Default deny-all policies are implemented with specific allows for:
- nginx ingress traffic
- Platform component communication
- DNS resolution
- Kubernetes API access

### Resource Quotas

Environment-specific quotas:
- **Development**: 2 CPU, 4Gi RAM, 10 pods
- **Production**: 16 CPU, 32Gi RAM, 100 pods
- **Platform**: 8 CPU, 16Gi RAM, 50 pods

### RBAC

Three main roles:
- **platform-admin**: Full cluster access
- **platform-operator**: Platform component management
- **developer**: Application deployment access

## Security Considerations

- All applications use automated sync with self-heal
- Network policies implement least-privilege access
- Resource quotas prevent resource exhaustion
- RBAC provides role-based access control
- Secrets should be managed externally (e.g., External Secrets Operator)

## Monitoring

Platform components include:
- Prometheus metrics endpoints
- ServiceMonitor configurations
- Health checks and readiness probes
