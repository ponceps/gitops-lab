# gitops-lab

A local lab for experimenting with GitOps workflows on a [kind](https://kind.sigs.k8s.io/)
(Kubernetes-in-Docker) cluster.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) (or a compatible container runtime)
- [mise](https://mise.jdx.dev/) to manage the pinned tool versions

Tool versions are pinned in [`mise.toml`](./mise.toml):

| Tool    | Version |
| ------- | ------- |
| kind    | 0.32.0  |
| kubectl | 1.36.1  |

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

### Tear down

```bash
kind delete cluster --name gitops-lab
```

## Configuration

- [`kind-config.yaml`](./kind-config.yaml) — cluster topology and pinned node image.
  Uncomment the `worker` node to run a multi-node cluster.
- [`create-cluster.sh`](./create-cluster.sh) — wrapper that creates the cluster from
  the config above.
