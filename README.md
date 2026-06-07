# gitops-lab

A local lab for experimenting with GitOps workflows on a [kind](https://kind.sigs.k8s.io/)
(Kubernetes-in-Docker) cluster.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) (or a compatible container runtime)
- [mise](https://mise.jdx.dev/) to manage the pinned tool versions

Tool versions are pinned in [`mise.toml`](./mise.toml):

| Tool     | Version | Purpose                            |
| -------- | ------- | ---------------------------------- |
| kind     | 0.32.0  | Local Kubernetes cluster           |
| kubectl  | 1.36.1  | Kubernetes CLI                     |
| helm     | 3.16.3  | Chart templating / releases        |
| helmfile | 0.169.1 | Declarative Argo CD install        |
| kubectx  | 0.11.0  | Switch between kube contexts       |
| kubens   | 0.11.0  | Switch between namespaces          |

Install them with:

```bash
mise install
```

## Usage

### Create the cluster

```bash
./create-cluster.sh
```

This spins up a single-node (control-plane) kind cluster named `gitops-lab`,
pinned to Kubernetes **v1.36.1** via the node image in
[`kind-config.yaml`](./kind-config.yaml). The script is idempotent — if a cluster
with that name already exists, it exits without recreating it.

Override the cluster name with the `CLUSTER_NAME` environment variable:

```bash
CLUSTER_NAME=my-lab ./create-cluster.sh
```

### Verify

```bash
kubectl cluster-info --context kind-gitops-lab
kubectl get nodes
```

### Install Argo CD (IaC)

Argo CD is installed declaratively with [Helmfile](https://helmfile.readthedocs.io/).
The first install needs the helm-diff plugin (or use `helmfile sync` instead):

```bash
helm plugin install https://github.com/databus23/helm-diff   # once
helmfile apply
```

This installs the `argo-cd` chart (pinned to chart 9.5.19 / Argo CD v3.4.3) into
the `argocd` namespace, then the `argocd-apps` chart, which creates a bootstrap
**ApplicationSet** (the officially recommended alternative to app-of-apps). Its
git directory generator emits one Argo CD `Application` per subdirectory of
[`apps/`](./apps), so anything committed there is reconciled automatically.

> The ApplicationSet pulls from `https://github.com/ponceps/gitops-lab.git`.
> Since the repo is private, register repo credentials in Argo CD (or make the
> repo public) so it can clone. See [`values/bootstrap.yaml`](./values/bootstrap.yaml).

#### Access the UI

1. Port-forward the Argo CD server:

   ```bash
   kubectl -n argocd port-forward svc/argocd-server 8080:80   # then open http://localhost:8080
   ```

2. Get the initial admin password:

   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret \
     -o jsonpath='{.data.password}' | base64 -d; echo
   ```

3. Log in as user `admin` with that password.

> The `argocd-initial-admin-secret` is for bootstrap only. After first login,
> change the admin password and delete the secret, as the
> [getting-started guide](https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)
> recommends.

To add a workload, drop a directory of manifests under `apps/` and commit it —
the ApplicationSet generates the Application for you. See
[`apps/README.md`](./apps/README.md).

### Tear down

```bash
helmfile destroy             # remove Argo CD + bootstrap
kind delete cluster --name gitops-lab
```

## Configuration

- [`kind-config.yaml`](./kind-config.yaml) — cluster topology and pinned node image.
  Uncomment the `worker` node to run a multi-node cluster.
- [`create-cluster.sh`](./create-cluster.sh) — wrapper that creates the cluster from
  the config above.
- [`helmfile.yaml`](./helmfile.yaml) — declarative Argo CD install + bootstrap releases
  (`argo-cd` and `argocd-apps`).
- [`values/`](./values) — Helm values for the `argo-cd` chart and the bootstrap
  ApplicationSet (`argocd-apps`).
- [`apps/`](./apps) — GitOps target dir; child `Application` manifests live here.
