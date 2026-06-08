# apps/

A bootstrap **ApplicationSet** (git directory generator) watches
`apps/*/overlays/*`. Every per-environment overlay becomes one Argo CD
`Application`:

| Directory                   | Application   | Namespace |
| --------------------------- | ------------- | --------- |
| `apps/nginx/overlays/dev`   | `nginx-dev`   | `dev`     |
| `apps/nginx/overlays/stage` | `nginx-stage` | `stage`   |
| `apps/nginx/overlays/prod`  | `nginx-prod`  | `prod`    |

So each app is a **Kustomize** project with a shared `base/` and one overlay per
environment (`dev` / `stage` / `prod` are simulated as namespaces in the single
cluster):

```
apps/
└── nginx/
    ├── base/                 # deployment + service, shared across envs
    │   ├── kustomization.yaml
    │   ├── deployment.yaml
    │   └── service.yaml
    └── overlays/
        ├── dev/kustomization.yaml      # namespace: dev,   images[].newTag
        ├── stage/kustomization.yaml    # namespace: stage, images[].newTag
        └── prod/kustomization.yaml     # namespace: prod,  images[].newTag
```

## Adding an app

Create `apps/<app>/base` and `apps/<app>/overlays/{dev,stage,prod}`, then commit.
The ApplicationSet generates `<app>-dev` / `<app>-stage` / `<app>-prod`
automatically.

## Promotion (Kargo, later)

Each overlay pins the image via Kustomize `images[].newTag`. That's the field
[Kargo](https://docs.kargo.io/) updates when promoting Freight through the
`dev → stage → prod` stages: it commits a new tag to the next environment's
overlay and Argo CD syncs it. Keeping per-env config in distinct directories
(not branches) is what makes that promotion flow work.
