# SOP — W1-006 Metrics Logging Local Validation
**ID:** SOP_W1-006_METRICS_LOGGING_LOCAL_VALIDATION
**Versión:** 1.0
**Fecha:** 2026-06-15
**Autor:** Claude Cowork (basado en evidencia de validación de Andrew — Antigravity IDE)
**Estado:** Validado en producción

---

## 1. Purpose

Procedimiento reutilizable para validar que la capa de logging de `agg_metrics` funciona correctamente desde una terminal local con acceso a la red `172.16.0.x`. Cubre desde la carga de variables de entorno hasta la inserción y cierre de sesión.

Usar este SOP al:
- Resetear credenciales y verificar que la nueva contraseña funciona.
- Incorporar un nuevo entorno de ejecución (nueva máquina, nuevo agente).
- Verificar integridad de la capa de logging tras cambios de schema.
- Confirmar que un nuevo script de logging se comporta correctamente.

---

## 2. Preconditions

| Requisito | Detalle |
|---|---|
| Red | Acceso a `172.16.0.12:3306` (LAN/DMZ — no sandbox Claude) |
| Python | ≥ 3.10 |
| pymysql | Instalado (ver paso 4) |
| Usuario MySQL | `agg_metrics_writer` (INSERT/UPDATE — no root) |
| `.env` o env vars | Credenciales cargadas en sesión de terminal — nunca en Git |
| Directorio de trabajo | `D:\Work\ACER-DEV-GD\Projects\Cowork\Projects\AGG\agg-main` |
| `PYTHONPATH` | Apuntando a `agg-main` (ver paso 3) |

---

## 3. Environment variables

Cargar en la sesión PowerShell antes de cualquier ejecución. **Nunca hardcodear la contraseña ni commitearla.**

```powershell
$env:AGG_ROOT        = "D:\Work\ACER-DEV-GD\Projects\Cowork\Projects\AGG"
$env:AGG_MAIN_ROOT   = "D:\Work\ACER-DEV-GD\Projects\Cowork\Projects\AGG\agg-main"
$env:AGG_VAULT_ROOT  = "D:\Work\ACER-DEV-GD\Projects\Cowork\Projects\AGG\vault-local"

$env:AGG_DB_HOST     = "172.16.0.12"
$env:AGG_DB_PORT     = "3306"
$env:AGG_DB_NAME     = "agg_metrics"
$env:AGG_DB_USER     = "agg_metrics_writer"
$env:AGG_DB_PASSWORD = "<contraseña actual — leer de gestor de secretos>"

$env:PYTHONPATH = (Get-Location).Path
```

> **Nota PYTHONPATH:** Los scripts usan imports tipo `from scripts.common.db_client import get_connection`. Sin `PYTHONPATH` apuntando a `agg-main`, Python no resuelve estos imports y los scripts fallan con `ModuleNotFoundError`.

---

## 4. Dependency check

```powershell
pip show pymysql
```

Si no está instalado:

```powershell
pip install pymysql
```

Resultado esperado:
```text
Successfully installed pymysql-x.x.x
```

---

## 5. Validation commands

Ejecutar en este orden exacto. Respetar el protocolo de resolución de padres.

### 5.1 Dry-run pre-inserción

```powershell
python .\scripts\metrics\validate_metrics_insert.py `
  --task-ref W1-006-VALIDATION `
  --session-ref SESS-W1-006-VALIDATION-001 `
  --dry-run
```

Resultado esperado: `"overall": "PASSED"` (task y session aún no existen → dry-run muestra el plan)

---

### 5.2 Crear tarea padre

```powershell
python .\scripts\metrics\log_task.py `
  --task-ref W1-006-VALIDATION `
  --task-type ad_hoc_support `
  --title "Validate W1-006 metrics logging layer" `
  --status active
```

Resultado esperado:
```json
{
  "action": "created",
  "task_ref": "W1-006-VALIDATION",
  "task_id": <int>,
  "status": "active"
}
```

Reejecutar: debe retornar `"action": "skipped"` (idempotencia).

---

### 5.3 Crear sesión

```powershell
python .\scripts\metrics\log_session.py `
  --task-ref W1-006-VALIDATION `
  --session-ref SESS-W1-006-VALIDATION-001 `
  --actor Andrew `
  --actor-role owner `
  --session-type execution `
  --status open `
  --summary "Validate W1-006 metrics logging layer from local terminal"
```

Resultado esperado:
```json
{
  "action": "created",
  "session_ref": "SESS-W1-006-VALIDATION-001",
  "session_id": <int>,
  "task_id": <int>,
  "status": "open"
}
```

---

### 5.4 Resolver padres

