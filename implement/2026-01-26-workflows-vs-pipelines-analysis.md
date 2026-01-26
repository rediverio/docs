# Workflows vs Pipelines - Analysis & Implementation Plan

**Created:** 2026-01-26
**Status:** PLANNING
**Last Updated:** 2026-01-26

---

## Executive Summary

Sau khi phân tích chi tiết codebase, kết luận: **Workflows và Pipelines là 2 hệ thống KHÁC NHAU** phục vụ mục đích riêng biệt. Cần giữ riêng và kết nối với nhau.

---

## 1. Phân tích chi tiết

### 1.1 Pipelines (Backend - Đã implement)

**Mục đích:** Scan execution orchestration - Điều phối việc chạy các scanner tools

**Location:**
- Backend: `api/internal/domain/pipeline/`, `api/internal/app/pipeline_service.go`
- Frontend: `ui/src/app/(dashboard)/(mobilization)/pipelines/` (mới tạo)

**Entities:**
```
PipelineTemplate
├── name, description, version
├── triggers: [manual, schedule, webhook, api, on_asset_discovery]
├── settings: max_parallel_steps, fail_fast, timeout
└── steps: PipelineStep[]

PipelineStep
├── step_key, name, order
├── tool: scanner tool name
├── capabilities: [scan, web, network, etc.]
├── config: tool-specific JSON config
├── depends_on: step dependencies
├── condition: always, never, expression, asset_type, step_result
├── retry: max_retries, retry_delay_seconds
└── ui_position: {x, y} for visual builder

PipelineRun → StepRun → Command → Agent execution
```

**Tích hợp:**
- Agent system (command queue, polling)
- Findings collection
- Scan configs (Scan.PipelineID)

### 1.2 Workflows (Frontend Mock - Chưa có backend)

**Mục đích:** General automation orchestration - Tự động hóa các security operations

**Location:**
- Frontend: `ui/src/app/(dashboard)/(mobilization)/workflows/page.tsx`
- Backend: ❌ Chưa có

**Node Types (từ mock data):**
```
Trigger (Green)
├── New Critical Finding
├── Schedule: Every Monday 2:00 AM
├── Finding Age > 48 hours
└── New Asset Discovered

Condition (Yellow)
├── IF/THEN logic
└── 2 output handles: yes/no

Action (Blue)
├── Assign to Team Lead
├── Full Asset Scan
├── Generate Report
├── Create Jira Ticket
└── Update Priority

Notification (Purple)
├── Send Slack Alert
├── Email to Security Team
└── Slack Notification
```

### 1.3 So sánh

| Aspect | Workflows | Pipelines |
|--------|-----------|-----------|
| **Layer** | Frontend only (mock) | Backend (fully implemented) |
| **Purpose** | General automation | Scan execution |
| **Actions** | External (Slack, Jira, assign) | Internal (scanner tools) |
| **Agent** | Không có | Tích hợp sâu |
| **Findings** | Trigger source | Output collection |
| **Database** | ❌ Chưa có | ✅ Có đầy đủ |
| **API** | ❌ Chưa có | ✅ Có đầy đủ |

---

## 2. Quyết định kiến trúc

### Giữ riêng biệt + Kết nối

```
┌─────────────────────────────────────────────────────────────┐
│                     WORKFLOWS (Automation)                   │
│  ┌─────────┐    ┌───────────┐    ┌──────────────────────┐   │
│  │ Trigger │───▶│ Condition │───▶│ Action               │   │
│  │         │    │           │    │ ├─ Assign User       │   │
│  │ Finding │    │ Severity? │    │ ├─ Send Notification │   │
│  │ Created │    │           │    │ ├─ Create Ticket     │   │
│  └─────────┘    └───────────┘    │ └─ RUN PIPELINE ────────────┐
│                                  └──────────────────────┘   │  │
└─────────────────────────────────────────────────────────────┘  │
                                                                  │
                    ┌─────────────────────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────────────────────┐
│                     PIPELINES (Scan Execution)               │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐   │
│  │ Step 1  │───▶│ Step 2  │───▶│ Step 3  │───▶│ Step N  │   │
│  │ Nmap    │    │ Nuclei  │    │ SSLScan │    │ Report  │   │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘   │
│                         │                                    │
│                         ▼                                    │
│                   [Agent Execution]                          │
│                         │                                    │
│                         ▼                                    │
│                   [Findings Output]                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Implementation Plan

### Phase 1: Reorganize Current Work ✅ DONE

- [x] Pipelines backend (UIPosition, parallel control)
- [x] Pipelines frontend page `/pipelines`
- [x] API types, hooks, endpoints
- [x] WorkflowBuilder component (reusable)

### Phase 2: Rename for Clarity 🔜 TODO

**Mục tiêu:** Phân biệt rõ ràng giữa Scan Pipelines và Automation Workflows

```
/workflows              → Giữ nguyên (Automation Workflows)
/pipelines              → Rename thành /scan-pipelines hoặc giữ nguyên
```

**Files to update:**
- [ ] Rename page title: "Scan Pipelines" thay vì "Pipelines"
- [ ] Update navigation menu
- [ ] Update docs

### Phase 3: Build Workflows Backend 🔜 TODO (Future)

**Mục tiêu:** Tạo backend cho Automation Workflows

```
api/internal/
├── domain/workflow/
│   ├── entity.go           # Workflow, Node, Edge, NodeType
│   ├── node_types.go       # Trigger, Condition, Action, Notification
│   ├── execution.go        # WorkflowRun, NodeRun
│   ├── errors.go
│   └── repository.go       # Interfaces
├── infra/
│   ├── postgres/
│   │   ├── workflow_repository.go
│   │   └── workflow_run_repository.go
│   └── http/
│       ├── handler/
│       │   ├── workflow_handler.go
│       │   └── workflow_execution_handler.go
│       └── routes/workflow.go
└── app/
    ├── workflow_service.go         # Core logic
    └── workflow_executor.go        # Execution engine
