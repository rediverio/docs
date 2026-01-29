---
layout: default
title: Refactoring Guide - rediverio to exposure.io
parent: Guides
nav_order: 99
---

# Refactoring Guide: rediverio → exposure.io

This document outlines the migration plan from `rediverio` to `exposure.io` branding and infrastructure.

---

## Executive Summary

| Metric | Value |
|--------|-------|
| **Total references** | ~4,400 |
| **Files affected** | ~690 |
| **Go modules** | 3 (api, agent, sdk) |
| **Docker images** | 6+ |
| **Estimated effort** | 3-5 days |
| **Risk level** | Medium-High |

---

## 1. Scope of Changes

### 1.1 Go Module Paths

| Current | New |
|---------|-----|
| `github.com/rediverio/api` | `github.com/exposureio/api` |
| `github.com/rediverio/agent` | `github.com/exposureio/agent` |
| `github.com/rediverio/sdk` | `github.com/exposureio/sdk` |

**Files affected:**
- `api/go.mod`
- `agent/go.mod`
- `sdk/go.mod`
- All `*.go` files with imports (~662 files)

### 1.2 Docker Images

| Current | New |
|---------|-----|
| `rediverio/api` | `exposureio/api` |
| `rediverio/agent` | `exposureio/agent` |
| `rediverio/ui` | `exposureio/ui` |
| `rediverio/migrations` | `exposureio/migrations` |
| `rediverio/admin-ui` | `exposureio/admin-ui` |
| `rediverio/seed` | `exposureio/seed` |

**Files affected:**
- `docker-compose*.yml` (~10 files)
- Kubernetes manifests
- CI/CD workflows
- Helm values

### 1.3 Domains & URLs

| Current | New |
|---------|-----|
| `api.rediver.io` | `api.exposure.io` |
| `app.rediver.io` | `app.exposure.io` |
| `docs.rediver.io` | `docs.exposure.io` |
| `charts.rediver.io` | `charts.exposure.io` |
| `admin.rediver.io` | `admin.exposure.io` |
| `grpc.rediver.io` | `grpc.exposure.io` |

### 1.4 Database & Services

| Current | New |
|---------|-----|
| `DB_NAME=rediver` | `DB_NAME=exposure` |
| `APP_NAME=rediver` | `APP_NAME=exposure` |
| `KEYCLOAK_REALM=rediver` | `KEYCLOAK_REALM=exposure` |
| `SMTP_FROM=noreply@rediver.io` | `SMTP_FROM=noreply@exposure.io` |

### 1.5 Helm Charts

| Current | New |
|---------|-----|
| `rediver/platform-agent` | `exposure/platform-agent` |
| `helm repo add rediver https://charts.rediver.io` | `helm repo add exposure https://charts.exposure.io` |

---

## 2. Migration Phases

### Phase 1: Preparation (Day 1)

#### 1.1 Prerequisites
- [ ] Create GitHub organization: `github.com/exposureio`
- [ ] Create Docker Hub namespace: `exposureio/`
- [ ] Setup DNS for `*.exposure.io`
- [ ] Backup all databases

#### 1.2 Create Migration Script

```bash
#!/bin/bash
# migrate-names.sh

# Go module paths
find . -name "*.go" -type f -exec sed -i 's|github.com/rediverio/|github.com/exposureio/|g' {} +
find . -name "go.mod" -type f -exec sed -i 's|github.com/rediverio/|github.com/exposureio/|g' {} +
find . -name "go.sum" -type f -exec sed -i 's|github.com/rediverio/|github.com/exposureio/|g' {} +

# Docker images
find . -name "*.yml" -o -name "*.yaml" | xargs sed -i 's|rediverio/|exposureio/|g'
find . -name "Dockerfile*" | xargs sed -i 's|rediverio/|exposureio/|g'

# Domains
find . -type f \( -name "*.md" -o -name "*.yml" -o -name "*.yaml" -o -name "*.env*" \) \
    -exec sed -i 's|rediver\.io|exposure.io|g' {} +

# Database/App names in env files
find . -name ".env*" -type f -exec sed -i 's|DB_NAME=rediver|DB_NAME=exposure|g' {} +
find . -name ".env*" -type f -exec sed -i 's|APP_NAME=rediver|APP_NAME=exposure|g' {} +
```

### Phase 2: Code Changes (Day 2-3)

#### 2.1 Update Go Modules

```bash
# Step 1: Update go.mod files
cd api && sed -i 's|github.com/rediverio/|github.com/exposureio/|g' go.mod
cd ../agent && sed -i 's|github.com/rediverio/|github.com/exposureio/|g' go.mod
cd ../sdk && sed -i 's|github.com/rediverio/|github.com/exposureio/|g' go.mod

# Step 2: Update all imports
find . -name "*.go" -exec sed -i 's|github.com/rediverio/|github.com/exposureio/|g' {} +

# Step 3: Tidy modules
cd api && go mod tidy
cd ../agent && go mod tidy
cd ../sdk && go mod tidy

# Step 4: Verify build
go build ./...
```

#### 2.2 Update Docker References

Files to update:
- `setup/docker-compose.yml`
- `setup/docker-compose.prod.yml`
- `setup/docker-compose.staging.yml`
- All files in `setup/environments/`

#### 2.3 Update Helm Charts

```bash
cd charts/platform-agent
# Update Chart.yaml, values.yaml, templates/*
sed -i 's|rediver|exposure|g' Chart.yaml values.yaml
find templates -name "*.yaml" -exec sed -i 's|rediver|exposure|g' {} +
```

#### 2.4 Update CI/CD

Files in `.github/workflows/`:
- Update Docker image names
- Update Helm repo URLs
- Update deployment targets

