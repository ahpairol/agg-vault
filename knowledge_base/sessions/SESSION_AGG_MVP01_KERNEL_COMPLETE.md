# Session Snapshot: AGG-MVP01-KERNEL-COMPLETE

**Nombre de sesión:** AGG-MVP01-KERNEL-COMPLETE
**Fecha de cierre:** 2026-06-16
**Para retomar:** di "retomemos la sesión AGG-MVP01-KERNEL-COMPLETE"
**Estado:** CERRADA — MVP-01 Local Governance Kernel completo

---

## Triángulo operativo

```text
Emma (GPT Custom — consultora estratégica) → Andrew (HITL/owner) → Claude Cowork (ejecutor)
```

Workflow: Emma propone → Andrew aprueba → Claude Cowork ejecuta.

---

## Estado del proyecto

### Fases completadas en esta sesión

| Fase | Descripción | Estado |
|---|---|---|
| W1-006 | Metrics Logging Layer + Hardening | ✅ Done |
| W1-007 | Agent Admission Control Layer | ✅ Done |
| W1-008 | Governed Agent Execution Protocol | ✅ Done |
| README | Update agg-main/README.md post W1-006/007/008 | ✅ Done |

### Pendientes Andrew (post-sesión)

```text
1. git add . && git commit && git push  (agg-main — W1-007 + W1-008 + README)
2. git add . && git commit && git push  (vault-local — este snapshot)
3. Jira: mover AGG-W1-007 y AGG-W1-008 → Done
4. Ejecutar Test 2: run_admission_check.ps1 con Emma manifest → esperado ADMITTED
5. Ejecutar Test 3: run_metric_tool.ps1 validate_metrics_insert --dry-run → esperado PASSED
```

---

## Arquitectura del kernel MVP-01

### Repositorios

```text
D:\Work\ACER-DEV-GD\Projects\Cowork\Projects\AGG\
├── agg-main/     → gobernanza, config, scripts, contratos (GitHub: ahpairol/AGG-MAIN)
└── vault-local/  → SOPs, LL, DPs, sessions (GitHub: ahpairol/agg-vault)
```

### Los 4 pilares

```text
1. Carga segura de entorno
   scripts/env/load_agg_env.ps1

2. Métricas gobernadas (MySQL agg_metrics @ 172.16.0.12:3306)
   scripts/metrics/run_metric_tool.ps1

3. Admisión de agentes
   scripts/governance/run_admission_check.ps1

4. Protocolo de ejecución gobernada
   templates/execution/governed_execution_packet.template.yaml
   scripts/governance/validate_governed_execution.py
```

### Flujo operativo completo (Step 0–8)

```text
Step 0  Load Environment           → load_agg_env.ps1
Step 1  Admission Check            → run_admission_check.ps1
Step 2  Vault-First Discovery      → vault-local/knowledge_base/
Step 3  Task/Session Validation    → run_metric_tool.ps1 validate_metrics_insert --dry-run
Step 4  HITL Gate                  → detener si acción crítica
Step 5  Execute via Official Runner → run_metric_tool.ps1 / run_admission_check.ps1
Step 6  Log Metrics                → log_model_usage / log_hitl_event
Step 7  Log Incident if Needed     → log_incident
Step 8  Report Outcome             → governed_execution_report.template.md
```

---

## Archivos clave por módulo

### W1-006 — Metrics Logging Layer

```text
scripts/env/load_agg_env.ps1
scripts/metrics/log_task.py
scripts/metrics/log_session.py
scripts/metrics/resolve_parents.py
scripts/metrics/log_model_usage.py
scripts/metrics/log_hitl_event.py
scripts/metrics/log_incident.py
scripts/metrics/validate_metrics_insert.py
scripts/metrics/run_metric_tool.ps1
requirements.txt
config/mysql.env.example
docs/contracts/CONTRACT_DB_ACCESS.md
docs/contracts/CONTRACT_METRICS_LOGGING.md
```

### W1-007 — Agent Admission Control

```text
config/agent_registry.yaml
config/agent_policy_profiles.yaml
config/agent_admission_rules.yaml
scripts/governance/__init__.py
scripts/governance/agent_admission_check.py
scripts/governance/run_admission_check.ps1
scripts/governance/validate_agent_registry.py
scripts/governance/quarantine_agent.py
templates/agents/agent_manifest.template.yaml
templates/agents/emma-advisor.test.manifest.yaml
templates/agents/unknown-agent.test.manifest.yaml
templates/agents/dashboard-metrics-reader.test.manifest.yaml
docs/contracts/CONTRACT_AGENT_ADMISSION.md
docs/contracts/CONTRACT_AGENT_REGISTRY.md
docs/contracts/CONTRACT_AGENT_QUARANTINE.md
AGENTS.md (actualizado — sección 12 Agent Admission Rule + sección 13 Final Rule)
vault-local/knowledge_base/sops/SOP_AGENT_ADMISSION_CONTROL.md
vault-local/knowledge_base/decisions/DP_W1-007_AGENT_ADMISSION_CONTROL_LAYER.md
```

