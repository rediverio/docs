---
layout: default
title: Migrations
parent: Database
nav_order: 2
---

# Database Migrations

We use `golang-migrate` to manage database schema changes. Migrations are stored in `api/migrations/`.

## Migration List

Below is a summary of key migrations applied to the system:

| ID | Name | Description |
|----|------|-------------|
| 001 | `init_extensions` | Enable UUID-ossp, pgcrypto |
| 002 | `users_auth` | Create users table |
| 003 | `tenants` | Multi-tenancy support |
| 004 | `assets` | Asset inventory table |
| 007 | `findings` | Vulnerability findings |
| 045 | `rename_workers_to_agents` | Terminology update |
| 046 | `access_control_foundation` | RBAC tables |
| 047 | `roles_in_database` | Dynamic role management |

## Working with Migrations

### Create a new migration

```bash
make migrate-create name=add_new_table
```

### Apply migrations

```bash
# Apply all pending migrations (up)
make migrate-up

# Rollback last migration (down)
make migrate-down
```
