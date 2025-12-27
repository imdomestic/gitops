# Home GitOps (Argo CD + App-of-Apps + Helm)

This repo is a **batteries-included** GitOps layout for a single cluster (example: `home`) that:

- Manages **Argo CD itself** (optional self-management app)
- Uses **App of Apps** (a single `root` Application that creates other Applications)
- Creates **namespaces** declaratively
- Installs **Agones** via Helm, configured for Shulker (`gameservers.namespaces` + allocator enabled)
- Installs **Shulker** via Helm (chart is in Shulker's Git repo)
- Applies Shulker CRDs for:
  - `ProxyFleet` (Velocity proxies)
  - `MinecraftServerFleet` lobby
  - `MinecraftServerFleet` PaperMC servers (example: `paper`)

## What you must edit

Search & replace these placeholders:

- `https://github.com/<YOU>/<REPO>.git`  -> your git repo URL
- `main`        -> your branch name (if not `main`)

Files to edit:
- `clusters/home/bootstrap/root-app.yaml`
- `clusters/home/apps/*.yaml` (repoURL fields)

## Bootstrap (first time)

### 0) Push this repo to GitHub/Gitea

Make it **public** first (simplest), or later add repo credentials in Argo CD.

### 1) Install Argo CD (one-time)

Create namespace and install the official manifest (standard install):
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

> `install.yaml` is the standard Argo CD installation manifest and includes cluster-scoped permissions suitable for deploying apps in the same cluster. (See Argo CD installation docs.)

### 2) Apply the root app (App-of-Apps)

```bash
kubectl apply -f clusters/home/bootstrap/root-app.yaml
```

### 3) Open Argo CD UI and sync

If you don't have an Ingress/LoadBalancer, do a port-forward:
```bash
kubectl -n argocd port-forward svc/argocd-server 8080:80
```

Log in, then **sync** the `root` app.

## Notes / gotchas

### LoadBalancer in homelab
Shulker docs note that **ProxyFleet creates a Service of type `LoadBalancer` by default**. If you don't have a cloud LB, install MetalLB, or change the ProxyFleet service type to `NodePort` / use an Ingress.

### Versions
- Agones chart version is pinned to `1.54.0` in this template (adjust if you want).
- Shulker is pinned to `v0.13.0` (adjust as desired).

## Repo layout

```
clusters/home/bootstrap/         # manual bootstrap entrypoints
clusters/home/apps/              # app-of-apps children (Applications)
infra/namespaces/                # Namespace objects
platform/argocd/                 # (optional) manage Argo CD itself
platform/agones/                 # Helm values for Agones
platform/shulker/                # Helm values for Shulker (optional)
apps/mc/                         # Shulker CRs: ProxyFleet + lobby + paper fleet
```
