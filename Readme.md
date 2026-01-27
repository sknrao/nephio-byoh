# Nephio Integration Workflow - Complete Guide

## Starting Point

✅ **Prerequisites (Already Done):**
- Nephio management cluster is running
- `my-ran` workload cluster is running (with Calico CNI)
- `my-core` workload cluster is running (with Calico CNI)
- `kubectl` configured with contexts for all 3 clusters
- GitHub account with permissions to create repositories

## End Goal

After completing this workflow:
- ✅ Both workload clusters registered with Nephio
- ✅ Infrastructure packages deployed (ConfigSync, storage, networking)
- ✅ GitOps-based management enabled
- ✅ Ready to deploy 5G network functions (OAI RAN, Free5GC)

---

## Workflow Overview

```
Step 1: Create 4 Git Repositories
         ↓
Step 2: Populate Blueprint Repo with Packages
         ↓
Step 3: Populate Management Config Repo
         ↓
Step 4: Apply Management Configuration
         ↓
Step 5: Verify Repository Registration
         ↓
Step 6: Create and Apply PackageVariants
         ↓
Step 7: Approve PackageRevisions
         ↓
Step 8: Bootstrap ConfigSync on Workload Clusters
         ↓
Step 9: Verify Deployment
         ↓
Step 10: Test Everything
```

**Estimated Time:** 90-120 minutes

## Step 1: Create 4 Git Repositories

```bash
# Set your GitHub organization/username
export GITHUB_ORG="your-github-username"

# Create all 4 repositories
gh repo create ${GITHUB_ORG}/nephio-management-config --public
gh repo create ${GITHUB_ORG}/nephio-blueprints --public
gh repo create ${GITHUB_ORG}/nephio-my-ran --public
gh repo create ${GITHUB_ORG}/nephio-my-core --public
```

## Step 2: Populate Blueprint Repo with Packages

Copy from nephio-blueprints.

## Step 3: Populate Management Config Repo

Copy from nephio-management-config

## Step 4: Apply Management Configuration

```bash
# Switch to management cluster
kubectl config use-context nephio-mgmt

# Verify you're on the right cluster
kubectl cluster-info

# Apply ClusterContexts
kubectl apply -k cluster-contexts/

# Expected output:
# clustercontext.infra.nephio.org/my-ran created
# clustercontext.infra.nephio.org/my-core created

# Apply Repository CRs
kubectl apply -k repositories/

# Expected output:
gerr# repository.config.porch.kpt.dev/nephio-blueprints created
# repository.config.porch.kpt.dev/nephio-my-ran created
# repository.config.porch.kpt.dev/nephio-my-core created
```

**✅ Checkpoint:** Clusters registered, repositories registered with Porch

## Step 5: Verify Repository Registration

### 5.1 Check ClusterContexts

```bash
kubectl get clustercontexts

# Expected output:
# NAME      AGE
# my-ran    30s
# my-core   30s
```

### 5.2 Check Repositories

```bash
kubectl get repositories

# Expected output (wait 30-60 seconds for Ready):
# NAME                 TYPE   CONTENT   DEPLOYMENT   READY
# nephio-blueprints    git    Package   false        True
# nephio-my-ran        git    Package   true         True
# nephio-my-core       git    Package   true         True
```

### 5.3 Check Package Discovery

```bash
# Wait for Porch to discover packages (may take 1-2 minutes)
sleep 60

kubectl get packagerevisions

# Expected output (should show packages from blueprints repo):
# NAME                                    PACKAGE              REVISION   ...
# nephio-blueprints-xxx-cluster-baseline  cluster-baseline     v1         ...
# nephio-blueprints-xxx-platform-addons   platform-addons      v1         ...
# nephio-blueprints-xxx-multus-cni        multus-cni          v1         ...
# nephio-blueprints-xxx-whereabouts-ipam  whereabouts-ipam    v1         ...
```

**✅ Checkpoint:** Porch has discovered all packages from blueprint repo

---

## Step 6: Create and Apply PackageVariants

### 6.1 Create cluster-baseline,platform-addons, and networking PackageVariants

When you copied from management-config its already created.

### 6.2 Commit and Apply

```bash
# Commit PackageVariants to git
git add .
git commit -m "Add PackageVariants for baseline, addons, and networking"
git push

# Apply cluster-baseline PackageVariants
kubectl apply -k packagevariants/baseline/

# Wait and check
sleep 10
kubectl get packagevariants | grep baseline

# Apply platform-addons PackageVariants
kubectl apply -k packagevariants/addons/

# Wait and check
sleep 10
kubectl get packagevariants | grep addons

# Apply networking PackageVariants
kubectl apply -k packagevariants/networking/

# Wait and check
sleep 10
kubectl get packagevariants | grep -E "multus|whereabouts"
```

**✅ Checkpoint:** All PackageVariants created and applied

---

## Step 7: Approve PackageRevisions

### 7.1 Wait for Porch to Render Packages

```bash
# Wait for rendering (takes 30-60 seconds)
echo "Waiting for Porch to render packages..."
sleep 60

# Check PackageRevisions
kubectl get packagerevisions | grep -E "my-ran|my-core"

# Expected to see packages in "Draft" state
```

### 7.2 Approve All PackageRevisions

```bash
# Approve all Draft packages for my-ran
kubectl get packagerevisions -o name | grep my-ran | grep -v blueprints | while read pr; do
  echo "Approving $pr"
  kubectl patch $pr --type=merge -p '{"spec":{"lifecycle":"Published"}}'
done

# Approve all Draft packages for my-core
kubectl get packagerevisions -o name | grep my-core | grep -v blueprints | while read pr; do
  echo "Approving $pr"
  kubectl patch $pr --type=merge -p '{"spec":{"lifecycle":"Published"}}'
done

# Verify all are Published
kubectl get packagerevisions | grep -E "my-ran|my-core"
```

