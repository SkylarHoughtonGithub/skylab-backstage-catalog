# Tetris Deployment Guide

## Overview
This guide covers deploying the Tetris game application using the `avian19/tetrisv1:latest` Docker image to our Kubernetes cluster via ArgoCD.

## Prerequisites
- ArgoCD access and permissions
- kubectl access to the target cluster
- Access to the workloads repository

## Deployment Architecture

```
Internet → Ingress Controller → Service → Deployment → Pod(s)
                                                    ↓
                                             avian19/tetrisv1:latest
```

## Standard Deployment Process

### 1. Update Manifests
Navigate to your workloads repository and update the Tetris manifests:

```bash
cd skylab-argo-workloads/worloads/tetris-react/dev/
```

Key files to review:
- `kustomization.yaml` - Kustomization file to hold dependent resources
- `deployment.yaml` - Container specs and replicas
- `service.yaml` - Service configuration
- `ingress.yaml` - External access configuration
- `namespace.yaml` - Namespace setup

### 2. Commit and Push Changes
```bash
git add .
git commit -m "Update Tetris deployment configuration"
git push origin main
```

### 3. ArgoCD Sync
ArgoCD should automatically detect changes within 3 minutes, or manually sync:

**Via ArgoCD UI:**
1. Navigate to ArgoCD dashboard
2. Find `tetris-react` application
3. Click "Sync" → "Synchronize"

**Via CLI:**
```bash
argocd app sync tetris-react
```

### 4. Verify Deployment
Check deployment status:
```bash
kubectl get pods -n dev -l app=tetris-react
kubectl get svc -n dev -l app=tetris-react
kubectl get ingress -n dev tetris-ingress
```

Expected output:
```
NAME                           READY   STATUS    RESTARTS   AGE
tetris-react-xxx-xxx          1/1     Running   0          2m

NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
tetris-service     ClusterIP   10.110.xxx.xxx   <none>        80/TCP    2m

NAME     CLASS    HOSTS                              ADDRESS                                                                                            PORTS   AGE
tetris  traefik   tetris.skylarhoughtongithub.local  192.168.80.101,192.168.80.102,192.168.80.103,192.168.80.111,192.168.80.112,192.168.80.113          80      2m
```

## Configuration Parameters

### Environment Variables
The Tetris container may accept these environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3000` | Internal container port |

### Resource Limits
Standard resource allocation:
- **CPU Request:** 100m
- **CPU Limit:** 500m  
- **Memory Request:** 128Mi
- **Memory Limit:** 512Mi

### Scaling
To scale the application:
```bash
kubectl scale deployment tetris-react -n dev --replicas=3
```

Or update `deployment.yaml`:
```yaml
spec:
  replicas: 3  # Increase from 1
```

## Rollback Procedures

### Via ArgoCD
1. Go to ArgoCD UI → tetris-react app
2. Click "History and Rollback"
3. Select previous revision
4. Click "Rollback"

### Via kubectl
```bash
# View rollout history
kubectl rollout history deployment/tetris-react -n dev

# Rollback to previous version
kubectl rollout undo deployment/tetris-react -n dev

# Rollback to specific revision
kubectl rollout undo deployment/tetris-react -n dev --to-revision=2
```

## Health Checks
The deployment includes:
- **Readiness Probe:** HTTP GET on port 3000 at `/`
- **Liveness Probe:** HTTP GET on port 3000 at `/`
- **Startup time:** 30 seconds initial delay

## Network Configuration
- **Internal Service:** `tetris-svc.dev.svc.cluster.local:80`
- **External Access:** `http://tetris.skylarhoughtongithub.local`
- **Service Type:** ClusterIP (internal only)
- **Ingress Class:** Default nginx ingress controller

## Security Considerations
- Container runs as non-root user
- No privileged access required
- Only port 80/3000 exposed
- ReadOnlyRootFilesystem: true (if configured)

## Post-Deployment Testing
1. **Basic connectivity:**
   ```bash
   curl -I http://tetris.skylarhoughtongithub.local
   ```
   Expected: `HTTP/1.1 200 OK`

2. **Game loads properly:**
   - Visit http://tetris.skylarhoughtongithub.local
   - Verify game interface renders
   - Test basic game controls

3. **Health endpoint (if available):**
   ```bash
   kubectl port-forward -n dev svc/tetris-svc 8080:80
   curl http://localhost:8080/health
   ```

## Monitoring and Logs
View application logs:
```bash
# Real-time logs
kubectl logs -f -n dev deployment/tetris-react

# Last 100 lines
kubectl logs -n dev deployment/tetris-react --tail=100

# Logs from all replicas
kubectl logs -n dev -l app=tetris-react --all-containers=true
```