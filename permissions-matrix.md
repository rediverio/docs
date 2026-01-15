# Permission Matrix

Ma trận phân quyền đầy đủ cho Rediver CTEM Platform.

---

## Role Hierarchy

| Role | Priority | Description |
|------|:--------:|-------------|
| **Owner** | 4 | Toàn quyền quản lý tenant, bao gồm billing và xóa tenant |
| **Admin** | 3 | Quản lý members, settings, và tất cả resources |
| **Member** | 2 | Tạo và chỉnh sửa resources, không quản lý team |
| **Viewer** | 1 | Chỉ đọc dữ liệu |

---

## Role-Based Permissions

### Team Management

| Permission | Owner | Admin | Member | Viewer |
|-----------|:-----:|:-----:|:------:|:------:|
| `team:read` | ✅ | ✅ | ✅ | ✅ |
| `team:update` | ✅ | ✅ | ❌ | ❌ |
| `team:delete` | ✅ | ❌ | ❌ | ❌ |
| `members:read` | ✅ | ✅ | ✅ | ✅ |
| `members:invite` | ✅ | ✅ | ❌ | ❌ |
| `members:manage` | ✅ | ✅ | ❌ | ❌ |
| `billing:read` | ✅ | ❌ | ❌ | ❌ |
| `billing:manage` | ✅ | ❌ | ❌ | ❌ |

### Asset Management (CTEM Discovery)

| Permission | Owner | Admin | Member | Viewer |
|-----------|:-----:|:-----:|:------:|:------:|
| `assets:read` | ✅ | ✅ | ✅ | ✅ |
| `assets:write` | ✅ | ✅ | ✅ | ❌ |
| `assets:delete` | ✅ | ✅ | ❌ | ❌ |
| `repositories:read` | ✅ | ✅ | ✅ | ✅ |
| `repositories:write` | ✅ | ✅ | ✅ | ❌ |
| `repositories:delete` | ✅ | ✅ | ❌ | ❌ |
| `branches:read` | ✅ | ✅ | ✅ | ✅ |
| `branches:write` | ✅ | ✅ | ✅ | ❌ |
| `branches:delete` | ✅ | ✅ | ❌ | ❌ |
| `components:read` | ✅ | ✅ | ✅ | ✅ |
| `components:write` | ✅ | ✅ | ✅ | ❌ |
| `components:delete` | ✅ | ✅ | ❌ | ❌ |

### Vulnerability & Findings (CTEM Prioritization)

| Permission | Owner | Admin | Member | Viewer |
|-----------|:-----:|:-----:|:------:|:------:|
| `vulnerabilities:read` | ✅ | ✅ | ✅ | ✅ |
| `vulnerabilities:write` | ✅ | ✅ | ❌ | ❌ |
| `vulnerabilities:delete` | ✅ | ✅ | ❌ | ❌ |
| `findings:read` | ✅ | ✅ | ✅ | ✅ |
| `findings:write` | ✅ | ✅ | ✅ | ❌ |
| `findings:delete` | ✅ | ✅ | ❌ | ❌ |

### Scans & Validation (CTEM Validation)

| Permission | Owner | Admin | Member | Viewer |
|-----------|:-----:|:-----:|:------:|:------:|
| `scans:read` | ✅ | ✅ | ✅ | ✅ |
| `scans:write` | ✅ | ✅ | ✅ | ❌ |
| `pentest:read` | ✅ | ✅ | ✅ | ✅ |
| `pentest:write` | ✅ | ✅ | ✅ | ❌ |
| `credentials:read` | ✅ | ✅ | ✅ | ✅ |
| `credentials:write` | ✅ | ✅ | ✅ | ❌ |

### Remediation & Mobilization (CTEM Mobilization)

| Permission | Owner | Admin | Member | Viewer |
|-----------|:-----:|:-----:|:------:|:------:|
| `remediation:read` | ✅ | ✅ | ✅ | ✅ |
| `remediation:write` | ✅ | ✅ | ✅ | ❌ |
| `workflows:read` | ✅ | ✅ | ✅ | ✅ |
| `workflows:write` | ✅ | ✅ | ✅ | ❌ |