### 7.3 Verify Git Commits

```bash
# Check nephio-my-ran repo
cd ../nephio-my-ran
git pull

ls -la
# Should see: cluster-baseline/, platform-addons/, multus-cni/, whereabouts-ipam/

# Check nephio-my-core repo
cd ../nephio-my-core
git pull

ls -la
# Should see: cluster-baseline/, platform-addons/, multus-cni/, whereabouts-ipam/
```

**✅ Checkpoint:** All packages rendered and committed to downstream repos

---

## Step 9: Verify Deployment (10 min)

### 9.1 Verify ConfigSync on Both Clusters

```bash
# On my-ran cluster
kubectl config use-context my-ran

# Check ConfigSync status
kubectl get rootsync -n config-management-system

# Expected output:
# NAME        SOURCECOMMIT   SYNCCOMMIT     STATUS
# root-sync   abc123...      abc123...      SYNCED

# Check ConfigSync pods
kubectl get pods -n config-management-system

# Expected: All pods Running

# On my-core cluster
kubectl config use-context my-core

kubectl get rootsync -n config-management-system
kubectl get pods -n config-management-system
```

### 9.2 Verify Namespaces Created

```bash
# On my-ran
kubectl config use-context my-ran
kubectl get namespaces | grep -E "openairinterface|local-path"

# Expected:
# openairinterface
# local-path-storage
# config-management-system

# On my-core
kubectl config use-context my-core
kubectl get namespaces | grep -E "free5gc|local-path"

# Expected:
# free5gc
# local-path-storage
# config-management-system
```

### 9.3 Verify Storage Deployed

```bash
# On my-ran
kubectl config use-context my-ran

kubectl get storageclass
# Expected: local-path (default)

kubectl get pods -n local-path-storage
# Expected: local-path-provisioner pod Running

# On my-core
kubectl config use-context my-core

kubectl get storageclass
kubectl get pods -n local-path-storage
```

### 9.4 Verify Networking Deployed

```bash
# On my-ran
kubectl config use-context my-ran

kubectl get daemonsets -n kube-system | grep -E "multus|whereabouts"
# Expected: kube-multus-ds, whereabouts

kubectl get pods -n kube-system -l name=multus
kubectl get pods -n kube-system -l name=whereabouts

# On my-core
kubectl config use-context my-core

kubectl get daemonsets -n kube-system | grep -E "multus|whereabouts"
kubectl get pods -n kube-system -l name=multus
kubectl get pods -n kube-system -l name=whereabouts
```

**✅ Checkpoint:** All infrastructure components deployed to both clusters

---

## Step 10: Test Everything (15 min)

### 10.1 Test Storage on my-ran

```bash
kubectl config use-context my-ran

# Create test PVC
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-path
EOF

# Wait and check
sleep 10
kubectl get pvc test-pvc

# Expected: STATUS = Bound

# Create test pod using PVC
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: default
spec:
  containers:
  - name: test
    image: busybox
    command: ["sh", "-c", "echo 'Storage test successful' > /data/test.txt && sleep 3600"]
    volumeMounts:
    - name: test-volume
      mountPath: /data
  volumes:
  - name: test-volume
    persistentVolumeClaim:
      claimName: test-pvc
EOF

# Wait and check
sleep 20
kubectl get pod test-pod

# Expected: STATUS = Running

# Verify data written
kubectl exec test-pod -- cat /data/test.txt
# Expected: Storage test successful

# Cleanup
kubectl delete pod test-pod
kubectl delete pvc test-pvc
```

### 10.2 Test Multus on my-ran

```bash
# Check Multus is ready
kubectl get daemonset -n kube-system kube-multus-ds

# Check Multus logs (should show no errors)
kubectl logs -n kube-system -l name=multus --tail=50

# Success if no errors in logs
```

### 10.3 Test Whereabouts on my-ran

```bash
# Check Whereabouts is ready
kubectl get daemonset -n kube-system whereabouts

# Check Whereabouts logs
kubectl logs -n kube-system -l name=whereabouts --tail=50

# Success if no errors in logs
```

### 10.4 Repeat Tests on my-core

```bash
kubectl config use-context my-core

# Test storage (same steps as above)
# Test Multus (same steps as above)
# Test Whereabouts (same steps as above)
```

### 10.5 Summary Check

```bash
# Check everything from management cluster
kubectl config use-context nephio-mgmt

echo "=== ClusterContexts ==="
kubectl get clustercontexts

echo "=== Repositories ==="
kubectl get repositories

echo "=== PackageVariants ==="
kubectl get packagevariants

echo "=== PackageRevisions ==="
kubectl get packagerevisions | grep Published

# Check workload clusters
echo "=== my-ran Status ==="
kubectl --context=my-ran get pods -A | grep -v kube-system | head -20

echo "=== my-core Status ==="
kubectl --context=my-core get pods -A | grep -v kube-system | head -20
```

**✅ Final Checkpoint:** All systems operational and tested!

---
## Completion Checklist

- ✅ 4 git repositories created
- ✅ Blueprint repo populated with packages
- ✅ Management config repo populated with CRs
- ✅ ClusterContexts registered
- ✅ Repositories registered with Porch
- ✅ PackageVariants created and applied
- ✅ PackageRevisions approved
- ✅ ConfigSync bootstrapped on both clusters
- ✅ All infrastructure deployed
- ✅ Storage tested and working
- ✅ Networking tested and working

