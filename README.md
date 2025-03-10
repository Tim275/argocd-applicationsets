# ArgoCD GitOps Workflow

## Overview

This project implements a sophisticated GitOps workflow using ArgoCD with a three-tier architecture pattern that separates manifests, ApplicationSets, and root applications. The implementation enables automated deployments across multiple environments (QA, Staging, Production) with proper dependency management and synchronization.

## Architecture Design

### Three-tier Architecture

The implementation follows a clean separation of concerns with three distinct layers:

1. **Kubernetes Manifests (Category 1)**: Raw application and infrastructure resources
2. **ArgoCD ApplicationSets (Category 2)**: Environment-specific generators that dynamically create applications
3. **Root Application (Category 3)**: Single entry point for bootstrapping the entire platform

### Component Separation

Rather than using one complex ApplicationSet, the implementation separates components by type:

- **Infrastructure ApplicationSet**: Manages core platform components like CloudNative PostgreSQL operator
- **Databases ApplicationSet**: Handles database cluster deployments
- **Applications ApplicationSet**: Controls application deployments with proper dependencies

## Directory Structure

```
kubecraft/
└── argocd/
    ├── apps/                     # Application manifests
    │   └── nginx/                # example application
    │       ├── base/
    │       │   ├── deployment.yaml
    │       │   ├── kustomization.yaml
    │       │   └── service.yaml
    │       └── envs/
    │           ├── production/
    │           ├── qa/
    │           └── staging/
    ├── infrastructure/           # Infrastructure components
    │   ├── cnpg/                 # CloudNative PostgreSQL operator
    │   │   ├── base/
    │   │   │   ├── cnpg-grafana.yaml
    │   │   │   ├── cnpg.yaml
    │   │   │   ├── kustomization.yaml
    │   │   │   └── namespace.yaml
    │   │   └── envs/
    │   └── databases/            # PostgreSQL clusters
    │       └── kubecraft-db/
    │           ├── base/
    │           └── envs/
    ├── appsets/                  # ApplicationSets (separated by type)
    │   ├── kustomization.yaml
    │   ├── qa-apps-appset.yaml
    │   ├── qa-appset.yaml
    │   ├── qa-cnpg-appset.yaml
    │   └── qa-databases-appset.yaml
    └── root-app.yaml            # Root application
```

## Implementation Details

### Sync Wave Orchestration

The implementation uses ArgoCD sync waves to ensure components deploy in the correct order:

1. Infrastructure components are deployed first (sync wave -5)
2. Database clusters are deployed next (sync wave 0)
3. Applications are deployed last (sync wave 5)

This ensures dependencies are properly satisfied before dependent applications are deployed.

### Environment-Specific Configurations

Each application supports multiple environments through Kustomize overlays:
- Production
- Staging
- QA

The ApplicationSets automatically detect and deploy applications with the appropriate environment overlay based on directory structure.

## Benefits

- **Clean Separation of Concerns**: Kubernetes manifests are completely separated from ArgoCD configuration
- **Simplified Development Workflow**: Developers can work with standard Kubernetes tools without needing to understand ArgoCD specifics
- **Automatic Application Discovery**: ApplicationSets automatically detect and deploy applications with appropriate overlays
- **Better Organization**: Using multiple dedicated ApplicationSets instead of one complex one provides clearer organization and prevents changes to one component from affecting others
- **Dependency Management**: Components deploy in the correct order through sync waves

## Setup Instructions

### Bootstrap the Entire Platform

1. Apply the root application to bootstrap everything:
   ```bash
   kubectl apply -f argocd/root-app.yaml
   ```

2. Sync the application:
   ```bash
   argocd app sync root-argocd-app --grpc-web
   ```

### Expected Result

When properly deployed, you should see a dependency graph similar to this in the ArgoCD UI:

![ArgoCD Application Hierarchy](https://github.com/user-attachments/assets/73e5c4a0-af9a-4ea6-a69d-bbbf1dcbedaf)

## References

This implementation was inspired by and follows patterns from:
- [Many AppSets Demo](https://github.com/kostis-codefresh/many-appsets-demo)
- [How to Structure Your Argo CD Repositories Using Application Sets](https://codefresh.io/blog/how-to-structure-your-argo-cd-repositories/)
