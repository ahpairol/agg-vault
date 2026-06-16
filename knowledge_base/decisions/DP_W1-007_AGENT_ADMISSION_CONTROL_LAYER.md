# Decision Packet: Agent Admission Control Layer

**ID:** DP-W1-007
**Fecha:** 2026-06-16
**Autor:** Claude Cowork (W1-007)
**Aprobado por:** Andrew (HITL implícito al aprobar W1-007)
**Referencia:** AGG-MVP01-W1-007

---

## Decision

AGG introduces an Agent Admission Control Layer before material actions.

Todo agente que opere bajo `AGG/` debe pasar un admission check antes de ejecutar cualquier acción que cambie estado o acceda a servicios globales. Un agente que no pase el check debe detenerse, reportar el bloqueo, y no ejecutar la acción.

---

## Rationale

AGG puede contener agentes creados manualmente, clonados de otros entornos, o heredados de repos externos. La presencia en una carpeta no es suficiente para confiar en un agente.

Riesgos específicos identificados:

```text
1. Andrew puede crear agentes manualmente copiando instrucciones — sin registro formal.
2. Se pueden clonar repos con agentes heredados que no conocen las reglas de AGG.
3. Un agente puede existir en una carpeta pero no estar gobernado por AGG.
4. Un agente puede intentar escribir fuera de su sandbox.
5. Un agente puede usar herramientas globales sin permiso.
```

Sin un control de admisión formal, cualquier instrucción en una carpeta puede actuar como agente y ejecutar acciones materiales.

---

## Consequences

La implementación de W1-007 establece:

1. **agent_registry.yaml** como fuente canónica de agentes admitidos.
2. **agent_policy_profiles.yaml** como fuente de perfiles de política reutilizables.
3. **agent_admission_rules.yaml** como fuente de reglas de comportamiento por defecto.
4. **agent_admission_check.py** como validador oficial de admisión.
5. **run_admission_check.ps1** como runner oficial para invocar la validación.
6. **skill-creator** como la única autoridad para proponer nuevos registros de agentes (con HITL de Andrew).
7. **quarantine_agent.py** como herramienta para documentar agentes no admitidos.

Consecuencias operativas:

```text
- Los agentes deben tener manifest, entrada en registry, policy profile y scope alignment.
- Los agentes no admitidos se detienen y reportan.
- La self-registration está prohibida.
- Todo registro de agente requiere HITL de Andrew.
- Los agentes en status planned o suspended no pueden ejecutar acciones materiales.
```

---

## Scope

Esta capa es MVP-01. No implementa:

```text
- Firmas criptográficas de manifests
- Fingerprints de proceso
- Self-registration automática
- RBAC complejo
- Dashboard de agentes
- Integración n8n o Jira
- Automatización de cuarentena por filesystem watcher
- Modificación de schema MySQL
- Milvus
```

Estas capacidades se documentan como pendientes para fases posteriores.

---

## Archivos creados

```text
agg-main/config/agent_registry.yaml                          (nuevo)
agg-main/config/agent_policy_profiles.yaml                   (nuevo)
agg-main/config/agent_admission_rules.yaml                   (nuevo)
agg-main/scripts/governance/__init__.py                      (nuevo)
agg-main/scripts/governance/agent_admission_check.py         (nuevo)
agg-main/scripts/governance/run_admission_check.ps1          (nuevo)
agg-main/scripts/governance/validate_agent_registry.py       (nuevo)
agg-main/scripts/governance/quarantine_agent.py              (nuevo)
agg-main/templates/agents/agent_manifest.template.yaml       (nuevo)
agg-main/templates/agents/emma-advisor.test.manifest.yaml    (nuevo)
agg-main/templates/agents/unknown-agent.test.manifest.yaml   (nuevo)
agg-main/templates/agents/dashboard-metrics-reader.test.manifest.yaml (nuevo)
agg-main/docs/contracts/CONTRACT_AGENT_ADMISSION.md          (nuevo)
agg-main/docs/contracts/CONTRACT_AGENT_REGISTRY.md           (nuevo)
agg-main/docs/contracts/CONTRACT_AGENT_QUARANTINE.md         (nuevo)
agg-main/AGENTS.md                                           (actualizado — sección 12)
vault-local/knowledge_base/sops/SOP_AGENT_ADMISSION_CONTROL.md (nuevo)
vault-local/knowledge_base/decisions/DP_W1-007_AGENT_ADMISSION_CONTROL_LAYER.md (nuevo)
```

---

## Lesson Learned

La gobernanza de agentes en un sistema multi-agente emergente requiere que el control de identidad y alcance esté establecido desde el primer agente, no cuando ya existen múltiples agentes descontrolados. El costo de establecer esta capa ahora es mínimo; el costo de hacerlo después de escalar sería alto.

---

## References

- SOP: `vault-local/knowledge_base/sops/SOP_AGENT_ADMISSION_CONTROL.md`
- Admission contract: `agg-main/docs/contracts/CONTRACT_AGENT_ADMISSION.md`
- Registry contract: `agg-main/docs/contracts/CONTRACT_AGENT_REGISTRY.md`
- Quarantine contract: `agg-main/docs/contracts/CONTRACT_AGENT_QUARANTINE.md`
- Registry: `agg-main/config/agent_registry.yaml`
