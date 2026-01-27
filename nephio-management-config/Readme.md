# Nephio Management Configuration

This repository contains all Custom Resources for managing the Nephio deployment:

## Structure

- `cluster-contexts/` - ClusterContext CRs for registering workload clusters
- `repositories/` - Repository CRs for registering git repos with Porch
- `packagevariants/` - PackageVariant CRs for deploying packages to clusters

## Usage

```bash
# Apply all configuration to management cluster
kubectl config use-context nephio-mgmt
kubectl apply -k .

# Or apply selectively
kubectl apply -k cluster-contexts/
kubectl apply -k repositories/
kubectl apply -k packagevariants/
```

## Repositories

This creates:
- **ClusterContexts**: Registers my-ran and my-core clusters
- **Repository CRs**: Points Porch to blueprint and deployment repos
- **PackageVariants**: Instructions for deploying packages

## Notes

- This is NOT a Porch Repository (deployment: false)
- Contents are applied directly with kubectl
- Changes require manual kubectl apply (no auto-sync)
