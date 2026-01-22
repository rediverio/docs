---
layout: default
title: Feature Roadmap
parent: UI Documentation
---

# Feature Roadmap

**Last Updated:** 2026-01-08

This document lists planned features and development phases for the platform.

---

## Overview

The Platform follows the 5-stage CTEM (Continuous Threat Exposure Management) framework:

| Phase | Status | Description |
|-------|--------|-------------|
| Scoping | Partial | Define attack surface and business context |
| Discovery | Partial | Identify assets, vulnerabilities, exposures |
| Prioritization | Partial | Rank risks based on impact |
| Validation | Partial | Verify threats and test controls |
| Mobilization | Partial | Execute remediation |

---

## Implemented Features (Current)

### Dashboard
- [x] CTEM Process Overview
- [x] Quick Actions
- [x] Security Metrics

### Scoping
- [x] Attack Surface Overview
- [x] Asset Groups Management
- [x] Scope Configuration

### Discovery
- [x] Scan Management
- [x] Scan Runners
- [x] Assets: Domains, Websites, Services, Repositories, Cloud Resources
- [x] Credential Leaks

### Prioritization
- [x] Risk Analysis Dashboard
- [x] Business Impact Assessment

### Validation
- [x] Attack Simulation
- [x] Control Testing

### Mobilization
- [x] Remediation Tasks
- [x] Workflows

### Settings
- [x] Tenant Settings
- [x] Users & Roles
- [x] Access Control (Groups & Permission Sets)
- [x] Integrations

---

## Planned Features (Roadmap)

### Phase 1: Scoping - Business Context

#### Business Units
- **Description:** Organize assets by business unit/department.
- **Priority:** High
- **Features:** Grouping, Risk aggregation, Department ownership.

#### Crown Jewels
- **Description:** Identify and protect critical assets.
- **Priority:** High
- **Features:** Criticality tagging, Impact classification.

### Phase 2: Discovery - Extended Assets

#### Extended Inventory
- **Hosts:** Server & Endpoint inventory (OS fingerprinting, software inventory).
- **Containers:** K8s & Registry scanning.
- **Databases:** Schema analysis & access review.
- **Mobile Apps:** App catalog & API discovery.

### Phase 2: Discovery - Exposures

#### Vulnerabilities & Misconfigurations
- **Vulnerabilities:** Centralized CVE tracking & patching.
- **Misconfigurations:** CIS benchmarks & IaC analysis.
- **Secrets:** Code & API key exposure detection.

### Phase 3: Prioritization - Extended

#### Advanced Scoring
- **Exposure Scoring:** Custom risk scoring engine.
- **Threat Intelligence:** Active threat tracking & EPSS integration.
- **Attack Path Analysis:** Graph-based risk prioritization.

### Phase 4: Validation - Extended

#### Advanced Testing
- **Penetration Testing:** Campaign management & reporting.
- **Response Validation:** Detection rule testing & SLA tracking.

### Phase 5: Mobilization - Extended

#### Collaboration & Operations
- **Ticketing:** Jira/ServiceNow bi-directional sync.
- **Collaboration:** Comments, @mentions, notifications.
- **Exception Management:** Risk acceptance workflows.

---

## Development Priority

### High Priority (Next Sprint) - VALIDATION FIRST
1. **Penetration Testing Module** (Campaigns, Findings)
2. **Response Validation** (Detection tests)
3. **Vulnerabilities Management**

### Medium Priority (Q2)
1. Integrations (Jira/ServiceNow)
2. Notification Center
3. Threat Intelligence

### Low Priority (Q3+)
1. Extended Asset Types
2. Advanced Analytics

---

**Last Updated:** 2026-01-08
