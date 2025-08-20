# ${{ values.name | title }} Deployment Guide

## Overview
This guide covers deploying the ${{ values.name | title }} game application using the `${{ values.image }}` Docker image to the **${{ values.targetCluster }}** Kubernetes cluster via ArgoCD.

## Prerequisites
- ArgoCD access and permissions
- kubectl access to the **${{ values.targetCluster }}** cluster
- Access to the backstage configs repository

## Deployment Architecture

```
Internet → Ingress Controller → Service → Deployment → Pod(s)
                                                    ↓
                                             ${{ values.image }}
```

## Configuration Parameters

### Container Configuration
- **Image**: `${{ values.image }}`
- **Replicas**: ${{ values.replicas }}
- **Port**: ${{ values.containerPort }}

### Resource Allocation
- **CPU Request**: ${{ values.cpuRequest }}
- **CPU Limit**: ${{ values.cpuLimit }}
- **Memory Request**: ${{ values.memoryRequest }}
- **Memory Limit**: ${{ values.memoryLimit }}

### Network Configuration
- **Service Type**: ${{ values.serviceType }}
- **Namespace**: `${{ values.name }}-${{ values.environment }}`
{%- if values.enableIngress %}
- **External Access**: https://${{ values.subdomain | default(values.name) }}.skylarhoughtongithub.local
{%- endif %}

## Deployment Process

### 1. Update Manifests
Navigate to your backstage configs repository:

```bash
cd skylab-backstage-configs/apps/${{ values.environment }}/${{ values.name }}/
```

Key files:
- `kustomization.yaml` - Kustomization configuration
- `deployment.yaml` - Container specs and replicas
- `service.yaml` - Service configuration
{%- if values.enableIngress %}
- `ingress.yaml` - External access configuration
{%- endif %}
- `namespace.yaml` - Namespace setup

### 2. Commit and Push Changes
```bash
git add .
git commit -m "Update ${{ values.name }} deployment configuration"
git push origin main
```

### 3. ArgoCD Sync
ArgoCD should automatically detect changes, or manually sync:

**Via ArgoCD UI:**
1. Navigate to ArgoCD dashboard
2. Find `${{ values.name }}-${{ values.environment }}` application
3. Click "Sync" → "Synchronize"

**Via CLI:**
```bash
argocd app sync ${{ values.name }}-${{ values.environment }}
```

### 4. Verify Deployment
Check deployment status:
```bash
kubectl get pods -n ${{ values.name }}-${{ values.environment }} -l app=${{ values.name }}
kubectl get svc -n ${{ values.name }}-${{ values.environment }} -l app=${{ values.name }}
{%- if values.enableIngress %}
kubectl get ingress -n ${{ values.name }}-${{ values.environment }} ${{ values.name }}-ingress
{%- endif %}
```

## Scaling
To scale the application:
```bash
kubectl scale deployment ${{ values.name }}-deployment -n ${{ values.name }}-${{ values.environment }} --replicas=3
```

## Health Checks
{%- if values.enableHealthChecks %}
The deployment includes:
- **Readiness Probe**: HTTP GET on port ${{ values.containerPort }} at `${{ values.healthCheckPath }}`
- **Liveness Probe**: HTTP GET on port ${{ values.containerPort }} at `${{ values.healthCheckPath }}`
- **Startup time**: 30 seconds initial delay
{%- else %}
Health checks are disabled for this deployment.
{%- endif %}

## Post-Deployment Testing
1. **Basic connectivity:**
   {%- if values.enableIngress and values.subdomain %}
   ```bash
   curl -I https://${{ values.subdomain }}.skylarhoughtongithub.local
   ```
   {%- else %}
   ```bash
   kubectl port-forward -n ${{ values.name }}-${{ values.environment }} svc/${{ values.name }}-service 8080:80
   curl http://localhost:8080
   ```
   {%- endif %}

2. **Game loads properly:**
   {%- if values.enableIngress and values.subdomain %}
   - Visit https://${{ values.subdomain }}.skylarhoughtongithub.local
   {%- else %}
   - Port-forward and visit http://localhost:8080
   {%- endif %}
   - Verify game interface renders
   - Test basic game controls

## Monitoring and Logs
View application logs:
```bash
# Real-time logs
kubectl logs -f -n ${{ values.name }}-${{ values.environment }} deployment/${{ values.name }}-deployment

# Last 100 lines
kubectl logs -n ${{ values.name }}-${{ values.environment }} deployment/${{ values.name }}-deployment --tail=100

# Logs from all replicas
kubectl logs -n ${{ values.name }}-${{ values.environment }} -l app=${{ values.name }} --all-containers=true
```
