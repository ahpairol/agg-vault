---
title: SOP — MySQL agg_metrics DDL Execution
version: 1.0
status: active
created: 2026-06-14
author: Claude Cowork
reviewed_by: Andrew
task: AGG-MVP01-W1-005
jira: AGG-38
tags: [mysql, ddl, agg_metrics, hitl, dbeaver]
---

# SOP — MySQL agg_metrics DDL Execution

## Purpose

Procedure for executing schema DDL against the `agg_metrics` MySQL database on `172.16.0.12:3306`. Covers pre-execution checks, execution method, verification, and known issues.

## Scope

- Server: `172.16.0.12:3306`
- Database: `agg_metrics`
- Authorized users for DDL: `agg_admin` (schema changes), never root
- Applies to: CREATE TABLE, ALTER TABLE, DROP TABLE, index changes

## HITL requirement

**All DDL requires explicit Andrew HITL before execution.**

Required phrase:
```text
Andrew HITL — approve DDL execution for agg_metrics on 172.16.0.12:3306 using the Andrew-created non-root MySQL user, not root.
```

No connection or SQL execution before this phrase is received.

---

## Pre-execution checklist

1. HITL phrase received from Andrew in current session.
2. Confirm target server: `172.16.0.12:3306`.
3. Confirm user: `agg_admin` — not root.
4. Confirm credentials available in local environment (not in Git or vault-local).
5. Run `SHOW DATABASES LIKE 'agg_metrics'` — confirm database exists.
6. Run `USE agg_metrics; SHOW TABLES` — confirm expected schema state.
7. If unexpected tables exist: STOP and report to Andrew before proceeding.

---

## Execution method

### Option A — DBeaver (recommended for AGG MVP)

Claude Cowork's sandbox has no network route to `172.16.0.x`. DDL must be executed by Andrew directly.

1. Open DBeaver connected to `172.16.0.12:3306` as `agg_admin`.
2. In the left panel, right-click `agg_metrics` → **SQL Editor** → **Open SQL Script**.
3. Verify the active schema dropdown shows `agg_metrics`.
4. **Execute each CREATE TABLE statement individually** (Ctrl+Enter per statement).
   - Do NOT paste multiple CREATE TABLE statements and run as a script — DBeaver fails to split them correctly.
   - See Known Issues §1.
5. After each statement, verify no error before proceeding to next.

### Option B — Python script (from a machine with network access to 172.16.0.x)

```bash
# Set credentials via environment variable — never hardcode
$env:MYSQL_PWD="your_password"   # PowerShell
# or
set MYSQL_PWD=your_password       # CMD

python database/schema/run_ddl_w1005.py
```

Script reads `MYSQL_HOST`, `MYSQL_PORT`, `MYSQL_USER`, `MYSQL_PWD` from environment.

### Option C — mysql CLI

```bash
MYSQL_PWD=your_password mysql -h 172.16.0.12 -P 3306 -u agg_admin agg_metrics
```

Then paste CREATE TABLE statements one at a time.

---

## Execution order (FK dependency)

```text
1. tasks          — no FK dependencies
2. sessions       — FK → tasks(id)
3. model_usage    — FK → tasks(id), sessions(id)
4. hitl_events    — FK → tasks(id), sessions(id)
5. incidents      — FK → tasks(id), sessions(id)
```

Steps 3, 4, 5 are independent of each other — only require tasks and sessions to exist first.

---

## Post-execution verification

```sql
-- Confirm all 5 tables
SHOW TABLES;

-- Confirm each table structure
DESCRIBE tasks;
DESCRIBE sessions;
DESCRIBE model_usage;
DESCRIBE hitl_events;
DESCRIBE incidents;

-- Confirm 7 FK constraints
SELECT
  TABLE_NAME,
  CONSTRAINT_NAME,
  REFERENCED_TABLE_NAME,
  REFERENCED_COLUMN_NAME
FROM information_schema.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = 'agg_metrics'
  AND REFERENCED_TABLE_NAME IS NOT NULL
ORDER BY TABLE_NAME;
```

Expected FK result: 7 rows.

| TABLE_NAME | CONSTRAINT_NAME | REFERENCED_TABLE_NAME |
|---|---|---|
| hitl_events | fk_hitl_events_session_id | sessions |
| hitl_events | fk_hitl_events_task_id | tasks |
| incidents | fk_incidents_session_id | sessions |
| incidents | fk_incidents_task_id | tasks |
| model_usage | fk_model_usage_session_id | sessions |
| model_usage | fk_model_usage_task_id | tasks |
| sessions | fk_sessions_task_id | tasks |

---

## Rollback (pre-data only)

Safe only before any operational data is written. Requires separate Andrew HITL.

```sql
DROP TABLE IF EXISTS incidents;
DROP TABLE IF EXISTS hitl_events;
DROP TABLE IF EXISTS model_usage;
DROP TABLE IF EXISTS sessions;
DROP TABLE IF EXISTS tasks;
DROP DATABASE IF EXISTS agg_metrics;
```

After first real data insertion: rollback requires Andrew HITL + explicit decision. GUARDRAILS.md §3 applies.

---

## Known issues

### 1. DBeaver multi-statement script execution

**Symptom:** Running multiple CREATE TABLE statements as a script in DBeaver fails with SQL syntax error pointing to the second statement.

**Cause:** DBeaver's statement delimiter handling in script mode does not reliably split consecutive DDL statements by semicolon.

**Workaround:** Execute each CREATE TABLE statement individually. Select one statement → Ctrl+Enter. Confirm success → move to next.

**Does not affect:** Single-statement execution, SELECT queries, or mysql CLI.

### 2. Claude Cowork sandbox network isolation

**Symptom:** PyMySQL connection to `172.16.0.12` fails with "Network is unreachable" from the Claude Cowork sandbox.

**Cause:** The sandbox Linux environment has no routing to the `172.16.0.x` private network segment.

**Workaround:** Andrew executes DDL directly from his machine (DBeaver, mysql CLI, or Python script).

**Implication:** Any automated logging script that runs inside Claude Cowork and needs to write to `agg_metrics` must also run from a machine with network access to `172.16.0.12`.

---

## Security rules

- Never use root for DDL or application connections.
- Never store credentials in Git, vault-local, task files, DEVLOG, or screenshots.
- Credentials must be passed via environment variables or entered at runtime.
- All DDL requires Andrew HITL — no exceptions.
- FOREIGN_KEY_CHECKS must never be disabled to bypass insertion errors.

---

## Related

- Schema source of truth: `agg-main/database/schema/README.md` v0.2.1-proposal
- DDL review: `agg-main/database/schema/draft_ddl_review.md`
- Task: `agg-main/tasks/active/AGG-MVP01-W1-005-execute-agg-metrics-ddl.md`
- GUARDRAILS: `agg-main/GUARDRAILS.md §3, §6`
- Tool registry: `agg-main/config/tool_registry.yaml`
- Lesson Learned: `vault-local/knowledge_base/lessons_learned/LL_AGG_METRICS_SCHEMA_EXECUTION.md`

---

## Execution log

| Date | Version | Executed by | Method | Result |
|---|---|---|---|---|
| 2026-06-14 20:20:12 | v0.2.1-proposal | Andrew | DBeaver / agg_admin | ✅ 5 tables, 7 FKs verified |
