# Tetris Troubleshooting Guide

## Common Issues and Solutions

### 🔴 Game Not Loading / 502 Bad Gateway

**Symptoms:**
- Browser shows "502 Bad Gateway" or "Service Unavailable"
- Game page doesn't load at tetris.skylarhoughtongithub.local

**Diagnostic Steps:**
1. Check pod status:
   ```bash
   kubectl get pods -n dev -l app=tetris-react
   ```

2. Check recent events:
   ```bash
   kubectl get events -n dev --sort-by='.lastTimestamp' | tail -10
   ```

3. Verify service endpoints:
   ```bash
   kubectl get endpoints -n dev tetris-svc
   ```

**Common Causes & Solutions:**

| Cause | Solution |
|-------|----------|
| Pod crash looping | Check logs: `kubectl logs -n dev deployment/tetris-react` |
| Image pull failure | Verify image exists: `docker pull avian19/tetrisv1:latest` |
| Resource limits exceeded | Increase memory/CPU limits in deployment.yaml |
| Wrong container port | Ensure port 3000 is exposed in deployment |

### 🟡 Slow Loading or Performance Issues

**Symptoms:**
- Game loads but is sluggish
- High response times
- Intermittent timeouts

**Diagnostic Commands:**
```bash
# Check resource usage
kubectl top pods -n dev -l app=tetris-react

# Check node resources
kubectl top nodes

# Test response time
curl -w "@curl-format.txt" -o /dev/null -s http://tetris.skylarhoughtongithub.local
```

**curl-format.txt content:**
```
time_namelookup:  %{time_namelookup}\n
time_connect:     %{time_connect}\n
time_pretransfer: %{time_pretransfer}\n
time_total:       %{time_total}\n
```

**Solutions:**
- Scale up replicas: `kubectl scale deployment tetris-react -n dev --replicas=2`
- Increase resource limits
- Check node capacity and consider node scaling

### 🔴 ArgoCD Sync Failures

**Symptoms:**
- ArgoCD shows "OutOfSync" status
- Red health status in ArgoCD UI
- Changes not deploying

**Diagnostic Steps:**
1. Check ArgoCD application status:
   ```bash
   argocd app get tetris-react
   ```

2. View sync errors in ArgoCD UI or:
   ```bash
   argocd app sync tetris-react --dry-run
   ```

**Common Issues:**

| Error | Solution |
|-------|----------|
| "resource mapping not found" | Update ArgoCD or check API versions |
| "permission denied" | Verify ArgoCD service account permissions |
| "manifest validation failed" | Check YAML syntax in manifests |
| "namespace not found" | Ensure namespace.yaml is applied first |

### 🟠 DNS/Ingress Issues

**Symptoms:**
- Can't reach tetris.skylarhoughtongithub.local
- DNS resolution fails
- "This site can't be reached"

**Diagnostic Steps:**
1. Test DNS resolution:
   ```bash
   nslookup tetris.skylarhoughtongithub.local
   dig tetris.skylarhoughtongithub.local
   ```

2. Check ingress configuration:
   ```bash
   kubectl describe ingress tetris-ingress -n dev
   kubectl get ingress -n dev -o yaml
   ```

3. Verify ingress controller:
   ```bash
   kubectl get pods -n ingress-nginx  # or your ingress namespace
   ```

**Solutions:**
- Update /etc/hosts file for local testing: `<CLUSTER_IP> tetris.skylarhoughtongithub.local`
- Check ingress controller logs
- Verify ingress class annotations
- Ensure DNS records are configured properly

### 🔴 Container/Image Issues

**Symptoms:**
- Pods stuck in ImagePullBackOff
- ErrImagePull status
- Container exits immediately

**Diagnostic Commands:**
```bash
# Describe pod for detailed error info
kubectl describe pod -n dev -l app=tetris-react

# Check image pull secrets (if needed)
kubectl get secrets -n dev

# Test image pull manually
docker pull avian19/tetrisv1:latest
```

**Solutions:**
- Verify image tag exists on Docker Hub
- Check for typos in image name/tag
- Add imagePullSecrets if using private registry
- Update to latest working image tag

### 🟡 Configuration Issues

**Symptoms:**
- Game loads but doesn't work properly
- Missing features or broken functionality
- Console errors in browser

**Browser Debugging:**
1. Open Developer Tools (F12)
2. Check Console tab for JavaScript errors
3. Check Network tab for failed requests
4. Look for 404s or CORS errors

**Common Config Problems:**
```bash
# Check environment variables
kubectl exec -n dev deployment/tetris-react -- env | grep -E "(PORT|NODE_ENV)"

# Verify mounted configs (if any)
kubectl exec -n dev deployment/tetris-react -- ls -la /app/config/
```

## Emergency Procedures

### 🚨 Complete Service Outage
1. **Immediate actions:**
   ```bash
   # Check overall cluster health
   kubectl get nodes
   kubectl get pods -A | grep -v Running
   
   # Quick restart of tetris deployment
   kubectl rollout restart deployment/tetris-react -n dev
   ```

2. **If restart doesn't help:**
   ```bash
   # Rollback to last working version
   kubectl rollout undo deployment/tetris-react -n dev
   ```

3. **Escalation:** Contact platform team if cluster-wide issues

### 🆘 Quick Health Check Script
Save as `health-check.sh`:
```bash
#!/bin/bash
echo "=== Tetris Health Check ==="

echo "1. Pod Status:"
kubectl get pods -n dev -l app=tetris-react

echo -e "\n2. Service Status:"
kubectl get svc -n dev tetris-svc

echo -e "\n3. Ingress Status:"
kubectl get ingress -n dev tetris-ingress

echo -e "\n4. Recent Events:"
kubectl get events -n dev --sort-by='.lastTimestamp' | tail -5

echo -e "\n5. HTTP Test:"
curl -I -s --max-time 10 http://tetris.skylarhoughtongithub.local || echo "❌ HTTP test failed"

echo -e "\n6. ArgoCD Sync Status:"
argocd app get tetris-react --output json | jq -r '.status.sync.status'
```

Make executable: `chmod +x health-check.sh`

## Log Analysis Patterns

### Normal Startup Logs
Look for these patterns in healthy deployments:
```
Starting the development server...
webpack compiled with 0 errors
Server listening on port 3000
```

### Warning Signs
Watch for these problematic patterns:
```
Error: ENOSPC: no space left on device
FATAL ERROR: Ineffective mark-compacts near heap limit
UnhandledPromiseRejectionWarning
Connection refused to database
```

### Log Commands Cheat Sheet
```bash
# Live tail logs from all replicas
kubectl logs -f -n dev -l app=tetris-react

# Grep for errors in logs
kubectl logs -n dev deployment/tetris-react | grep -i error

# Get logs from previous container (if crashed)
kubectl logs -n dev deployment/tetris-react --previous

# Export logs to file for analysis
kubectl logs -n dev deployment/tetris-react > tetris-logs.txt
```

## Contact Information
- **Platform Team:** platform@shoutsky.com
- **ArgoCD Dashboard:** https://argocd.skylarhoughtongithub.local
- **Monitoring:** https://grafana.skylarhoughtongithub.local