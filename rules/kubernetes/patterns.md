# Kubernetes Patterns

## GitOps Deployment Flow

```
Developer → PR to gitops repo → Review → Merge
                                           ↓
                                    ArgoCD/Flux detects change
                                           ↓
                                    Sync to cluster (pull-based)
                                           ↓
                                    Health checks pass → done
                                    Health checks fail → auto-rollback
```

## Environment Promotion

```bash
# Promote dev → prod via Kustomize overlay changes
# 1. Test in dev overlay
# 2. Update prod overlay image tag (manually or via image updater)
# 3. PR → review → merge → ArgoCD syncs prod
```

ArgoCD Image Updater for automated tag promotion:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  annotations:
    argocd-image-updater.argoproj.io/image-list: app=registry.example.com/my-app
    argocd-image-updater.argoproj.io/app.update-strategy: semver
    argocd-image-updater.argoproj.io/write-back-method: git
```

## Health Checks (ALWAYS configure)

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /readyz
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5

startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

## Secrets via External Secrets Operator

Never store secrets in git. Use ExternalSecret to pull from AWS Secrets Manager/SSM:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: app-secrets
  data:
    - secretKey: DATABASE_URL
      remoteRef:
        key: /prod/database-url
    - secretKey: API_KEY
      remoteRef:
        key: /prod/api-key
```

## ConfigMap for Non-Sensitive Config

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  APP_ENV: "production"
```

## Rolling Deployments with PDB

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0

---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: my-app
```

## HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

## Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-netpol
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes: ["Ingress", "Egress"]
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api-gateway
      ports:
        - port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: database
      ports:
        - port: 5432
```

## Kustomize Components Pattern

For reusable cross-cutting concerns (monitoring sidecar, istio injection, etc.):

```yaml
# components/monitoring-sidecar/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component
patches:
  - target:
      kind: Deployment
    patch: |
      - op: add
        path: /spec/template/spec/containers/-
        value:
          name: metrics-exporter
          image: prom/node-exporter:latest
          ports:
            - containerPort: 9100

# overlays/prod/kustomization.yaml
components:
  - ../../components/monitoring-sidecar
```