### W1-008 — Governed Agent Execution Protocol

```text
docs/contracts/CONTRACT_GOVERNED_AGENT_EXECUTION.md
docs/contracts/CONTRACT_AGENT_TOOL_EXECUTION.md
templates/execution/governed_execution_packet.template.yaml
templates/execution/governed_execution_report.template.md
scripts/governance/validate_governed_execution.py
examples/governed_execution/W1-008_validation_packet.yaml
vault-local/knowledge_base/sops/SOP_GOVERNED_AGENT_EXECUTION_PROTOCOL.md
vault-local/knowledge_base/decisions/DP_W1-008_GOVERNED_AGENT_EXECUTION_PROTOCOL.md
```

---

## Reglas técnicas críticas (no olvidar)

### MySQL

- Host exclusivo LAN/DMZ: `172.16.0.12:3306` — el sandbox de Cowork NO tiene ruta a este host (LL-001-A)
- DB: `agg_metrics`, tablas: `tasks`, `sessions`, `model_usage`, `hitl_events`, `incidents`
- FK enforcement activo siempre — nunca deshabilitar `FOREIGN_KEY_CHECKS`
- Orden de inserción obligatorio: task → session → model_usage/hitl_events/incidents
- `resolve_parents` antes de insertar hijos
- `session_type` catalog válido: `{planning, execution, validation, debugging, review, support}`

### YAML Windows paths

- Backslashes en YAML double-quoted = escape sequences inválidas (`\s`, `\m`, `\r`)
- **Siempre single quotes** para rutas Windows: `'.\scripts\metrics\run_metric_tool.ps1'`

### PowerShell

- Set env vars: `Set-Item -Path "Env:$key" -Value $value` (NO `$env:($key) = $value`)
- Slice de args: `if ($args.Count -gt 1) { $toolArgs = $args[1..($args.Count - 1)] } else { $toolArgs = @() }`
- Usar ASCII puro en strings de error (sin em dash, sin caracteres UTF-8)

### Admission check

- Exit codes: 0=ADMITTED, 2=HITL_REQUIRED, 1=todo lo demás
- Root detection: walk up desde manifest parent buscando `config/agent_registry.yaml`
- Auto-registration: PROHIBIDO bajo cualquier circunstancia

### Ejecución

- `direct_python_allowed: false` en production — solo via runners PS1
- Runners oficiales: `run_metric_tool.ps1` y `run_admission_check.ps1`
- `validate_governed_execution.py` es read-only — NO ejecuta acciones, solo valida estructura

---

## Agentes registrados (agent_registry.yaml)

| agent_id | status | policy_profile | scope_root |
|---|---|---|---|
| agg-main.skill-creator | active | skill_creator | agg-main |
| agg-main.emma-advisor | active | advisor | agg-main |
| dashboard.metrics-reader | planned | local_analyst | dashboard |

---

## Security guardrails (vigentes MVP-01)

```text
PROHIBIDO:
- Commitear .env, tokens, claves API, credenciales DB, secretos n8n, claves Cloudflare
- root MySQL en scripts
- MySQL fuera de LAN/DMZ
- Auto-registro de agentes
- FOREIGN_KEY_CHECKS = 0
- Insertar hijos sin parent resuelto
- Python directo en producción (solo debugging humano autorizado)

REQUIERE HITL:
- credential_change
- schema_change
- delete_data
- global_config_change
- modify_agent_registry
- cross_project_write
- exposición pública
- costo > USD 10 por operación
- cualquier cambio irreversible
```

---

## Próxima tarea

**W1-009 — First Governed Local Agent Bootstrap**

Objetivo: crear el primer agente local real bajo un proyecto controlado:
1. Registrarlo en `agent_registry.yaml`
2. Crear su manifest
3. Pasar admission check
4. Generar execution packet
5. Consultar Vault-First
6. Validar task/session en MySQL
7. Ejecutar acción mínima vía runner
8. Registrar métricas

---

## Documentos de gobierno de referencia inmediata

```text
agg-main/AGENTS.md                    → lectura obligatoria al inicio de cada sesión
agg-main/GUARDRAILS.md                → autoridad máxima de seguridad
agg-main/docs/contracts/             → 7 contratos operativos
vault-local/knowledge_base/sops/     → SOPs operativos
vault-local/knowledge_base/decisions/→ Decision Packets
```
