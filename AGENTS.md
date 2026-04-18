# AGENTS.md — ActionsCI/gitops

## What This Repo Is

This is the GitOps state store for ActionsCI services running on EKS. It holds
per-environment Helm values overrides that ArgoCD watches and applies to the
cluster. The only thing that changes here at deploy time is `image.tag` — the
Helm chart reference itself lives in the k8s repo alongside the ArgoCD
Application manifest.

## Path Convention

```
service/<service-name>/<env>/values.yaml
```

Every file is a partial Helm values override. ArgoCD merges it on top of the
chart's own `values.yaml`. At minimum each file must contain:

```yaml
image:
  tag: "x.y.z"
```

Supported environments: `dev`, `staging`, `sandbox`, `production`.

## How Deploys Work

The [service-deployment-gitops](https://github.com/actionsci/reusable-workflows/blob/main/.github/workflows/service-deployment-gitops.yaml)
reusable workflow:

1. Checks out this repo.
2. Uses `yq` to update `image.tag` in the target `values.yaml`.
3. Commits and pushes to `main`.
4. ArgoCD detects the commit and syncs the cluster.

The workflow is the **only** automated writer to this repo. Do not create
tooling that writes to `main` by any other path.

## Repo Structure

```
gitops/
├── AGENTS.md
├── README.md
├── .gitignore
└── service/
    ├── <service-name>/
    │   ├── AGENTS.md
    │   ├── dev/values.yaml
    │   ├── staging/values.yaml
    │   ├── sandbox/values.yaml
    │   └── production/values.yaml
    └── pr-smoke/
        ├── AGENTS.md
        └── sandbox/values.yaml   ← test fixture only, sandbox env only
```

Each service directory has its own AGENTS.md extending (never overriding) these
root rules.

## Golden Rules

1. **`values.yaml` is the only file type inside `service/*/`.**  
   No `Chart.yaml`, no `Chart.lock`, no ArgoCD Application manifests, no
   Kustomize overlays, no secrets. The chart reference lives in the k8s repo.

2. **Never vendor subcharts here.**  
   Dependencies resolve from the OCI registry at sync time. No `charts/`
   directories.

3. **Never commit secrets or credentials.**  
   `values.yaml` files must not contain image pull secrets, database URLs,
   API keys, or any sensitive value. Those are injected by the ArgoCD app via
   External Secrets or SSM references defined in the k8s repo.

4. **`service/pr-smoke/` is a permanent test fixture — do not delete it.**  
   It exists so the reusable-workflow PR smoke job can verify the end-to-end
   write path. Deleting it breaks CI in ActionsCI/reusable-workflows.

5. **Do not push directly to `main`.**  
   All human changes go through a PR. The deploy workflow is the only exception
   (it has a dedicated commit identity and bypasses branch protection via a
   deploy key).

6. **Only add `sandbox` and `production` initially for new services.**  
   `dev` and `staging` entries are added when those environments are
   provisioned in the k8s repo. Seeding them early creates orphaned ArgoCD
   Applications.

7. **`image.tag` is the only field the deploy workflow manages.**  
   Do not structure `values.yaml` files so that `image.tag` is nested under
   an alias or renamed key — the workflow's `yq` path is `.image.tag`.

8. **Preserve all other keys when editing `values.yaml`.**  
   Reviewers and agents must not remove non-`image.tag` values that a service
   team has intentionally placed (replica counts, resource overrides, feature
   flags). Those are env-specific tuning, not clutter.

## Onboarding a New Service

1. Create `service/<service-name>/AGENTS.md` describing the service, its
   chart, and any env-specific notes.
2. Create `service/<service-name>/<env>/values.yaml` for each provisioned
   environment, seeded with `image.tag: "0.0.1"`.
3. Open a PR — do not merge directly to `main`.
4. The k8s repo team wires up the ArgoCD Application pointing at this path.

## Common Mistakes Agents Make

1. Adding `Chart.yaml` files (the chart lives in the k8s repo, not here).
2. Adding ArgoCD Application YAML, Kustomization files, or Helm release objects.
3. Seeding all four environments for a service that only has `sandbox` and
   `production` provisioned.
4. Deleting `service/pr-smoke/` because it looks like an unused placeholder.
5. Hardcoding registry URLs or image names that should come from the ArgoCD app
   spec in the k8s repo.
6. Nesting `image.tag` under a different key, breaking the workflow's `yq`
   selector.
7. Committing a `charts/` directory after running `helm dependency update`
   locally.

## Agent Escalation Points

Stop and flag to a human when:

- A task requires writing anything other than `values.yaml` files under `service/`.
- A task requires pushing directly to `main`.
- A task requires deleting `service/pr-smoke/`.
- The scope of changes would affect more than one service at once (batch
  migrations need explicit approval).
- Existing `values.yaml` files contain keys beyond `image.tag` that aren't
  documented in the service's own AGENTS.md.

## Maintenance

Root golden rules are owned by the Platform Engineering team. Propose changes
via PR with the `platform-eng-review` label.

Service-level AGENTS.md files are owned by each service team and may only add
stricter constraints, never relax root rules.
