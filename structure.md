#==============================================================================
# REPOSITORY 1: nephio-management-config
# Type: Regular git repo (NOT a Porch Repository)
# Purpose: Store all management cluster configuration
# URL: https://github.com/YOUR-ORG/nephio-management-config.git
#==============================================================================

nephio-management-config/
│
├── README.md
├── kustomization.yaml                   # Root kustomization
│
├── cluster-contexts/
│   ├── kustomization.yaml
│   ├── clustercontext-my-ran.yaml
│   └── clustercontext-my-core.yaml
│
├── repositories/
│   ├── kustomization.yaml
│   ├── repository-blueprints.yaml       # Repository CR for blueprints
│   ├── repository-my-ran.yaml           # Repository CR for ran
│   ├── repository-my-core.yaml          # Repository CR for core
│   └── git-credentials-secret.yaml      # Optional, if using private repos
│
└── packagevariants/
    ├── kustomization.yaml
    │
    ├── baseline/
    │   ├── kustomization.yaml
    │   ├── baseline-my-ran.yaml         # PackageVariant CR
    │   └── baseline-my-core.yaml        # PackageVariant CR
    │
    ├── addons/
    │   ├── kustomization.yaml
    │   ├── addons-my-ran.yaml
    │   └── addons-my-core.yaml
    │
    └── networking/
        ├── kustomization.yaml
        ├── multus-my-ran.yaml
        ├── multus-my-core.yaml
        ├── whereabouts-my-ran.yaml
        ├── whereabouts-my-core.yaml
        ├── nads-my-ran.yaml
        └── nads-my-core.yaml

# How to use:
#   git clone https://github.com/YOUR-ORG/nephio-management-config.git
#   cd nephio-management-config
#   kubectl config use-context nephio-mgmt
#   kubectl apply -k .


#==============================================================================
# REPOSITORY 2: nephio-blueprints
# Type: Git repo + Porch Repository (deployment: false)
# Purpose: Blueprint/template packages
# URL: https://github.com/YOUR-ORG/nephio-blueprints.git
#==============================================================================

nephio-blueprints/
│
├── README.md
│
├── cluster-baseline/
│   ├── Kptfile
│   ├── configsync.yaml
│   ├── rootsync.yaml
│   ├── git-credentials-secret.yaml
│   ├── pod-security.yaml
│   ├── node-configuration.yaml
│   ├── default-resource-limits.yaml
│   └── storage-class.yaml
│
├── platform-addons/
│   ├── Kptfile
│   ├── storage/
│   │   └── local-path-provisioner.yaml
│   ├── monitoring/
│   │   └── metrics-server.yaml
│   └── resource-management/
│       └── resource-quotas.yaml
│
└── networking/
    ├── multus-cni/
    │   ├── Kptfile
    │   └── multus-daemonset.yaml
    │
    ├── whereabouts-ipam/
    │   ├── Kptfile
    │   └── whereabouts.yaml
    │
    ├── network-intents/
    │   ├── Kptfile
    │   ├── control-plane.yaml
    │   └── user-plane.yaml
    │
    └── network-attachment-renderer/
        ├── Kptfile
        ├── nad-renderer-config.yaml
        └── examples/
            ├── ran-nads.yaml
            └── core-nads.yaml

# How Porch sees it:
#   Registered via Repository CR in nephio-management-config
#   Porch discovers packages automatically
#   Referenced by PackageVariants as upstream source


#==============================================================================
# REPOSITORY 3: nephio-my-ran
# Type: Git repo + Porch Repository (deployment: true)
# Purpose: Rendered packages for RAN cluster
# URL: https://github.com/YOUR-ORG/nephio-my-ran.git
#==============================================================================

nephio-my-ran/
│
├── README.md    # Only file initially - Porch populates the rest
│
# After PackageVariants are processed by Porch:
│
├── cluster-baseline/
│   ├── Kptfile
│   ├── configsync.yaml
│   ├── rootsync.yaml                # git.dir: / or git.repo: nephio-my-ran
│   ├── pod-security.yaml
│   ├── node-configuration.yaml
│   ├── default-resource-limits.yaml
│   ├── storage-class.yaml
│   └── resourcegroup.yaml           # Porch metadata
│
├── platform-addons/
│   ├── Kptfile
│   ├── storage/
│   ├── monitoring/
│   ├── resource-management/
│   └── resourcegroup.yaml
│
├── multus-cni/
│   ├── Kptfile
│   ├── multus-daemonset.yaml
│   └── resourcegroup.yaml
│
├── whereabouts-ipam/
│   ├── Kptfile
│   ├── whereabouts.yaml
│   └── resourcegroup.yaml
│
└── network-attachments/
    ├── Kptfile
    ├── ran-ctrl-net.yaml
    ├── ran-user-net.yaml
    └── resourcegroup.yaml

