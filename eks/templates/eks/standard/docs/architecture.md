# Architecture

## Infrastructure Components

### Networking
- **VPC**: `${{ values.name }}-vpc` (192.168.0.0/16)
- **Subnets**: 
  - `${{ values.name }}-us-east-2a` (192.168.10.0/24)
  - `${{ values.name }}-us-east-2c` (192.168.11.0/24)
- **Internet Gateway**: `${{ values.name }}-igw`
- **Route Table**: `${{ values.name }}-public-rt`

### Security
- **Security Group**: `${{ values.name }}-sg`
- **Cluster Role**: `${{ values.name }}-cluster-role`
- **Node Role**: `${{ values.name }}-node-role`

### Compute
- **EKS Cluster**: `${{ values.name }}` (Kubernetes 1.28)
- **Node Group**: `${{ values.name }}-nodegroup`

## Crossplane Resources

This cluster is defined as a Crossplane Composition that creates and manages all AWS resources automatically.

### Connection Secret
Cluster credentials are stored in:
- **Namespace**: `crossplane-system`
- **Secret**: `${{ values.name }}-conn`