```powershell
python .\scripts\metrics\resolve_parents.py `
  --task-ref W1-006-VALIDATION `
  --session-ref SESS-W1-006-VALIDATION-001
```

Resultado esperado:
```json
{
  "task_ref": "W1-006-VALIDATION",
  "task_id": <int>,
  "session_ref": "SESS-W1-006-VALIDATION-001",
  "session_id": <int>,
  "status": "resolved"
}
```

---

### 5.5 Registrar uso de modelo

```powershell
python .\scripts\metrics\log_model_usage.py `
  --task-ref W1-006-VALIDATION `
  --session-ref SESS-W1-006-VALIDATION-001 `
  --provider Other `
  --model-name validation-test `
  --model-role validator `
  --purpose "Validate W1-006 model usage logging" `
  --input-tokens 10 `
  --output-tokens 5 `
  --estimated-cost-usd 0 `
  --cost-tier none `
  --success-status success `
  --notes "Inserted from local terminal during W1-006 validation"
```

Resultado esperado: `"action": "created"` con `usage_id` numérico.

---

### 5.6 Registrar evento HITL

```powershell
python .\scripts\metrics\log_hitl_event.py `
  --task-ref W1-006-VALIDATION `
  --session-ref SESS-W1-006-VALIDATION-001 `
  --trigger-type validation `
  --risk-level low `
  --request-summary "Validate W1-006 HITL logging from local terminal" `
  --decision approved `
  --decided-by Andrew `
  --notes "Test HITL event generated during W1-006 validation"
```

Resultado esperado: `"action": "created"` con `event_id` numérico.

---

### 5.7 Registrar incidente de prueba

```powershell
python .\scripts\metrics\log_incident.py `
  --task-ref W1-006-VALIDATION `
  --session-ref SESS-W1-006-VALIDATION-001 `
  --severity low `
  --category other `
  --description "Synthetic incident to validate incident logging" `
  --status open `
  --notes "Test incident generated during W1-006 validation"
```

Resultado esperado: `"action": "created"` con `incident_id` numérico.

---

### 5.8 Cerrar sesión

```powershell
python .\scripts\metrics\log_session.py `
  --task-ref W1-006-VALIDATION `
  --session-ref SESS-W1-006-VALIDATION-001 `
  --actor Andrew `
  --actor-role owner `
  --status closed `
  --summary "Closed W1-006 validation session after successful tests"
```

Resultado esperado:
```json
{
  "action": "closed",
  "session_ref": "SESS-W1-006-VALIDATION-001",
  "session_id": <int>
}
```

---

### 5.9 Dry-run post-inserción

```powershell
python .\scripts\metrics\validate_metrics_insert.py `
  --task-ref W1-006-VALIDATION `
  --session-ref SESS-W1-006-VALIDATION-001 `
  --dry-run
```

Resultado esperado: `"overall": "PASSED"`, con task y session en status `"found"`.

---

## 6. Expected results summary

| Script | Resultado esperado |
|---|---|
| `validate_metrics_insert.py` (pre) | `PASSED` — plan de inserción visible |
| `log_task.py` | `"action": "created"` → segundo run: `"skipped"` |
| `log_session.py` (open) | `"action": "created"` |
| `resolve_parents.py` | `"status": "resolved"` |
| `log_model_usage.py` | `"action": "created"` |
| `log_hitl_event.py` | `"action": "created"` |
| `log_incident.py` | `"action": "created"` |
| `log_session.py` (close) | `"action": "closed"` |
| `validate_metrics_insert.py` (post) | `PASSED` — task/session en `"found"` |

---

## 7. Cleanup

Al terminar la sesión de validación, limpiar la variable de contraseña de la sesión PowerShell:

```powershell
Remove-Item Env:AGG_DB_PASSWORD
```

Las demás variables de entorno no contienen secretos y pueden mantenerse en la sesión.

> No eliminar los registros de validación de la base de datos. Sirven como evidencia de que el sistema funciona y como seed de datos para pruebas futuras.

---

## 8. Known issues

| Issue | Impacto | Estado |
|---|---|---|
| `PYTHONPATH` no configurado | `ModuleNotFoundError` en todos los scripts | Resolver antes de ejecutar (paso 3) |
| `pymysql` no instalado | `ModuleNotFoundError: No module named 'pymysql'` | Resolver con `pip install pymysql` (paso 4) |
| Sandbox Claude sin acceso a `172.16.0.x` | Scripts no pueden conectar desde Claude Cowork | Ejecutar siempre desde máquina Andrew (LL-001-A) |
| `--session-type` no validado por enum | Valores no estándar aceptados silenciosamente | Hardening pendiente — ver LL_W1-006 |
| Contraseña expuesta durante validación inicial | Rotación inmediata requerida | Resuelta — ver LL_W1-006 hallazgo 9 |
