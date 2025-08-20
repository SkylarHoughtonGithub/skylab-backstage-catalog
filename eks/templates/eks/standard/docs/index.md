# ${{ values.name | title }} EKS Cluster

## Overview

This EKS cluster is managed by Crossplane and deployed via ArgoCD in the **${{ values.environment }}** environment.

## Configuration

| Parameter | Value |
|-----------|-------|
| Cluster Name | ${{ values.name }} |
| Environment | ${{ values.environment }} |
| Platform | ${{ values.clusterType }} |
| Variant | ${{ values.clusterVariant }} |
| Region | ${{ values.region }} |
| Node Count | ${{ values.nodeCount }} |
| Node Size | ${{ values.nodeSize }} |

## Quick Links

- [AWS Console](https://console.aws.amazon.com/eks/home?region=${{ values.region }}#/clusters/${{ values.name }})
- [ArgoCD Applications](https://argocd.skylarhoughtongithub.local/applications?search=${{ values.name }}-crossplane-infra-${{ values.environment }})
- [Architecture Details](architecture.md)
- [Operations Guide](operations.md)