### Dashboard & Reports

| Permission | Owner | Admin | Member | Viewer |
|-----------|:-----:|:-----:|:------:|:------:|
| `dashboard:read` | ✅ | ✅ | ✅ | ✅ |
| `reports:read` | ✅ | ✅ | ✅ | ✅ |
| `reports:write` | ✅ | ✅ | ✅ | ❌ |
| `audit:read` | ✅ | ✅ | ✅ | ✅ |

### SLA & Integrations

| Permission | Owner | Admin | Member | Viewer |
|-----------|:-----:|:-----:|:------:|:------:|
| `sla:read` | ✅ | ✅ | ✅ | ✅ |
| `sla:write` | ✅ | ✅ | ❌ | ❌ |
| `sla:delete` | ✅ | ✅ | ❌ | ❌ |
| `scm-connections:read` | ✅ | ✅ | ✅ | ✅ |
| `scm-connections:write` | ✅ | ✅ | ❌ | ❌ |
| `scm-connections:delete` | ✅ | ✅ | ❌ | ❌ |
| `integrations:read` | ✅ | ✅ | ✅ | ✅ |
| `integrations:manage` | ✅ | ✅ | ❌ | ❌ |

---

## Role Assignment Rules

| Action | Owner | Admin | Member | Viewer |
|--------|:-----:|:-----:|:------:|:------:|
| Assign Owner role | ❌ | ❌ | ❌ | ❌ |
| Assign Admin role | ✅ | ❌ | ❌ | ❌ |
| Assign Member role | ✅ | ✅ | ❌ | ❌ |
| Assign Viewer role | ✅ | ✅ | ❌ | ❌ |

> **Note:** Owner role cannot be assigned via invitation. Transfer ownership requires special process.

---

## Permission Summary by Role

### Owner
- Full access to all resources
- Manage billing and subscription
- Delete tenant
- Transfer ownership
- Manage all team settings including security and API

### Admin
- Manage team members (invite, update role, remove)
- Create/edit/delete most resources
- Manage SLA policies and integrations
- Cannot delete tenant or manage billing

### Member
- Create and edit resources (assets, findings, scans)
- Run scans and validations
- Add comments and update statuses
- Cannot manage team or delete resources

### Viewer
- Read-only access to all data
- View dashboards and reports
- Cannot create, edit, or delete anything

---

## API Endpoint Permissions

| Endpoint | Method | Permission Required |
|----------|--------|---------------------|
| `/api/v1/assets` | GET | `assets:read` |
| `/api/v1/assets` | POST | `assets:write` |
| `/api/v1/assets/{id}` | PUT | `assets:write` |
| `/api/v1/assets/{id}` | DELETE | `assets:delete` |
| `/api/v1/findings` | GET | `findings:read` |
| `/api/v1/findings` | POST | `findings:write` |
| `/api/v1/findings/{id}` | DELETE | `findings:delete` |
| `/api/v1/vulnerabilities` | GET | `vulnerabilities:read` |
| `/api/v1/vulnerabilities` | POST | `vulnerabilities:write` |
| `/api/v1/dashboard/stats` | GET | `dashboard:read` |
| `/api/v1/audit-logs` | GET | `audit:read` |
| `/api/v1/sla-policies` | GET | `sla:read` |
| `/api/v1/sla-policies` | POST | `sla:write` |
| `/api/v1/components` | GET | `components:read` |
| `/api/v1/scm-connections` | GET | `scm-connections:read` |

> Full API reference: [api-reference.md](./api-reference.md)

---

## Implementation Notes

### Permission Check in Backend
```go
// Route with permission middleware
r.GET("/assets", h.List, middleware.Require(permission.AssetsRead))
r.POST("/assets", h.Create, middleware.Require(permission.AssetsWrite))
r.DELETE("/assets/{id}", h.Delete, middleware.Require(permission.AssetsDelete))
```

### Role Methods
```go
role.CanRead()     // All roles
role.CanWrite()    // Owner, Admin, Member
role.CanInvite()   // Owner, Admin
role.CanDelete()   // Owner only (for tenant)
role.Priority()    // 4, 3, 2, 1
```
