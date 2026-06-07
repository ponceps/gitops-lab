# apps/

A bootstrap **ApplicationSet** (git directory generator) watches this directory.
Every **subdirectory** of `apps/` becomes one Argo CD `Application` automatically:

- the Application is named after the directory (`apps/podinfo` → app `podinfo`),
- its source path is that directory,
- it deploys into a namespace of the same name (auto-created).

So you don't write `Application` manifests here — you drop a **directory of
workload manifests** (plain YAML, a Helm chart, or a kustomization) and commit
it. Argo CD generates and syncs the Application for you.

> Argo CD only treats subdirectories as apps, so this `README.md` is ignored.

## Example

```
apps/
└── podinfo/
    └── deployment.yaml      # plain manifests, Chart.yaml, or kustomization.yaml
```

`apps/podinfo/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: podinfo
spec:
  replicas: 1
  selector:
    matchLabels: { app: podinfo }
  template:
    metadata:
      labels: { app: podinfo }
    spec:
      containers:
        - name: podinfo
          image: ghcr.io/stefanprodan/podinfo:6.7.0
          ports:
            - containerPort: 9898
```

Commit that and Argo CD creates a `podinfo` Application in the `podinfo`
namespace. The generator/template is defined in
[`values/bootstrap.yaml`](../values/bootstrap.yaml).
