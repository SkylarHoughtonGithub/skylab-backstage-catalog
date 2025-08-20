# ${{ values.name | title }} Game

## Overview
This is a ${{ values.name | title }} game implementation using the `${{ values.image }}` Docker image, deployed on the **${{ values.targetCluster }}** Kubernetes cluster in the **${{ values.environment }}** environment.

## Configuration

| Parameter | Value |
|-----------|-------|
| Application Name | ${{ values.name }} |
| Environment | ${{ values.environment }} |
| Target Cluster | ${{ values.targetCluster }} |
| Image | ${{ values.image }} |
| Replicas | ${{ values.replicas }} |
| Resources | ${{ values.cpuRequest }}/${{ values.cpuLimit }} CPU, ${{ values.memoryRequest }}/${{ values.memoryLimit }} Memory |
{%- if values.enableIngress and values.subdomain %}
| External URL | https://${{ values.subdomain }}.skylarhoughtongithub.local |
{%- endif %}

## Quick Links
{%- if values.enableIngress and values.subdomain %}
- [Live Game](https://${{ values.subdomain }}.skylarhoughtongithub.local)
{%- endif %}
- [Docker Image](https://hub.docker.com/r/${{ values.image.split(':')[0] }})
- [ArgoCD Application](https://argocd.skylarhoughtongithub.local/applications?search=${{ values.name }}-${{ values.environment }})
- [Source Code](https://github.com/SkylarHoughtonGithub/skylab-backstage-configs/tree/main/apps/${{ values.environment }}/${{ values.name }})

## Architecture
The application runs as a containerized React app with the following components:
- **Frontend**: React-based ${{ values.name | title }} game
- **Deployment**: Kubernetes via ArgoCD on **${{ values.targetCluster }}** cluster
- **Namespace**: `${{ values.name }}-${{ values.environment }}`
{%- if values.enableIngress %}
- **Ingress**: Available at ${{ values.subdomain | default(values.name) }}.skylarhoughtongithub.local
{%- endif %}
