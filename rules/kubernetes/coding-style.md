# Kubernetes / Helm / Kustomize / ArgoCD Coding Style

## GitOps-First

All cluster state lives in git. No manual `kubectl apply` in production. ArgoCD syncs from git (pull-based).

## Cluster Repository Structure

Bootstrap with a base ApplicationSet, then per-app kustomization. Split into control-plane and apps, numbered for sync wave ordering:

```
clusters/
├── bootstrap/                         # Base ApplicationSet(s)
│   └── applicationset.yaml            # Discovers and deploys everything below
│
├── control-plane/                     # Cluster-wide infra
│   ├── 100-namespaces/                # Namespace definitions
│   │   └── kustomization.yaml
│   ├── 101-crds/                      # CRD installs (separate from apps)
│   │   └── kustomization.yaml
│   ├── 200-cert-manager/
│   │   └── kustomization.yaml
│   ├── 201-external-secrets/
│   │   └── kustomization.yaml
│   ├── 202-ingress-nginx/
│   │   └── kustomization.yaml
│   └── 300-monitoring/
│       └── kustomization.yaml
│
└── apps/                              # Application workloads
    ├── 100-databases/
    │   └── kustomization.yaml
    ├── 200-backend-api/
    │   └── kustomization.yaml
    ├── 201-worker/
    │   └── kustomization.yaml
    └── 300-frontend/
        └── kustomization.yaml
```

**Numbered sync waves** (same convention as Terraform/Ansible):
| Range | Layer | Purpose |
|-------|-------|---------|
| 100-199 | Foundation | Namespaces, CRDs, networking, DNS |
| 200-299 | Platform services | Cert-manager, secrets operator, ingress, databases |
| 300-399 | Application | Backend, frontend, workers |

## Sync Policy

**Automated sync for application-layer (300+)** — these change frequently (image tags, config):

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=false    # Namespaces managed separately
```

**Manual or semi-automated sync for sensitive/static workloads** (databases, identity providers, CRD controllers) — these rarely change and mistakes are costly. Use automated sync sparingly for 100-200 level control-plane resources.

## CRDs and Namespaces — Special Handling

**CRDs:**
- Install separately from the application that uses them
- Use `ServerSideApply=true` (CRDs exceed annotation size limits with client-side)
- Never prune CRDs — set `Prune=false` or exclude from pruning
- If a chart bundles CRD management (e.g., via a Job), let it handle it

**Namespaces:**
- Manage separately from applications
- Never prune namespaces — losing a namespace deletes everything in it
- Create in the lowest sync wave (100)

```yaml
syncPolicy:
  syncOptions:
    - ServerSideApply=true       # Required for CRDs
    - PruneLast=true
    - RespectIgnoreDifferences=true
ignoreDifferences:
  - group: apiextensions.k8s.io
    kind: CustomResourceDefinition
    jsonPointers:
      - /status
```

## Helm Hooks → ArgoCD Hooks

Helm lifecycle hooks don't work well with GitOps. Convert to ArgoCD resource hooks:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
        - name: migrate
          image: app:latest
          command: ["npm", "run", "db:migrate"]
      restartPolicy: Never
```

| Helm Hook | ArgoCD Equivalent | Use Case |
|-----------|-------------------|----------|
| `pre-install` / `pre-upgrade` | `PreSync` | DB migrations, config validation |
| `post-install` / `post-upgrade` | `PostSync` | Cache warming, notifications |
| `pre-delete` | `SyncFail` | Cleanup on failed sync |

## Sync Wave Annotations

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "100"   # Matches directory number
```

## Naming

- Resources: `kebab-case` — `my-app-web`
- Labels: consistent across all resources
- Namespaces: per-application or per-team

## Required Labels

```yaml
metadata:
  labels:
    app.kubernetes.io/name: my-app
    app.kubernetes.io/component: web
    app.kubernetes.io/part-of: platform
    app.kubernetes.io/managed-by: argocd
```

## Resource Limits (ALWAYS set)

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

## Security Context (ALWAYS set)

```yaml
securityContext:
  runAsNonRoot: true
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
```

## Helm in GitOps

- Use `values.yaml` for defaults, `values-<env>.yaml` for overrides
- Template helpers in `_helpers.tpl`
- Disable Helm hooks in ArgoCD (`--no-hooks`) — use ArgoCD hooks instead
- Prefer rendering to Kustomize or using ArgoCD's native Helm source

## Tools

- **GitOps**: ArgoCD (primary)
- **Linting**: `helm lint`, `kubeval`, `kubeconform`
- **Security**: `kubesec scan`, `kube-score`, `trivy`
- **Diff**: `argocd app diff`, `helm diff upgrade`
- **Validation**: `kubectl --dry-run=server`
- **Secrets**: External Secrets Operator (never store secrets in git)
