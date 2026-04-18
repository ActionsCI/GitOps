# GitOps

This repository holds the per-environment `values.yaml` overrides that the
[service-deployment-gitops](https://github.com/actionsci/reusable-workflows/blob/main/.github/workflows/service-deployment-gitops.yaml)
reusable workflow writes on every deploy. Each file lives at
`service/<service>/<env>/values.yaml`; the workflow updates `image.tag` in-place
and commits the result back to `main`. ArgoCD picks up the change and syncs the
cluster — the Helm chart itself is referenced from the k8s repo, not here.

To onboard a new service, create `service/<service>/<env>/values.yaml` files seeded
with `image.tag: "0.0.1"` for each environment, following the same convention used
by `django-angular-boilerplate`. No ArgoCD Application manifests, Kustomize overlays,
or chart files belong here.

> **Note:** `service/pr-smoke/` is a test fixture used by the PR-smoke job in
> ActionsCI/reusable-workflows to verify the workflow's end-to-end write path.
> Do not delete it.
