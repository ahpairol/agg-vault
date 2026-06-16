# SOP: Governed Agent Execution Protocol

**ID:** SOP-W1-008-GOVERNED-EXECUTION
**Versión:** 1.0
**Fecha:** 2026-06-16
**Autor:** Claude Cowork (W1-008)
**Owner:** Andrew
**Aplica a:** todo agente AGG admitido antes de ejecutar una acción material

---

## 1. Purpose

Definir la secuencia obligatoria que todo agente AGG debe seguir antes, durante y después de una acción material, conectando las capas ya validadas: admission control (W1-007), logging de métricas (W1-006), y Vault-First protocol.

---

## 2. When this SOP applies

Ante toda acción material: modificar archivos, ejecutar scripts, escribir en base de datos, registrar métricas, hacer commit, cambiar configuración, usar herramientas globales, crear/modificar agentes, o tocar otro proyecto.

No aplica para acciones no materiales: leer manifest, reportar bloqueo, consultar documentación.

---

## 3. Preconditions

```text
- agg-main/.env existe y contiene variables AGG correctas.
- config/agent_registry.yaml existe y es válido.
- El agente tiene un manifest YAML con agent_id válido.
- El agente tiene task_ref y session_ref disponibles o puede crearlos.
- Ejecutar desde agg-main/ como directorio de trabajo.
```

---

## 4. Governed execution sequence

```text
Step 0  Load Environment
Step 1  Admission Check
Step 2  Vault-First Discovery
Step 3  Task/Session Validation
Step 4  HITL Gate
Step 5  Execute via Official Runner
Step 6  Log Metrics
Step 7  Log Incident if Needed
Step 8  Report Outcome
```

---

## 5. Admission check (Step 1)

```powershell
.\scripts\governance\run_admission_check.ps1 `
  --agent-manifest <path> `
  --action <action> `
  --project <project> `
  --scope-root <scope-root>
```

Corte inmediato si status != ADMITTED. Ver: `CONTRACT_AGENT_ADMISSION.md`.

---

## 6. Vault-First discovery (Step 2)

Consultar en orden:

```text
1. vault-local/knowledge_base/sops/      → buscar SOP relevante
2. vault-local/knowledge_base/lessons_learned/ → buscar LL aplicable
3. vault-local/knowledge_base/decisions/ → buscar DP relacionada
```

Documentar qué se consultó y qué se encontró en el reporte final. Si no existe evidencia, declarar la brecha explícitamente.

---

## 7. Task/Session validation (Step 3)

```powershell
.\scripts\metrics\run_metric_tool.ps1 validate_metrics_insert `
  --task-ref <TASK_REF> `
  --session-ref <SESSION_REF> `
  --dry-run
```

Si la tarea no existe y el agente tiene permiso:

```powershell
.\scripts\metrics\run_metric_tool.ps1 log_task `
  --task-ref <TASK_REF> --task-type <type> --title "<title>" --status active --owner Andrew
```

Si la sesión no existe:

```powershell
.\scripts\metrics\run_metric_tool.ps1 log_session `
  --task-ref <TASK_REF> --session-ref <SESSION_REF> `
  --actor "<agent>" --actor-role executor --session-type execution --status open
```

---

## 8. HITL gate (Step 4)

Detener y solicitar aprobación humana si la acción es:

```text
credential_change / schema_change / delete_data / global_config_change /
modify_agent_registry / cross_project_write / exposición pública / cambio irreversible
```

Registrar HITL event vía `run_metric_tool.ps1 log_hitl_event` antes de detener.

---

## 9. Tool execution via runners (Step 5)

```text
Métricas:    .\scripts\metrics\run_metric_tool.ps1 <tool> [args...]
Governance:  .\scripts\governance\run_admission_check.ps1 [args...]
```

No usar Python directo en producción. No ejecutar herramientas fuera del allowlist.

---

## 10. Metrics logging (Step 6)

Orden obligatorio:

```text
1. log_task (si no existe)
2. log_session (si no existe)
3. resolve_parents
4. log_model_usage / log_hitl_event / log_incident
```

---

## 11. Incident handling (Step 7)

```powershell
.\scripts\metrics\run_metric_tool.ps1 log_incident `
  --task-ref <TASK_REF> `
  --severity <low|medium|high|critical> `
  --category <category> `
  --description "<description>" `
  --status open
```

Registrar siempre que haya: tool_failure, permission_block, path_error, configuration_error, process_deviation, scope_creep, hallucination, rework, sync_issue, security.

---

## 12. Final report (Step 8)

Usar template: `agg-main/templates/execution/governed_execution_report.template.md`

El reporte debe incluir: admission status, Vault evidence, task/session, runners usados, métricas registradas, incidentes/HITL, final status, next action.

---

## 13. Known MVP limitations

```text
- No hay búsqueda vectorial en Vault — consulta manual por nombre de archivo.
- No hay automatización del flujo HITL — espera humana.
- No hay dashboard de ejecución de agentes.
- No hay fingerprint ni firma criptográfica de packets.
- validate_governed_execution.py no ejecuta el packet, solo lo valida estructuralmente.
- Los tests de task/session (Step 3) requieren acceso MySQL desde la máquina de Andrew.
```

---

## References

- Contract: `agg-main/docs/contracts/CONTRACT_GOVERNED_AGENT_EXECUTION.md`
- Tool execution contract: `agg-main/docs/contracts/CONTRACT_AGENT_TOOL_EXECUTION.md`
- Admission contract: `agg-main/docs/contracts/CONTRACT_AGENT_ADMISSION.md`
- Metrics contract: `agg-main/docs/contracts/CONTRACT_METRICS_LOGGING.md`
- Decision: `vault-local/knowledge_base/decisions/DP_W1-008_GOVERNED_AGENT_EXECUTION_PROTOCOL.md`
