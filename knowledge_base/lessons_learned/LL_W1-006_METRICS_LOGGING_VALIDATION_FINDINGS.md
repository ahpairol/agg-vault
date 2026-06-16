# LL — W1-006 Metrics Logging Validation Findings
**ID:** LL_W1-006_METRICS_LOGGING_VALIDATION_FINDINGS
**Versión:** 1.0
**Fecha:** 2026-06-15
**Origen:** Validación local de Andrew desde Antigravity IDE (PowerShell) contra MySQL `agg_metrics` en `172.16.0.12`
**Tarea origen:** AGG-MVP01-W1-006 / W1-006.2A
**Jira:** AGG-39

---

## Resumen

La capa de logging de W1-006 fue validada localmente y funciona correctamente contra MySQL real. Los scripts se ejecutaron end-to-end: task → session → resolve_parents → model_usage → hitl_event → incident → close session.

La fricción principal no estuvo en la lógica de los scripts sino en la preparación del entorno de ejecución (PYTHONPATH, dependencias, credenciales).

---

## Hallazgos operativos

### HL-001 — PYTHONPATH es prerequisito obligatorio

Los scripts usan imports relativos tipo `from scripts.common.db_client import get_connection`. Sin `PYTHONPATH` apuntando a `agg-main`, Python no resuelve el módulo y todos los scripts fallan con `ModuleNotFoundError`.

**Impacto:** Bloqueo total de ejecución si no se configura antes.

**Acción recomendada:** El wrapper de carga de entorno (próximo hardening) debe setear `PYTHONPATH` automáticamente.

---

### HL-002 — pymysql debe instalarse manualmente

`pymysql` no estaba disponible en el entorno inicialmente. Fue instalado con `pip install pymysql`.

**Impacto:** Bloqueo en el primer intento de conexión si no está presente.

**Acción recomendada:** Agregar `pymysql` a un `requirements.txt` en `agg-main/scripts/` o documentarlo como prerequisito explícito en onboarding de entorno.

---

### HL-003 — validate_metrics_insert.py requiere --task-ref incluso en --dry-run

El modo `--dry-run` no omite la validación del `task_ref`. Requiere que el parámetro esté presente aunque la tarea aún no exista en la base de datos.

**Impacto:** Si se llama sin `--task-ref`, falla con error de argparse requerido.

**Comportamiento correcto:** En dry-run con task inexistente, el script reporta el plan de inserción y marca `checks.task.status = "NOT_FOUND"`. No bloquea — informa.

---

### HL-004 — log_session.py usa --actor y --actor-role, no --agent-name

La CLI real usa `--actor` (nombre libre) y `--actor-role` con catálogo `{executor, advisor, owner}`. No existe `--agent-name`.

**Impacto:** Confusión en documentación informal o prompts de agentes que asuman otra nomenclatura.

**Acción recomendada:** La CLI observada es la fuente de verdad. No usar `--agent-name` en instrucciones ni documentación futura.

---

### HL-005 — status válido de sesión: open | closed | interrupted

El catálogo de `log_session.py` tiene exactamente tres valores válidos. Intentar `--status completed` u otros valores retorna error de validación.

**Impacto:** Error de CLI si se usa valor fuera de catálogo.

---

### HL-006 — log_model_usage.py valida catálogos cerrados: provider, model-role, cost-tier

Catálogos observados en validación:

| Campo | Valores válidos |
|---|---|
| `--provider` | `Anthropic`, `Google`, `Local`, `OpenAI`, `Other` |
| `--model-role` | `advisor`, `primary_executor`, `tool`, `validator` |
| `--cost-tier` | `high`, `medium`, `low`, `none` |

**Impacto:** Error de validación si se usa valor fuera de catálogo.

**Acción recomendada:** Estos catálogos deben mantenerse sincronizados con `tool_registry.yaml` y `CONTRACT_METRICS_LOGGING.md`.

---

### HL-007 — log_hitl_event.py requiere campos HITL específicos

Los campos reales son `--trigger-type` (texto libre), `--risk-level` (catálogo), `--request-summary`, `--decision` y `--decided-by`. No existe `--event-type` ni `--summary`.

**Impacto:** Errores de CLI si se usan nombres de campo incorrectos.

---

### HL-008 — log_incident.py usa --category, --description y --status, no --summary

Los campos de `log_incident.py` son `--severity`, `--category`, `--description`, `--status`. No existe `--summary`.

**Impacto:** Confusión con `log_session.py` que sí usa `--summary`.

---

### HL-009 — Contraseña expuesta accidentalmente durante validación inicial y luego rotada

Durante la sesión de validación, la contraseña de `agg_metrics_writer` fue expuesta accidentalmente. La rotación fue inmediata y la nueva credencial fue validada exitosamente (`validate_metrics_insert.py` → `"overall": "PASSED"`).

**Impacto:** Riesgo de seguridad temporal — resuelto.

**Acción preventiva:** El flujo de carga de entorno debe diseñarse para que la contraseña nunca sea visible en terminal, logs ni historial de shell. El próximo hardening debe incluir mecanismo de supresión de echo y limpieza de historial.

**Regla de oro:** Nunca documentar contraseñas reales en ningún archivo, chat, log ni comentario. Solo registrar el evento de rotación, no el valor.

---

### HL-010 — Contraseña rotada validada con dry-run PASSED

La validación post-rotación confirmó que la nueva credencial funciona correctamente. El flujo `validate_metrics_insert.py --dry-run` retornó `"overall": "PASSED"` con conexión a MySQL.

**Impacto:** Confirmación de que el proceso de rotación y re-validación funciona como procedimiento de emergencia reutilizable.

---

### HL-011 — --session-type no tiene validación de enum (hallazgo adicional de Claude Cowork)

Durante la validación, se usó `--session-type validation` que no está en el catálogo definido en el código (`{"execution", "review", "advisory", "closure", "troubleshooting", "hitl"}`). El script lo aceptó silenciosamente porque `session_type` no tiene validación de enum — solo `actor_role` y `status` la tienen.

**Impacto:** Datos inconsistentes pueden entrar en `agg_metrics.sessions.session_type`.

**Acción recomendada (pendiente de hardening):** Agregar validación de `session_type` con el mismo patrón que `actor_role` y `status`. No implementado en esta tarea — documentado para próxima iteración.

---

## Conclusiones

1. **La capa de logging funciona.** Todos los scripts principales fueron ejecutados exitosamente contra MySQL real con resultados esperados.
2. **La fricción principal está en la preparación del entorno**, no en la lógica de los scripts: PYTHONPATH, pymysql, y carga segura de credenciales.
3. **La CLI real es la fuente de verdad.** Cualquier documentación, instrucción de agente o wrapper debe derivarse de la CLI observada, no de supuestos.
4. **El próximo hardening debe crear un flujo oficial de carga de entorno para agentes:** `.env` local no versionado + loader/wrapper oficial que setee PYTHONPATH, verifique dependencias y suprima echo de contraseña.
5. **El proceso de rotación y re-validación funciona** como procedimiento de emergencia. Documentado en este LL.

---

## Próxima acción recomendada

**W1-006.3 (o W1-007, según priorización de Emma):** Hardening de carga de entorno y ejecución segura por agentes.

Objetivos:
- Wrapper PowerShell/Python oficial que cargue `.env` sin exponer contraseña en terminal.
- Verificación automática de PYTHONPATH y dependencias.
- Validación de `session_type` con catálogo cerrado en `log_session.py`.
- `requirements.txt` en `agg-main/scripts/`.
