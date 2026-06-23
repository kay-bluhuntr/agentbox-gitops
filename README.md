# agentbox-gitops

Deployment state for [agentbox](https://github.com/kay-bluhuntr/agentbox) — the
**desired version per environment**, separate from the application code.

## Why this repo exists

The application repo holds *what* the service is (source, Dockerfile, Helm chart).
This repo holds *which version runs where*. Keeping them apart means:

- **CI never commits to the app repo.** The app repo's `main` only changes by human
  pull request, so `git push` from a developer is never rejected by a bot commit.
- The deploy history is its own auditable timeline, independent of code changes.

## Layout

```
envs/
  dev/values.yaml    # Helm overrides for dev  — image.tag written by app-repo CI
  prod/values.yaml   # Helm overrides for prod — image.tag promoted via PR
```

## Flow

```
app-repo CI builds + pushes image ─► writes image.tag to envs/dev/values.yaml HERE
                                              │
                                    ArgoCD (values source = this repo) syncs dev
                                              │
                       promote: PR copies dev's tag into envs/prod/values.yaml
                                              │
                                    ArgoCD syncs prod on merge
```

ArgoCD reads the Helm **chart** from the app repo and these **values** from this
repo (multi-source Application). See `deploy/argocd/apps/` in the app repo.
