# Decision Packet: Governed Agent Execution Protocol

**ID:** DP-W1-008
**Fecha:** 2026-06-16
**Autor:** Claude Cowork (W1-008)
**Aprobado por:** Andrew (HITL implícito al aprobar W1-008)
**Referencia:** AGG-MVP01-W1-008

---

## Decision

AGG introduces a governed execution protocol required before material agent actions.

Todo agente AGG admitido debe seguir una secuencia determinista de 9 pasos (Step 0–8) antes, durante y después de ejecutar cualquier acción material. El protocolo es obligatorio. Ningún step puede omitirse.

---

## Rationale

Tras W1-006 (logging layer) y W1-007 (admission control), AGG tenía las piezas pero no la secuencia. Los riesgos detectados:

```text
- Un agente podía pasar admission check pero no consultar el Vault.
- Podía ejecutar herramientas sin task/session activa.
- Podía omitir el registro de métricas.
- Podía no saber cuándo detenerse por HITL.
- Podía mezclar runners oficiales con comandos directos de Python.
- Podía actuar sin reportar el outcome.
```

Sin un protocolo explícito, cada agente podía improvisar su flujo — aumentando hallucinations, rework, gaps de métricas y riesgo de seguridad.

---

## Consequences

La implementación de W1-008 establece:

```text
- CONTRACT_GOVERNED_AGENT_EXECUTION.md como referencia operativa del protocolo.
- CONTRACT_AGENT_TOOL_EXECUTION.md como referencia de uso de runners oficiales.
- governed_execution_packet.template.yaml como estructura mínima de ejecución.
- governed_execution_report.template.md como formato de reporte de outcome.
- validate_governed_execution.py como validator de packet pre-ejecución.
- W1-008_validation_packet.yaml como ejemplo de packet válido.
- SOP_GOVERNED_AGENT_EXECUTION_PROTOCOL.md en Vault como guía operativa.
```

Consecuencias operativas:

```text
- Los agentes deben pasar admission check ANTES de actuar.
- Los agentes deben consultar Vault-First ANTES de actuar.
- Los agentes deben validar task/session ANTES de registrar métricas.
- Los agentes deben pasar HITL gate para acciones críticas.
- Los agentes solo pueden ejecutar herramientas via runners oficiales.
- Los agentes deben registrar model_usage, hitl_events e incidents según el resultado.
- Los agentes deben entregar un reporte de outcome al finalizar.
```

---

## Scope

MVP-01. No incluye:

```text
- Automatización del flujo completo (n8n, Jira, webhook)
- Dashboard de ejecución de agentes
- Búsqueda vectorial de Vault (Milvus)
- Firma criptográfica de execution packets
- RBAC avanzado
- Nuevos agentes reales de proyecto
- Cambios de schema MySQL
```

---

## Archivos creados

```text
agg-main/docs/contracts/CONTRACT_GOVERNED_AGENT_EXECUTION.md     (nuevo)
agg-main/docs/contracts/CONTRACT_AGENT_TOOL_EXECUTION.md         (nuevo)
agg-main/templates/execution/governed_execution_packet.template.yaml (nuevo)
agg-main/templates/execution/governed_execution_report.template.md   (nuevo)
agg-main/scripts/governance/validate_governed_execution.py        (nuevo)
agg-main/examples/governed_execution/W1-008_validation_packet.yaml  (nuevo)
vault-local/knowledge_base/sops/SOP_GOVERNED_AGENT_EXECUTION_PROTOCOL.md (nuevo)
vault-local/knowledge_base/decisions/DP_W1-008_GOVERNED_AGENT_EXECUTION_PROTOCOL.md (nuevo)
```

---

## Lesson Learned

La gobernanza de agentes no termina con el admission check. Un agente admitido sin protocolo operativo puede causar el mismo daño que uno no admitido, pero de forma más difícil de detectar. El protocolo de ejecución gobernado es la capa que convierte el control de identidad en control de comportamiento.

---

## References

- SOP: `vault-local/knowledge_base/sops/SOP_GOVERNED_AGENT_EXECUTION_PROTOCOL.md`
- Contract: `agg-main/docs/contracts/CONTRACT_GOVERNED_AGENT_EXECUTION.md`
- Tool contract: `agg-main/docs/contracts/CONTRACT_AGENT_TOOL_EXECUTION.md`
- Admission DP: `vault-local/knowledge_base/decisions/DP_W1-007_AGENT_ADMISSION_CONTROL_LAYER.md`
- Metrics DP: `vault-local/knowledge_base/decisions/DP_W1-006_METRICS_LOGGING_LAYER_VALIDATED.md`