# How it's used:
#   Porch writes rendered packages here (via PackageVariants)
#   ConfigSync on my-ran cluster watches this repo
#   Automatically applies manifests to my-ran cluster


#==============================================================================
# REPOSITORY 4: nephio-my-core
# Type: Git repo + Porch Repository (deployment: true)
# Purpose: Rendered packages for CORE cluster
# URL: https://github.com/YOUR-ORG/nephio-my-core.git
#==============================================================================

nephio-my-core/
│
├── README.md    # Only file initially - Porch populates the rest
│
# After PackageVariants are processed by Porch:
│
├── cluster-baseline/
│   ├── Kptfile
│   ├── configsync.yaml
│   ├── rootsync.yaml                # git.dir: / or git.repo: nephio-my-core
│   ├── pod-security.yaml
│   ├── node-configuration.yaml
│   ├── default-resource-limits.yaml
│   ├── storage-class.yaml
│   └── resourcegroup.yaml
│
├── platform-addons/
│   ├── Kptfile
│   ├── storage/
│   ├── monitoring/
│   ├── resource-management/
│   └── resourcegroup.yaml
│
├── multus-cni/
│   ├── Kptfile
│   ├── multus-daemonset.yaml
│   └── resourcegroup.yaml
│
├── whereabouts-ipam/
│   ├── Kptfile
│   ├── whereabouts.yaml
│   └── resourcegroup.yaml
│
└── network-attachments/
    ├── Kptfile
    ├── core-ctrl-net.yaml
    ├── core-user-net.yaml
    └── resourcegroup.yaml

# How it's used:
#   Porch writes rendered packages here (via PackageVariants)
#   ConfigSync on my-core cluster watches this repo
#   Automatically applies manifests to my-core cluster


#==============================================================================
# SUMMARY OF 4 REPOSITORIES
#==============================================================================

# 1. nephio-management-config
#    - Regular git repo
#    - Contains: ClusterContexts, Repository CRs, PackageVariants
#    - Applied to: Management cluster (kubectl apply -k)
#    - Watched by: Nobody (manual apply)
#
# 2. nephio-blueprints
#    - Git repo + Porch Repository CR (deployment: false)
#    - Contains: Blueprint packages with ${setters}
#    - Applied to: Management cluster (Repository CR from repo #1)
#    - Watched by: Porch (reads packages)
#
# 3. nephio-my-ran
#    - Git repo + Porch Repository CR (deployment: true)
#    - Contains: Rendered packages for RAN
#    - Applied to: Management cluster (Repository CR from repo #1)
#    - Watched by: Porch (writes), ConfigSync on my-ran (reads)
#
# 4. nephio-my-core
#    - Git repo + Porch Repository CR (deployment: true)
#    - Contains: Rendered packages for CORE
#    - Applied to: Management cluster (Repository CR from repo #1)
#    - Watched by: Porch (writes), ConfigSync on my-core (reads)


#==============================================================================
# RELATIONSHIP DIAGRAM
#==============================================================================

#                    ┌──────────────────────────────────┐
#                    │ nephio-management-config         │
#                    │ (Git Repo - NOT Porch)           │
#                    │                                  │
#                    │ Contains:                        │
#                    │ • ClusterContext CRs             │
#                    │ • Repository CRs ────────────┐   │
#                    │ • PackageVariant CRs         │   │
#                    └──────────────────────────────┼───┘
#                                                   │
#                         kubectl apply -k          │
#                                ↓                  │
#                    ┌──────────────────────────────┼───┐
#                    │ Nephio Management Cluster    │   │
#                    │                              │   │
#                    │ Creates:                     │   │
#                    │ • ClusterContexts            │   │
#                    │ • Repository CRs ────────────┘   │
#                    │ • PackageVariants                │
#                    └──────────────────────────────────┘
#                                ↓
#                         Porch Controller
#                                ↓
#         ┌──────────────────────┼──────────────────────┐
#         │                      │                      │
#         ↓                      ↓                      ↓
# ┌───────────────┐      ┌──────────────┐      ┌──────────────┐
# │ nephio-       │      │ nephio-      │      │ nephio-      │
# │ blueprints    │      │ my-ran       │      │ my-core      │
# │ (reads)       │      │ (writes)     │      │ (writes)     │
# └───────────────┘      └──────────────┘      └──────────────┘
#                               │                      │
#                               │ ConfigSync           │ ConfigSync
#                               ↓                      ↓
#                        ┌──────────────┐      ┌──────────────┐
#                        │ my-ran       │      │ my-core      │
#                        │ cluster      │      │ cluster      │
#                        └──────────────┘      └──────────────┘


#==============================================================================
# FILE COUNTS
#==============================================================================

# nephio-management-config:  ~20 files (CRs for management)
# nephio-blueprints:         ~30 files (package templates)
# nephio-my-ran:             ~35 files (after Porch renders)
# nephio-my-core:            ~35 files (after Porch renders)
#
# Total:                     ~120 files across 4 repositories