# SOP: Agent Admission Control

**ID:** SOP-W1-007-ADMISSION
**Versión:** 1.0
**Fecha:** 2026-06-16
**Autor:** Claude Cowork (W1-007)
**Owner:** Andrew
**Aplica a:** todos los agentes bajo AGG v2

---

## 1. Purpose

Garantizar que solo agentes registrados, activos y dentro de su scope autorizado puedan ejecutar acciones materiales en AGG.

El admission check es la primera línea de control operativo para la capa de gobernanza de agentes AGG.

---

## 2. Preconditions

Antes de ejecutar un admission check:

- `agg-main/.env` debe existir y contener las variables de entorno AGG.
- `config/agent_registry.yaml` debe existir y ser válido.
- `config/agent_policy_profiles.yaml` debe existir.
- `config/agent_admission_rules.yaml` debe existir.
- El agente debe tener un manifest YAML con `agent_id`.
- Ejecutar desde `agg-main/` como directorio de trabajo.

---

## 3. Required files

| Archivo | Función |
|---|---|
| `config/agent_registry.yaml` | Registro canónico de agentes admitidos |
| `config/agent_policy_profiles.yaml` | Perfiles de política reutilizables |
| `config/agent_admission_rules.yaml` | Reglas de comportamiento por defecto |
| `scripts/governance/agent_admission_check.py` | Lógica de validación |
| `scripts/governance/run_admission_check.ps1` | Runner oficial (carga entorno) |
| `scripts/governance/validate_agent_registry.py` | Validación estructural del registry |
| `scripts/governance/quarantine_agent.py` | Generador de reportes de cuarentena |
| `templates/agents/agent_manifest.template.yaml` | Template para nuevos manifests |

---

## 4. Admission check command

```powershell
.\scripts\governance\run_admission_check.ps1 `
  --agent-manifest <path-to-manifest.yaml> `
  --action <action> `
  --project <project> `
  --scope-root <scope-root>
```

Ejemplo (Emma advisor, acción de métricas):

```powershell
.\scripts\governance\run_admission_check.ps1 `
  --agent-manifest .\templates\agents\emma-advisor.test.manifest.yaml `
  --action metrics.log_model_usage `
  --project agg-main `
  --scope-root agg-main
```

---

## 5. Expected statuses

| Estado | Exit code | Descripción |
|---|---|---|
| `ADMITTED` | 0 | Agente puede ejecutar la acción. |
| `HITL_REQUIRED` | 2 | Acción requiere aprobación de Andrew. |
| `DENIED` | 1 | Agente registrado pero acción no permitida. |
| `QUARANTINED` | 1 | Agente no registrado o no confiable. |
| `UNKNOWN_AGENT` | 1 | Manifest no tiene `agent_id` válido. |
| `SCOPE_VIOLATION` | 1 | Agente intenta actuar fuera de su sandbox. |
| `REGISTRY_MISMATCH` | 1 | Manifest no coincide con el registry. |
| `ERROR` | 1 | Fallo técnico del admission check. |

---

## 6. What to do if ADMITTED

```text
1. Continuar con la acción material planificada.
2. Ejecutar via runner oficial (run_metric_tool.ps1 o equivalente).
3. Registrar métricas del trabajo realizado.
```

---

## 7. What to do if not ADMITTED

```text
1. Detener inmediatamente. No ejecutar la acción.
2. Leer el campo "reason" del JSON de salida.
3. Según el estado:
   - HITL_REQUIRED   → solicitar aprobación de Andrew antes de continuar.
   - QUARANTINED     → ejecutar quarantine_agent.py y escalar a Andrew.
   - UNKNOWN_AGENT   → verificar manifest, crear agent_id si falta.
   - REGISTRY_MISMATCH → revisar inconsistencia manifest vs. registry; escalar a Andrew.
   - SCOPE_VIOLATION → el agente no puede operar en ese proyecto/scope.
   - DENIED          → la acción no está en el perfil del agente; solicitar cambio de perfil via HITL.
   - ERROR           → revisar logs, verificar archivos de config, reportar incidente.
4. Documentar el bloqueo si corresponde.
```

---

## 8. Quarantine workflow

```text
1. Admission check devuelve QUARANTINED o UNKNOWN_AGENT.
2. Ejecutar quarantine_agent.py:

   python .\scripts\governance\quarantine_agent.py `
     --agent-path <path> `
     --reason "<razon>" `
     --detected-project <proyecto>

3. Revisar el reporte generado en governance/quarantine/.
4. Escalar a Andrew con el reporte.
5. Andrew elige: adopt / migrate / reject / archive.
6. Si adopt: skill-creator crea manifest + solicita registro via HITL.
7. Cerrar con nota en DEVLOG.md.
```

---

## 9. Known limitations — MVP-01

```text
- No hay firmas criptográficas de manifests.
- No hay fingerprints de proceso.
- No hay self-registration automática ni flujo de adopción automatizado.
- No hay dashboard de agentes.
- No hay integración con n8n ni Jira para el flujo de cuarentena.
- No hay watcher de filesystem para detectar agentes nuevos automáticamente.
- La validación de scope_root es por coincidencia de string, no por filesystem enforcement.
- Los agentes en HITL_REQUIRED deben esperar aprobación manual — no hay flujo automático.
- Esta capa solo evalúa admisión. No registra métricas de la propia evaluación.
```

---

## References

- Decision: `vault-local/knowledge_base/decisions/DP_W1-007_AGENT_ADMISSION_CONTROL_LAYER.md`
- Admission contract: `agg-main/docs/contracts/CONTRACT_AGENT_ADMISSION.md`
- Registry contract: `agg-main/docs/contracts/CONTRACT_AGENT_REGISTRY.md`
- Quarantine contract: `agg-main/docs/contracts/CONTRACT_AGENT_QUARANTINE.md`
- Agent registry: `agg-main/config/agent_registry.yaml`
