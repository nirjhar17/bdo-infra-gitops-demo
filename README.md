# BDO infrastructure GitOps demo

A separate BDO example on the existing lab. No Globe resources are changed.

## First walkthrough: Git to a team namespace

Open the files in `direct-gitops/namespace` on GitHub before creating an Argo CD application.

- `namespace.yaml`: creates `bdo-payments-dev`.
- `rolebinding.yaml`: grants the `bdo-payments-developers` group the edit role within that namespace.
- `resourcequota.yaml`: sets the namespace budget (20 pods, 2 requested CPUs, 4Gi requested memory, 4 CPU and 8Gi memory limits).
- `limitrange.yaml`: provides default container requests and limits.
- `kustomization.yaml`: lists the four files Argo CD reads together.

No application workload is deployed. This walkthrough prepares the workspace in which applications can later run. The group binding does not create users or establish identity-provider group membership.

In the Argo CD GUI choose New App, name `bdo-direct-workspace`, project `bdo-direct`, manual sync, this repository URL, revision `main`, path `direct-gitops/namespace`, cluster `https://kubernetes.default.svc`, and destination namespace `bdo-payments-dev`. Leave auto-create namespace and pruning disabled. Inspect the diff before clicking Sync. The Namespace manifest creates the namespace.

After sync, verify ResourceQuota `team-budget`, LimitRange `container-defaults`, and RoleBinding `developers-edit` in OpenShift. Change the pod quota from 20 to 25 through a reviewed pull request, then manually sync again. Manual mode does not automatically correct drift.

## Second walkthrough: ACM governance

`acm/namespace` contains a BDO Policy, Placement and PlacementBinding. It creates a different namespace, `bdo-acm-demo`, through ACM. Policies live in `bdo-acm-policies` and target only `local-cluster`. The policy starts in inform mode: Argo CD delivers it, and ACM reports NonCompliant while the tenant resources are missing. Change both remediationAction entries to enforce through a reviewed Git change, merge and Sync. The ACM configuration-policy controller then creates the resources and reports Compliant. The direct application does not manage this namespace.

`bootstrap` contains administrator preparation, outside the folders watched by the applications. Do not sync the repository root. The private local HTML guide contains the full GUI walkthrough; it is intentionally excluded from this repository.

## Scope

This repository contains namespace provisioning, ACM namespace governance, and an execution-disabled SNO upgrade process preview in `acm/upgrade`. The upgrade policy remains in inform mode, and its ClusterGroupUpgrade has `enable: false`, `preCaching: false` and `backup: false`. No upgrade is started by a preview sync. The upgrade preview uses its own `bdo-upgrade-policies` namespace and `bdo-upgrade` AppProject. See `acm/upgrade/README.md` for the GUI application fields and release provenance.
