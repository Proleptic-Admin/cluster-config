# Cluster Configuration

Purpose: This repository contains Argo CD platform App-of-Apps configuration for managing infrastructure components across multiple environments.

## Overview

This repository implements an Argo CD "platform" App-of-Apps pattern to manage:

- **nginx ingress controller** - Load balancing and ingress management
- **cert-manager** - Automatic TLS certificate management with ACME DNS-01
- **ExternalDNS** - Automatic DNS record management with OpenStack Designate
- **metrics-server** - Cluster metrics collection
- **KEDA** - Kubernetes Event-driven Autoscaling
- **Observability Stack** - Victoria Metrics, Grafana, and Alertmanager for monitoring
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
├── projects/                 # Argo CD Projects
└── observability-appset.yaml # Observability ApplicationSet

manifests/                    # Kubernetes manifests managed by Kustomize
├── rbac/
├── network-policies/
├── quotas/
├── cert-manager/
└── namespaces/

observability/                # Observability stack configuration
├── victoria-metrics/         # Victoria Metrics time-series database
├── grafana/                  # Grafana dashboards and visualization
├── alertmanager/             # Alert routing and notification
└── policies/                 # Network policies and resource quotas

services/
└── _examples/                # Example monitoring configurations

overlays/                     # Environment-specific configurations
├── dev/                      # Development environment
└── prod/                     # Production environment
```

## Features

- **Automated sync** - All applications sync automatically with self-heal enabled
- **Environment overlays** - Separate configurations for dev/prod via Kustomize
- **Security** - Network policies with default deny, RBAC, and resource quotas
- **High availability** - Production configurations include multi-replica deployments
- **Monitoring** - Comprehensive observability with Victoria Metrics, Grafana, and Alertmanager
- **Multi-cluster** - ApplicationSets for deploying across multiple clusters

## Observability Stack

The observability stack provides comprehensive monitoring and alerting capabilities:

### Components

- **Victoria Metrics** - High-performance time-series database with 30-day retention
- **Grafana** - Visualization and dashboarding with OIDC authentication
- **Alertmanager** - Alert routing with environment-specific receivers

### Adding Monitoring to Services

#### 1. Pod Monitoring with VMPodScrape

Create a `VMPodScrape` resource to monitor your application pods:

```yaml
apiVersion: operator.victoriametrics.com/v1beta1
kind: VMPodScrape
metadata:
  name: my-app-pods
  namespace: my-namespace
spec:
  selector:
    matchLabels:
      app: my-app
  podMetricsEndpoints:
  - port: metrics
    path: /metrics
    interval: 30s
```

#### 2. Service Monitoring with VMServiceScrape

Create a `VMServiceScrape` resource to monitor services:

```yaml
apiVersion: operator.victoriametrics.com/v1beta1
kind: VMServiceScrape
metadata:
  name: my-app-service
  namespace: my-namespace
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    path: /metrics
    interval: 30s
```

#### 3. Alerting with VMRule

Create alert rules using `VMRule`:

```yaml
apiVersion: operator.victoriametrics.com/v1beta1
kind: VMRule
metadata:
  name: my-app-alerts
  namespace: my-namespace
spec:
  groups:
  - name: my-app
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
      labels:
        severity: critical
      annotations:
        summary: "High error rate detected"
```

#### 4. Dashboard Management

Add Grafana dashboards by:

1. Creating dashboard JSON files in `observability/grafana/provisioning/dashboards/dashboards/`
2. Committing the files to git
3. Argo CD will automatically sync the new dashboards

### Example Configurations

See `services/_examples/` for complete examples of:
- `vmrule-example.yaml` - Alert rule configurations
- `vmpod-scrape-example.yaml` - Pod and service monitoring configurations

## Deployment

### Prerequisites

- Kubernetes cluster with Argo CD installed
- Proper RBAC permissions for Argo CD
- OpenStack Designate credentials (for ExternalDNS)
- TSIG secrets (for cert-manager DNS-01)
- Clusters labeled with `tier: shared` or `tier: dedicated` for observability deployment

### Installation

1. **Deploy Argo CD Project:**
   ```bash
   kubectl apply -f apps/projects/platform-project.yaml
   ```

2. **Deploy Platform App-of-Apps:**
   ```bash
   kubectl apply -f apps/platform/platform-app-of-apps.yaml
   ```

3. **Deploy Observability ApplicationSet:**
   ```bash
   kubectl apply -f apps/observability-appset.yaml
   ```

4. **Update secrets with actual credentials:**
   - Update `manifests/cert-manager/external-dns-secret.yaml` with real OpenStack credentials
   - Update `manifests/cert-manager/cluster-issuers.yaml` with real email and DNS server
   - Update observability configurations with real OIDC, SMTP, and S3 credentials

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
- Observability stack communication
- DNS resolution
- Kubernetes API access

### Resource Quotas

Environment-specific quotas:
- **Development**: 2 CPU, 4Gi RAM, 10 pods
- **Production**: 16 CPU, 32Gi RAM, 100 pods
- **Platform**: 8 CPU, 16Gi RAM, 50 pods
- **Observability**: 8 CPU, 16Gi RAM, 30 pods

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
- Observability stack includes security contexts and network isolation

## Monitoring

Platform components include:
- Prometheus metrics endpoints
- ServiceMonitor configurations
- Health checks and readiness probes
- Comprehensive alerting for critical services
- Centralized logging and metrics collection via Victoria Metrics
