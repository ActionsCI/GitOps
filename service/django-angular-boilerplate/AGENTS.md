# AGENTS.md — service/django-angular-boilerplate

> Extends [root AGENTS.md](/AGENTS.md). Root rules always take precedence.

## Service Overview

`django-angular-boilerplate` is a Django + Angular web application deployed to
EKS. Its Helm chart lives in the
[django-angular-boilerplate](https://github.com/ActionsCI/django-angular-boilerplate)
repo under `helm/`. The ArgoCD Application that references this chart and points
at the values files below is defined in the k8s repo.

## Environments

| Env | Path | Notes |
|-----|------|-------|
| `dev` | `dev/values.yaml` | Continuous deploy on every merge to `main` |
| `staging` | `staging/values.yaml` | Promoted manually or by release workflow |
| `sandbox` | `sandbox/values.yaml` | PR smoke target; also used for ad-hoc testing |
| `production` | `production/values.yaml` | Requires sign-off before deploy workflow is triggered |

## values.yaml Structure

All four `values.yaml` files follow the same shape. The deploy workflow manages
only `image.tag`. Any other keys present are intentional env-specific overrides
owned by the django-angular-boilerplate team.

Minimum required content:

```yaml
image:
  tag: "x.y.z"
```

The chart's defaults (replica counts, resource limits, probes, ports) come from
`helm/values.yaml` in the application repo — do not duplicate them here unless
you are intentionally overriding for a specific environment.

## Local Golden Rules

1. **Do not add `image.registry` or `image.name` here.**  
   Both are set in the ArgoCD Application spec in the k8s repo. Duplicating
   them here creates drift risk.

2. **`production/values.yaml` requires a PR with at least one approval.**  
   Never merge changes to production directly, even for "trivial" edits like
   formatting. The deploy workflow enforces this via branch protection; do not
   bypass it.

3. **Do not add environment-specific secrets or DSNs.**  
   Database URLs, API keys, and third-party credentials are injected via
   External Secrets from AWS Secrets Manager. If a value looks sensitive, it
   does not belong in this file.
