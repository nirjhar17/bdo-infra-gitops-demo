# BDO SNO upgrade process preview — execution disabled

This is a non-executing preview of Git → Argo CD → ACM → TALM for the existing self-managed `globe-sno-lab` target. It does not upgrade the ROSA HCP hub.

The previously completed Globe upgrade reached 4.20.42. This policy describes 4.20.43, so a fresh comparison should report NonCompliant while the cluster remains on 4.20.42. NonCompliant describes a difference from the requested target, not an unhealthy cluster.

## Files

- `policy-upgrade-4.20.43.yaml`: both the outer Policy and embedded ConfigurationPolicy use `remediationAction: inform`. The definition checks the requested release and a completed update history entry. It cannot request an upgrade while in inform mode.
- `placement.yaml`: selects only `globe-sno-lab`, using BDO-specific Placement and PlacementBinding objects.
- `clustergroupupgrade.yaml`: sets `spec.enable: false`, `preCaching: false`, and `backup: false`. TALM must not start remediation. The selected policy is listed in `managedPolicies`; `maxConcurrency: 1` illustrates one cluster at a time.
- `kustomization.yaml`: groups the policy, placement, binding and disabled rollout resource for one manual Argo CD sync.

All hub objects are isolated in `bdo-upgrade-policies`. No existing Globe policy or completed CGU is modified. Administrator preparation is in `bootstrap/upgrade-permissions.yaml` and `bootstrap/upgrade-appproject.yaml`.

## Preview sequence

1. Show the current SNO version and the requested 4.20.43 target in Git.
2. Show both inform settings and the CGU's disabled enable flag.
3. Create an Argo CD Application named `bdo-sno-upgrade`, project `bdo-upgrade`, revision `main`, path `acm/upgrade`, destination cluster `https://kubernetes.default.svc`, namespace `bdo-upgrade-policies`, with manual sync and pruning disabled.
4. Inspect the diff and sync. Inspect the SNO-specific policy result in ACM and the version mismatch details.
5. Show the disabled CGU and unchanged cluster version. Do not enable it during this demo or change the source policy to enforce.

In a future separately approved rollout, enabling a valid CGU lets TALM coordinate remediation of its listed inform policies. Actual execution still depends on cluster readiness and OpenShift update preconditions. Multiple clusters and a concurrency limit illustrate fleet rollout; this lab targets one SNO and does not demonstrate fleet scale.

## Release provenance and validation

The linux/amd64 digest is from [Red Hat's published 4.20.43 release metadata](https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/4.20.43/release.txt). A published digest alone does not establish that the live cluster currently offers a recommended update path. Check the target's live available updates and health before any future execution. No 4.19-to-4.20 acknowledgement is added for this same-minor patch preview.

[TALM documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/edge_computing/cnf-talm-for-cluster-updates)
