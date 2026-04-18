# GitOps

This repository holds the Helm umbrella `Chart.yaml` manifests that the
[service-deployment-gitops](https://github.com/actionsci/reusable-workflows/blob/main/.github/workflows/service-deployment-gitops.yaml)
reusable workflow reads and writes on every deploy. Each file lives at
`service/<service>/<env>/Chart.yaml` and contains a `dependencies` entry with
`alias: service`; the workflow uses a `yq` selector on that alias to update the
chart version in-place, then commits and pushes the result back to `main`.

To onboard a new service, create the appropriate `service/<service>/<env>/Chart.yaml`
files following the same umbrella-chart convention used by `django-angular-boilerplate`.
No ArgoCD manifests, Kustomize overlays, or vendored subcharts belong here — the
deploy controller and OCI registry handle those concerns separately.

> **Note:** `service/pr-smoke/` is a test fixture used by the PR-smoke job in
> ActionsCI/reusable-workflows to verify the workflow's end-to-end write path.
> Do not delete it.
