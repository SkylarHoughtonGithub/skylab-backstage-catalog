
# Operations Guide

## ArgoCD Integration

**Application Name**: `{cluster-name}-crossplane-infra-${{ values.environment }}`

## Monitoring

### Health Checks
```bash
# Check ArgoCD application
kubectl get applications -n argocd | grep ${{ values.name }}

# Check Crossplane resources
kubectl get composite -A
kubectl get managed -A | grep ${{ values.name }}
```

### Accessing the Cluster
```bash
# Get connection secret
kubectl get secret ${{ values.name }}-conn -n crossplane-system -o yaml

# Update kubeconfig (after extracting from secret)
aws eks update-kubeconfig --region ${{ values.region }} --name ${{ values.name }}
```

## Scaling

### Node Group Scaling
The node group can be scaled by updating the Crossplane composition:
- **Current Size**: ${{ values.nodeCount }}
- **Min Size**: 1
- **Max Size**: ${{ values.nodeCount }}

## Backup & Recovery

### Cluster State
- Infrastructure is code-managed via Crossplane
- Application state should be backed up separately
- EKS control plane is managed by AWS
