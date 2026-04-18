# AGENTS.md — service/pr-smoke

> Extends [root AGENTS.md](/AGENTS.md). Root rules always take precedence.

## What This Is

`pr-smoke` is a **test fixture**, not a real service. It exists solely so the
PR smoke job in
[ActionsCI/reusable-workflows](https://github.com/ActionsCI/reusable-workflows)
can run the `service-deployment-gitops.yaml` workflow end-to-end and verify
that the workflow commits a real change (i.e. the target branch is ahead of
`main` after the run).

There is no application, no Docker image, and no ArgoCD Application backed by
this path. The `image.tag` value in `sandbox/values.yaml` is never actually
deployed anywhere.

## Rules

1. **Do not delete this directory or any file within it.**  
   Doing so breaks the PR smoke job in ActionsCI/reusable-workflows and
   causes the reusable workflow to emit a misleading "Skipped" warning.

2. **Only `sandbox` exists here — do not add other environments.**  
   The smoke test only targets `sandbox`. Adding `dev`, `staging`, or
   `production` entries would create orphaned ArgoCD Application targets.

3. **Do not add real service configuration.**  
   No resource limits, replica counts, secrets, or any values beyond the
   minimum `image.tag` seed. This is a throwaway fixture.
