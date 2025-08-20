
# ${{ values.name | title }} Troubleshooting Guide

## Common Issues and Solutions

### 🔴 Game Not Loading / 502 Bad Gateway

**Symptoms:**
- Browser shows "502 Bad Gateway" or "Service Unavailable"
{%- if values.enableIngress and values.subdomain %}
- Game page doesn't load at ${{ values.subdomain }}.skylarhoughtongithub.local
{%- else %}
- Port-forwarded application not responding
{%- endif %}

**Diagnostic Steps:**
1. Check pod status:
   ```bash
   kubectl get pods -n ${{ values.name }}-${{ values.environment }} -l app=${{ values.name }}
   ```

2. Check recent events:
   ```bash
   kubectl get events -n ${{ values.name }}-${{ values.environment }} --sort-by='.lastTimestamp' | tail -10
   ```

3. Verify service endpoints:
   ```bash
   kubectl get endpoints -n ${{ values.name }}-${{ values.environment }} ${{ values.name }}-service
   ```

**Common Solutions:**
- Check logs: `kubectl logs -n ${{ values.name }}-${{ values.environment }} deployment/${{ values.name }}-deployment`
- Verify image exists: `docker pull ${{ values.image }}`
- Check resource limits and node capacity
- Ensure port ${{ values.containerPort }} is correct

### 🟡 Performance Issues

**Symptoms:**
- Game loads but is sluggish
- High response times

**Diagnostic Commands:**
```bash
# Check resource usage
kubectl top pods -n ${{ values.name }}-${{ values.environment }} -l app=${{ values.name }}

# Check node resources on cluster: ${{ values.targetCluster }}
kubectl top nodes

{%- if values.enableIngress and values.subdomain %}
# Test response time
curl -w "@curl-format.txt" -o /dev/null -s https://${{ values.subdomain }}.skylarhoughtongithub.local
{%- endif %}
```

**Solutions:**
- Scale up replicas: `kubectl scale deployment ${{ values.name }}-deployment -n ${{ values.name }}-${{ values.environment }} --replicas=${{ values.replicas + 1 }}`
- Review resource limits: CPU: ${{ values.cpuLimit }}, Memory: ${{ values.memoryLimit }}

### 🔴 ArgoCD Sync Failures

**Symptoms:**
- ArgoCD shows "OutOfSync" status
- Red health status in ArgoCD UI

**Diagnostic Steps:**
```bash
# Check ArgoCD application status
argocd app get ${{ values.name }}-${{ values.environment }}

# View sync errors
argocd app sync ${{ values.name }}-${{ values.environment }} --dry-run
```

### 🔴 Cluster Access Issues

**Target Cluster**: ${{ values.targetCluster }}

Make sure you have access to the correct cluster and that the cluster is healthy:

```bash
# Check cluster connection
kubectl cluster-info

# Verify you're connected to the right cluster
kubectl config current-context

# Check cluster nodes
kubectl get nodes
```

## Emergency Procedures

### 🚨 Quick Restart
```bash
kubectl rollout restart deployment/${{ values.name }}-deployment -n ${{ values.name }}-${{ values.environment }}
```

### 🆘 Quick Health Check Script
```bash
#!/bin/bash
echo "=== ${{ values.name | title }} Health Check ==="

echo "1. Pod Status:"
kubectl get pods -n ${{ values.name }}-${{ values.environment }} -l app=${{ values.name }}

echo -e "\n2. Service Status:"
kubectl get svc -n ${{ values.name }}-${{ values.environment }} ${{ values.name }}-service

{%- if values.enableIngress