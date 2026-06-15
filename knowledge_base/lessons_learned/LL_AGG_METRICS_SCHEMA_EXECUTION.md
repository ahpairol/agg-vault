---
title: "LL — agg_metrics Schema Design and DDL Execution"
id: LL-001
date: 2026-06-14
task: AGG-MVP01-W1-003 / W1-004 / W1-005
jira: AGG-36 / AGG-37 / AGG-38
severity: low
confidence: high
status: closed
tags: [mysql, schema, ddl, dbeaver, hitl, fk, sandbox]
---

# LL-001 — agg_metrics Schema Design and DDL Execution

## Summary

First schema design and DDL execution cycle for AGG v2. Covers decisions made across W1-003 (schema design), W1-004 (DDL readiness), and W1-005 (execution). Three execution friction points encountered and resolved.

---

## What went well

- Two-round Emma + Andrew review process caught real issues before DDL execution:
  - VARCHAR PK → BIGINT AUTO_INCREMENT + `*_ref` pattern (performance)
  - ENUM → VARCHAR with documented controlled values (evolvability)
  - FK ON DELETE semantics (RESTRICT vs SET NULL) — resolved with clear rationale
  - `ad_hoc_support` task_type identified as governance gap — addressed before schema landed in production
- Hard FK enforcement held firm throughout — no pressure to soften to soft FKs
- Zero mismatches between `draft_ddl_review.md` and final executed schema
- Vault-First check prior to DDL drafting confirmed expected gaps (no SOP, no LL) — non-blocking and expected

---

## Lessons learned

### LL-001-A — Claude Cowork sandbox has no network route to private IPs

**What happened:** Claude attempted to connect to `172.16.0.12:3306` via PyMySQL from the Linux sandbox. Connection failed immediately with "Network is unreachable."

**Root cause:** The Cowork sandbox is a network-isolated container with no route to the `172.16.0.x` private segment.

**Impact:** Automated DDL execution from Claude not possible. Andrew executed manually.

**Rule going forward:**
- Any script that writes to `agg_metrics` must run from Andrew's machine or a VM/container with network access to `172.16.0.x`.
- Do not design logging workflows that assume Claude Cowork can reach the MySQL server directly.
- Alternative: expose an API or message queue layer between Claude and agg_metrics if automation is required.

---

### LL-001-B — DBeaver fails to execute multiple CREATE TABLE statements as a script

**What happened:** Pasting all 5 CREATE TABLE statements into DBeaver and running as a script (F5) resulted in SQL syntax errors on the second statement. Running each statement individually (Ctrl+Enter) worked correctly.

**Root cause:** DBeaver's script mode does not reliably split consecutive DDL statements by semicolon delimiter in this MySQL connection configuration.

**Impact:** Low — workaround is simple. No data loss. Added ~10 minutes to execution.

**Rule going forward:**
- When running DDL in DBeaver: execute each CREATE TABLE individually.
- Prefer mysql CLI for multi-statement DDL scripts when available.
- Document this in the MySQL SOP (done: `SOP_MYSQL_AGG_METRICS_DDL_EXECUTION.md §Known Issues §1`).

---

### LL-001-C — Python execution scripts should not be presented as copy-paste SQL

**What happened:** A Python script (`run_ddl_w1005.py`) was created to automate DDL execution. Andrew pasted the file contents directly into MySQL instead of running it as Python. MySQL returned a syntax error on the Python code.

**Root cause:** The script format (Python) was not clearly differentiated from the SQL blocks in the workflow communication.

**Impact:** Low — user error, no harm done. Resolved by providing pure SQL blocks.

**Rule going forward:**
- When the execution environment requires manual steps, provide pure SQL blocks — not Python scripts — as the primary execution artifact.
- Python scripts are appropriate when the operator can run them from a terminal. Otherwise SQL is clearer and safer.
- Label execution artifacts with their type and runtime explicitly at the top.

---

### LL-001-D — Schema review in Markdown before SQL prevents structural errors

**What happened:** By keeping DDL as review Markdown (`draft_ddl_review.md`) rather than a `.sql` file, Emma + Andrew could review and resolve 5 open decisions (OD-001 to OD-005) before a single SQL statement was executed.

**Outcome:** Zero schema corrections needed after execution. First-pass DDL matched design exactly.

**Rule going forward:**
- For any new schema work: design → Markdown review → Emma + Andrew approval → HITL → execution.
- Never execute SQL from a first draft. Always go through the review Markdown step.
- `draft_ddl_review.md` pattern is confirmed as the right approach for AGG DDL governance.

---

## Schema decisions — final state in production

| Decision | Final value | Rationale |
|---|---|---|
| PK type | BIGINT AUTO_INCREMENT | Performance in joins; human refs via `*_ref VARCHAR(100)` |
| VARCHAR vs ENUM | VARCHAR(50/100) for evolving fields | Avoid ALTER TABLE for value set expansion during MVP |
| FK enforcement | Hard (InnoDB constraints) | Audit DB — integrity > flexibility |
| task_id ON DELETE | RESTRICT | Prevent orphan metrics |
| session_id ON DELETE | SET NULL | Child records survive session removal via task_id |
| IF NOT EXISTS | Yes on all CREATE statements | Idempotent execution |
| Charset | utf8mb4 / utf8mb4_unicode_ci | Full Unicode + emoji support |
| Duration stored | No | Computed via TIMESTAMPDIFF |
| run_id | No separate table | session_id = MVP run_id |

---

## Verified production state — 2026-06-14

```text
Server:   172.16.0.12:3306
Database: agg_metrics
User:     agg_admin (non-root)
Tables:   tasks, sessions, model_usage, hitl_events, incidents
FKs:      7 — all verified via information_schema.KEY_COLUMN_USAGE
Root:     not used
```

---

## Related

- DEVLOG: D-017 to D-026
- Schema: `agg-main/database/schema/README.md` v0.2.1-proposal
- DDL review: `agg-main/database/schema/draft_ddl_review.md`
- SOP: `vault-local/knowledge_base/sops/SOP_MYSQL_AGG_METRICS_DDL_EXECUTION.md`
