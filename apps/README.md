# apps/

The root **app-of-apps** Application watches this directory (recursively) and
applies every Kubernetes manifest it finds. Argo CD only parses `*.yaml`,
`*.yml`, and `*.json`, so this README is ignored.

To add a workload to the cluster, drop an Argo CD `Application` here and commit
it — Argo CD picks it up automatically. Example `apps/podinfo.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: podinfo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://stefanprodan.github.io/podinfo
    chart: podinfo
    targetRevision: 6.7.0
  destination:
    server: https://kubernetes.default.svc
    namespace: podinfo
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```
