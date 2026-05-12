# 🚢 tetris-devops-config

**GitOps configuration repository** for [tetris-devops](https://github.com/wasimat404/tetris-devops) — the manifests that define what runs in the Kubernetes cluster.

This repo is watched by **ArgoCD**. Every commit here triggers a deploy.

---

## 🔗 Related repos

- [`tetris-devops`](https://github.com/wasimat404/tetris-devops) — application source, Dockerfile, CI pipeline
- This repo — Kubernetes manifests, environment configs

See the [main README](https://github.com/wasimat404/tetris-devops) and [architecture doc](https://github.com/wasimat404/tetris-devops/blob/main/ARCHITECTURE.md) for the full picture.

---

## 📁 Structure

```
tetris-devops-config/
└── envs/
    └── prod/
        ├── namespace.yaml    # Creates 'tetris' namespace
        ├── deployment.yaml   # 3 replicas, hardened security context
        └── service.yaml      # type: LoadBalancer → AWS NLB
```

The folder-per-env pattern scales: a future `envs/staging/` or `envs/dev/` slots in cleanly with its own ArgoCD Application.

---

## 🤖 Image tags are auto-updated

The `image:` field in `deployment.yaml` is pinned to a specific commit SHA, e.g.:

```yaml
image: zenew/tetris-devops:sha-a1b2c3d4e5f6...
```

CI in the app repo **automatically commits to this repo** with the new SHA after every successful image build. You will see bot commits from `github-actions[bot]`. That is by design.

```
bump: tetris image to sha-a1b2c3d4
```

---

## 🚢 ArgoCD Application

The `Application` watching this repo:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: tetris
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/wasimat404/tetris-devops-config
    targetRevision: main
    path: envs/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: tetris
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## ⚠️ Manual changes

You can `git push` directly to update production:

- Want to scale to 5 replicas? Edit `deployment.yaml`, push. ArgoCD detects within 3 min.
- Want to roll back? `git revert <commit>`, push. ArgoCD picks it up.

**Do not** `kubectl edit` or `kubectl apply` against the cluster. ArgoCD's `selfHeal` will revert it.

---

