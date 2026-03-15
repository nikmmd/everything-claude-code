---
name: deployment-patterns
description: Deployment strategies for ECS Fargate, Lambda, blue-green, rollback patterns, and CI/CD pipelines. Auto-triggers when working with deploy scripts, CI/CD config, or infrastructure changes.
compatibility: Designed for Claude Code
---

# Deployment Patterns

Auto-activates when working with deployment scripts, CI/CD configuration, or infrastructure changes.

## ECS Fargate Deployment

```bash
# SST Ion (preferred for this stack)
bunx sst deploy --stage dev
bunx sst deploy --stage prod
```

**Rolling update strategy:**
- `minimumHealthyPercent: 100` — never drop below current capacity
- `maximumPercent: 200` — spin up new before killing old
- Health check grace period > container startup time

```hcl
deployment_configuration {
  minimum_healthy_percent = 100
  maximum_percent         = 200
}

health_check_grace_period_seconds = 60
```

## Lambda Deployment

- Use aliases (`prod`, `dev`) pointing to specific versions
- Provisioned concurrency for latency-sensitive functions
- Bundle size matters — tree-shake, exclude dev deps

## Blue-Green / Canary

**Blue-Green (ALB):**
1. Deploy new version to "green" target group
2. Health checks pass on green
3. Switch ALB listener to green
4. Keep blue running for quick rollback
5. Decommission blue after confidence period

**Canary (weighted routing):**
1. Deploy new version alongside current
2. Route 5% traffic to new version
3. Monitor error rates and latency
4. Gradually increase: 5% → 25% → 50% → 100%
5. Rollback instantly by shifting weight back

## Rollback

**Immediate rollback checklist:**
1. Revert to previous task definition / image tag
2. Force new deployment: `aws ecs update-service --force-new-deployment`
3. Verify health checks pass
4. Investigate root cause from logs/metrics

**Database rollback considerations:**
- Forward-only migrations preferred (add column, not rename)
- If rollback needed, deploy a new forward migration that undoes changes
- Never roll back migrations that dropped data

## CI/CD Pipeline

```yaml
# Typical flow
build:
  - Install deps
  - Type check
  - Lint
  - Unit tests
  - Build container image
  - Push to ECR

deploy-dev:
  - Deploy to dev (auto on main merge)
  - Run integration tests
  - Run E2E tests against dev

deploy-prod:
  - Manual approval gate (or auto for trusted pipelines)
  - Deploy to prod
  - Smoke tests
  - Monitor error rate for 15 minutes
```

## Environment Promotion

```
feature branch → PR → merge to main
                         ↓
                    auto-deploy dev
                         ↓
                    integration tests pass
                         ↓
                    manual approval (or auto)
                         ↓
                    deploy prod
```

## Health Checks

Every deployable service needs:
- `/healthz` — liveness (is the process alive?)
- `/readyz` — readiness (can it serve traffic? DB connected?)

```typescript
// Minimal health check endpoint
app.get('/healthz', (c) => c.json({ status: 'ok' }))

app.get('/readyz', async (c) => {
  try {
    await db.execute(sql`SELECT 1`)
    return c.json({ status: 'ready' })
  } catch {
    return c.json({ status: 'not ready' }, 503)
  }
})
```

## Pre-Deploy Checklist

- [ ] All tests passing (unit, integration, E2E)
- [ ] No security vulnerabilities (`npm audit`, `trivy`)
- [ ] Database migrations applied and tested
- [ ] Environment variables configured in target environment
- [ ] Health check endpoints responding
- [ ] Rollback plan documented
- [ ] Monitoring/alerting in place
