# kargo/

[Kargo](https://docs.kargo.io/) promotion pipeline for the `nginx` app:
**dev → stage → prod**.

```
Warehouse(nginx image) ──Freight──▶ dev ──▶ stage ──▶ prod
```

On promotion into a stage, the shared `promote` PromotionTask rewrites
`apps/nginx/overlays/<stage>/kustomization.yaml` `images[].newTag` to the
Freight's tag, commits to `main`, and tells Argo CD to sync `nginx-<stage>`.

| File | Resource |
| --- | --- |
| `project.yaml` | Kargo `Project` (owns the `nginx` namespace) |
| `warehouse.yaml` | `Warehouse` watching `public.ecr.aws/nginx/nginx` |
| `promotion-task.yaml` | `PromotionTask promote` (git-clone → set-image → commit → push → argocd-update) |
| `stages.yaml` | `Stage`s dev/stage/prod |

## Not in Git: the git credential

Kargo needs write access to push promotion commits. The credential is a
`Secret` (label `kargo.akuity.io/cred-type: git`) created **out of band** so the
token never lands in this repo:

```bash
kubectl create secret generic nginx-repo \
  --namespace nginx \
  --from-literal=repoURL=https://github.com/ponceps/gitops-lab.git \
  --from-literal=username=<github-username> \
  --from-literal=password=<github-PAT-with-repo-write>
kubectl label secret nginx-repo -n nginx kargo.akuity.io/cred-type=git
```

## Install (control plane)

Kargo + cert-manager are installed via Helm (not yet in `helmfile.yaml`); see the
repo README / project notes.