### Phase 3: Infrastructure (Day 4)

#### 3.1 Docker Registry

```bash
# Build and push new images
docker build -t exposureio/api:latest ./api
docker build -t exposureio/agent:latest ./agent
docker build -t exposureio/ui:latest ./ui

docker push exposureio/api:latest
docker push exposureio/agent:latest
docker push exposureio/ui:latest
```

#### 3.2 Helm Repository

```bash
# Package and publish charts
helm package charts/platform-agent
# Upload to charts.exposure.io
```

#### 3.3 DNS Configuration

```
api.exposure.io      → Load Balancer IP
app.exposure.io      → CDN/Frontend
docs.exposure.io     → Documentation site
charts.exposure.io   → Helm chart repository
grpc.exposure.io     → gRPC endpoint
```

### Phase 4: Database Migration (Day 4-5)

#### Option A: Rename Database (Downtime Required)

```sql
-- Backup first!
pg_dump rediver > rediver_backup.sql

-- Rename database
ALTER DATABASE rediver RENAME TO exposure;

-- Update credentials if needed
ALTER USER rediver RENAME TO exposure;
```

#### Option B: Create New Database (Zero Downtime)

```sql
-- Create new database
CREATE DATABASE exposure;

-- Copy data
pg_dump rediver | psql exposure

-- Update application configuration
-- Switch traffic to new database
```

### Phase 5: Deployment & Verification (Day 5)

#### 5.1 Deployment Checklist

- [ ] Deploy new Docker images
- [ ] Update Kubernetes deployments
- [ ] Verify all services healthy
- [ ] Test API endpoints
- [ ] Test agent connectivity
- [ ] Test UI functionality

#### 5.2 Verification Commands

```bash
# Check API health
curl https://api.exposure.io/health

# Check agent connectivity
exposure-agent status

# Verify Docker images
docker pull exposureio/api:latest
docker pull exposureio/agent:latest
```

---

## 3. Breaking Changes

### 3.1 For SDK Users

Anyone using `github.com/rediverio/sdk` as a dependency must:

```bash
# Update go.mod
go get github.com/exposureio/sdk@latest

# Update imports
sed -i 's|github.com/rediverio/sdk|github.com/exposureio/sdk|g' *.go
```

### 3.2 For Agent Users

- Update agent configuration to point to new API URL
- Download new agent binary (if distribution URL changes)

```yaml
# Old config
api_url: https://api.rediver.io

# New config
api_url: https://api.exposure.io
```

### 3.3 For Helm Users

```bash
# Remove old repo
helm repo remove rediver

# Add new repo
helm repo add exposure https://charts.exposure.io
helm repo update

# Update releases
helm upgrade platform-agent exposure/platform-agent
```

---

## 4. Rollback Plan

### 4.1 Keep Old Infrastructure Running

During migration, maintain:
- Old Docker images available
- Old DNS records (with lower TTL)
- Database backup ready

### 4.2 Rollback Steps

```bash
# Revert DNS to old IPs
# Redeploy old Docker images
docker pull rediverio/api:latest
kubectl set image deployment/api api=rediverio/api:latest

# Restore database if needed
psql exposure < rediver_backup.sql
```

---

## 5. Search & Replace Patterns

### Primary Patterns

| Search | Replace |
|--------|---------|
| `github.com/rediverio/` | `github.com/exposureio/` |
| `rediverio/` (Docker) | `exposureio/` |
| `rediver\.io` | `exposure.io` |
| `api\.rediver\.io` | `api.exposure.io` |
| `DB_NAME=rediver` | `DB_NAME=exposure` |
| `APP_NAME=rediver` | `APP_NAME=exposure` |

### Files by Extension

```bash
# Go files
find . -name "*.go" -type f

# YAML/YML files
find . -name "*.yml" -o -name "*.yaml"

# Environment files
find . -name ".env*" -o -name "*.env.example"

# Markdown documentation
find . -name "*.md" -type f

# Shell scripts
find . -name "*.sh" -type f
```

---

## 6. Post-Migration Tasks

### 6.1 Documentation

- [ ] Update all README files
- [ ] Update API documentation
- [ ] Update deployment guides
- [ ] Update quick start guides

### 6.2 Communication

- [ ] Notify users of URL changes
- [ ] Update external integrations
- [ ] Update marketplace listings (if any)

### 6.3 Cleanup

- [ ] Archive old GitHub repos (after grace period)
- [ ] Remove old Docker images (after grace period)
- [ ] Remove old DNS records (after verification)

---

## 7. File Count by Category

| Category | Files | Occurrences |
|----------|-------|-------------|
| Go source files | 662 | ~2,500 |
| Documentation (*.md) | 516 | ~1,656 |
| YAML/YML configs | 50+ | ~1,096 |
| Environment files | 25+ | ~46 |
| Shell scripts | 10+ | ~19 |
| **Total** | **~690** | **~4,400** |

---

## 8. Timeline Summary

| Day | Tasks |
|-----|-------|
| Day 1 | Preparation, prerequisites, backup |
| Day 2 | Go module updates, import changes |
| Day 3 | Docker, Helm, CI/CD updates |
| Day 4 | Infrastructure, DNS, database |
| Day 5 | Deployment, verification, documentation |

---

## 9. Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Broken imports | Run `go build ./...` after each change |
| Docker pull failures | Pre-push all images before switching |
| DNS propagation | Use low TTL, maintain old records |
| Database issues | Full backup before any changes |
| User disruption | Announce migration window in advance |

---

## Related Documentation

- [Deployment Guide](./deployment.md)
- [Architecture Overview](../architecture/index.md)
- [SDK Quick Start](./sdk-quick-start.md)
