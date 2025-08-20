# Troubleshooting

## Common Issues

### Cluster Not Appearing
1. Check ArgoCD sync status
2. Verify ApplicationSet is running
3. Check cluster labels match ApplicationSet selectors

### Resources Stuck in Provisioning
1. Check Crossplane provider logs:
   ```bash
   kubectl logs -n crossplane-system deployment/crossplane-provider-aws
   ```
2. Verify AWS credentials and permissions
3. Check resource events in AWS console

### Access Denied
1. Verify IAM roles are properly attached
2. Check security group rules
3. Confirm VPC and subnet configuration

## Useful Commands

```bash
# Check all Crossplane resources for this cluster
kubectl get managed -A -l cluster-name=${{ values.name }}

# View Crossplane events
kubectl get events -n crossplane-system

# Check provider configuration
kubectl get providerconfigs

# View composition status
kubectl describe composition ${{ values.name }}-eks-cluster
```

## Support Contacts

- **Platform Team**: platform-team@skylab.com
- **On-call**: See PagerDuty rotation