```

**Database tables:**
```sql
-- Workflow definitions
CREATE TABLE workflows (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE,
    updated_at TIMESTAMP WITH TIME ZONE
);

-- Workflow nodes
CREATE TABLE workflow_nodes (
    id UUID PRIMARY KEY,
    workflow_id UUID NOT NULL REFERENCES workflows(id),
    node_type VARCHAR(50) NOT NULL, -- trigger, condition, action, notification
    node_key VARCHAR(100) NOT NULL,
    name VARCHAR(255) NOT NULL,
    config JSONB DEFAULT '{}',
    ui_position_x DOUBLE PRECISION DEFAULT 0,
    ui_position_y DOUBLE PRECISION DEFAULT 0
);

-- Workflow edges (connections)
CREATE TABLE workflow_edges (
    id UUID PRIMARY KEY,
    workflow_id UUID NOT NULL REFERENCES workflows(id),
    source_node_id UUID NOT NULL REFERENCES workflow_nodes(id),
    target_node_id UUID NOT NULL REFERENCES workflow_nodes(id),
    source_handle VARCHAR(50), -- for condition nodes: 'yes' or 'no'
    label VARCHAR(100)
);

-- Workflow executions
CREATE TABLE workflow_runs (
    id UUID PRIMARY KEY,
    workflow_id UUID NOT NULL REFERENCES workflows(id),
    tenant_id UUID NOT NULL,
    trigger_type VARCHAR(50) NOT NULL,
    trigger_data JSONB,
    status VARCHAR(50) NOT NULL,
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    error_message TEXT
);

-- Node executions
CREATE TABLE workflow_node_runs (
    id UUID PRIMARY KEY,
    workflow_run_id UUID NOT NULL REFERENCES workflow_runs(id),
    node_id UUID NOT NULL REFERENCES workflow_nodes(id),
    status VARCHAR(50) NOT NULL,
    input JSONB,
    output JSONB,
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    error_message TEXT
);
```

### Phase 4: Workflow-Pipeline Integration 🔜 TODO (Future)

**Mục tiêu:** Cho phép Workflow trigger Pipeline

**Action Type: "trigger_pipeline"**
```go
// workflow/entity.go
type ActionConfig struct {
    Type string `json:"type"` // "trigger_pipeline", "send_notification", etc.

    // For trigger_pipeline
    PipelineID string `json:"pipeline_id,omitempty"`
    PassContext bool `json:"pass_context,omitempty"`

    // For send_notification
    Channel string `json:"channel,omitempty"`
    Template string `json:"template,omitempty"`

    // For create_ticket
    TicketType string `json:"ticket_type,omitempty"`
    Assignee string `json:"assignee,omitempty"`
}

// workflow_executor.go
func (e *WorkflowExecutor) executeActionNode(ctx context.Context, node *WorkflowNode, input map[string]any) error {
    switch node.Config.Type {
    case "trigger_pipeline":
        return e.pipelineService.TriggerPipeline(ctx, app.TriggerPipelineInput{
            TemplateID:  shared.MustParseID(node.Config.PipelineID),
            TriggerType: pipeline.TriggerTypeAPI,
            Context:     input, // Pass workflow context to pipeline
        })
    case "send_notification":
        return e.notificationService.Send(ctx, node.Config.Channel, node.Config.Template, input)
    case "create_ticket":
        return e.ticketService.Create(ctx, node.Config.TicketType, node.Config.Assignee, input)
    // ... other action types
    }
}
```

### Phase 5: Connect Workflows Frontend 🔜 TODO (Future)

**Mục tiêu:** Kết nối frontend với backend API

- [ ] Replace mock data with API calls
- [ ] Add CRUD operations for workflows
- [ ] Add "Run Pipeline" action in workflow builder
- [ ] Add pipeline selector component
- [ ] Real-time execution monitoring

---

## 4. Current State Summary

| Component | Status | Location |
|-----------|--------|----------|
| Pipeline Backend | ✅ Done | `api/internal/domain/pipeline/` |
| Pipeline API | ✅ Done | `api/internal/infra/http/handler/pipeline_handler.go` |
| Pipeline Frontend Types | ✅ Done | `ui/src/lib/api/pipeline-types.ts` |
| Pipeline Frontend Hooks | ✅ Done | `ui/src/lib/api/pipeline-hooks.ts` |
| Pipeline Page | ✅ Done | `ui/src/app/(dashboard)/(mobilization)/pipelines/` |
| WorkflowBuilder Component | ✅ Done | `ui/src/features/pipelines/components/` |
| Workflow Backend | ❌ TODO | - |
| Workflow API | ❌ TODO | - |
| Workflow-Pipeline Integration | ❌ TODO | - |

---

## 5. Lưu ý quan trọng

1. **Không xóa code đã làm** - Pipeline backend và frontend đều hoạt động tốt
2. **Reuse WorkflowBuilder** - Component này có thể dùng cho cả 2 (pipelines và workflows)
3. **Naming convention:**
   - Pipelines = Scan execution (chạy scanner)
   - Workflows = Automation (notify, assign, ticket)
4. **Integration point:** Workflows có thể trigger Pipelines thông qua action type "trigger_pipeline"

---

## 6. Next Steps (Priority Order)

1. ✅ ~~Hoàn thành Pipeline frontend~~ (DONE)
2. 🔜 Rename/clarify page titles để phân biệt
3. 🔜 Update navigation menu
4. 📅 (Future) Build Workflows backend
5. 📅 (Future) Connect Workflows frontend to API
6. 📅 (Future) Add trigger_pipeline action type
