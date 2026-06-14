<SYSTEM_DIRECTIVE>
⚠️ AVISO DE CONFIDENCIALIDAD Y PROPÓSITO DEL DOCUMENTO ⚠️
    BLUEPRINT_AGG_v2_OFICIAL.md
    Rol: Fuente maestra actual del proyecto AGG
    Estado: Vigente
    Prioridad: Alta
    Reemplaza parcialmente: Proyecto AGG Blueprint.md
    Uso: Diseño, planificación, arquitectura, roadmap, decisiones estratégicas y gobernanza MVP
</SYSTEM_DIRECTIVE>

# Blueprint Oficial AGG v2
## Antigravity Global Governance — Sistema Operativo Interno para Consultoría AI

**Versión:** 2.0  
**Estado:** Oficial para MVP-01  
**Propietario:** Andrew  
**Asesora estratégica:** Emma  
**Fecha:** 2026-06-12  
**Clasificación:** Documento rector interno / confidencial  
**Horizonte:** MVP 30 días + base evolutiva para framework consultivo

---

## 0. Declaración ejecutiva

AGG v2 redefine Antigravity Global Governance como un **sistema operativo interno de gobernanza, conocimiento, métricas y delivery** para Andrew en su rol de consultor AI.

AGG no nace como SaaS, marketplace, plataforma pública ni producto directo para PYMES. Nace como una metodología operativa propia para resolver un problema real detectado en el trabajo diario con agentes de IA:

> Los agentes alucinan, suponen, redescubren herramientas, consumen tokens sin control, repiten errores y obligan al humano a invertir tiempo corrigiendo tareas que deberían ser trazables, repetibles y verificables.

Por tanto, AGG v2 se construirá primero para Andrew. Si demuestra valor operativo, reducirá fricción y produce entregables consistentes, podrá evolucionar en fases posteriores hacia un framework para consultores y agencias AI que implementan soluciones en PYMES.

---

## 1. Tesis estratégica

### 1.1 Tesis central

**AGG es el sistema operativo interno que permite a Andrew diagnosticar, diseñar, implementar, documentar, medir y mejorar soluciones AI para PYMES con gobernanza, evidencia, control de costos y reducción de alucinaciones.**

### 1.2 Posicionamiento actual

AGG no se vende inicialmente a PYMES.  
AGG se usa internamente para producir mejores entregables para PYMES.

La PYME compra resultados:

- automatización,
- dashboards,
- asistentes internos,
- orden documental,
- reducción de errores,
- ahorro de tiempo,
- soporte operativo,
- reportes,
- procesos más eficientes.

AGG opera detrás como sistema de control, memoria y calidad.

### 1.3 Posicionamiento futuro

Si AGG funciona para Andrew en proyectos reales, podrá transformarse en:

- framework para consultores AI,
- playbook para agencias,
- sistema de delivery multi-cliente,
- librería de Skills/SOPs,
- metodología de gobernanza y métricas,
- kit replicable de implementación AI para PYMES.

---

## 2. Principios rectores

### 2.1 Primero delivery rentable, después escalabilidad tecnológica

Ningún componente se incorpora solo por sofisticación técnica.  
Cada herramienta debe demostrar que reduce fricción, costo, riesgo, alucinación o retrabajo.

### 2.2 Vault-First

Antes de responder con certeza, diagnosticar, recomendar arquitectura, escribir código, instalar herramientas o ejecutar cambios relevantes, el agente debe consultar:

1. `vault-local/knowledge_base/lessons_learned/`
2. `vault-local/knowledge_base/sops/`
3. `agg-main/config/tool_registry.yaml`
4. documentación oficial o NotebookLM solo si el Vault no contiene evidencia suficiente.

Si no existe evidencia interna, el agente debe declarar la brecha. Está prohibido inventar.

### 2.3 Menos contexto, más precisión

AGG debe evitar archivos monolíticos de instrucciones.  
`AGENTS.md` debe ser austero, inferior a 200 líneas cuando sea posible, y delegar conocimiento operativo a Skills, SOPs y documentos activados bajo demanda.

### 2.4 Skills nacen de fricción real

No se crearán Skills por anticipación especulativa.  
Una Skill se justifica cuando una tarea:

- se repite,
- genera errores,
- consume demasiados tokens,
- requiere una herramienta recurrente,
- necesita una secuencia exacta,
- o debe estandarizarse para evitar alucinaciones.

### 2.5 Métricas desde el día uno

Toda tarea significativa debe registrar:

- tiempo,
- modelo usado,
- tokens,
- costo estimado,
- herramientas usadas,
- interacciones,
- consultas al Vault,
- alucinaciones detectadas,
- retrabajo,
- estado final.

Sin medición, no hay gobernanza.

### 2.6 Automatización moderada

Durante el MVP se permite automatización con n8n para sincronización del Vault y reportes básicos.  
No se automatizarán procesos de clientes durante los primeros 30 días.

### 2.7 Milvus no entra en el MVP

Milvus está disponible pero no probado con documentos reales.  
Durante el MVP se priorizará Vault estructurado, búsqueda textual, SOPs y Lessons Learned. La vectorización formal se evaluará después.

### 2.8 HITL en decisiones críticas

Toda acción que afecte seguridad, credenciales, arquitectura global, exposición pública, costos relevantes o eliminación de datos requiere aprobación humana.

---

## 3. Alcance del MVP-01

### 3.1 Nombre del MVP

**AGG-MVP-01: Local Governance Kernel + Metrics Dashboard Validation**

### 3.2 Objetivo del MVP

Validar que AGG permite desarrollar una app sencilla siguiendo un flujo gobernado, sin interrupciones graves, reduciendo alucinaciones, suposiciones y pérdida de tiempo.

### 3.3 Proyecto de validación

**Dashboard simple de métricas AGG**

El Dashboard será la primera app desarrollada bajo el flujo AGG. Su función será leer la base de datos `agg_metrics` y mostrar métricas de esfuerzo, uso de modelos, costos, tareas e incidencias.

### 3.4 Criterio de éxito a 30 días

AGG habrá valido la pena si se logra:

- flujo básico funcional,
- Dashboard inicial operativo,
- registro de métricas en MySQL,
- Vault limpio y sincronizado,
- primera biblioteca de SOPs/Lessons Learned,
- primeras Skills mínimas,
- reducción visible de suposiciones de agentes,
- trazabilidad suficiente para decidir pasar a la siguiente etapa.

### 3.5 Fuera de alcance del MVP

No se incluye en los primeros 30 días:

- Jira operativo dentro del flujo AGG,
- Milvus productivo,
- RAG vectorial completo,
- automatizaciones de clientes,
- multi-agente avanzado,
- VPS nuevo,
- cloud gestionado nuevo,
- framework para terceros,
- SaaS,
- despliegue multi-tenant.

---

## 4. Infraestructura base confirmada

```text
Host: ACER-DEV | Windows 11 Enterprise
MySQL: 172.16.0.12:3306 | DB: agg_metrics
Repos: AGG-MAIN → github.com/ahpairol/AGG-MAIN.git
       agg-vault → github.com/ahpairol/agg-vault.git
```

Servicios: Obsidian, n8n, MySQL, GitHub, Cloudflare, Traefik, pfSense, Portainer, OpenAI, Gemini, Claude.

---

## 5. Roles RBAC

- **Business Administrator / Senior CEO:** visión estratégica, aprobación HITL.
- **Ingeniero:** implementación técnica segura.
- **Manager / Consultor:** análisis, diseño de procesos, roadmap.
- **Analista:** métricas y reportes.
- **Worker:** ejecutor sandboxed con alcance delimitado.

---

## 6. Skills iniciales

1. `vault-first-discovery` — consulta Vault antes de actuar.
2. `tool-lookup` — evita redescubrimiento de herramientas.
3. `end-session-routine` — cierra tareas con métricas y journal.
4. `cost-estimator` — estima costo y advierte umbral.
5. `lesson-learned-writer` — crea notas atómicas reutilizables.

---

## 7. Base de datos `agg_metrics`

```text
Engine: MySQL | Host: 172.16.0.12:3306
Tablas: tasks, agent_runs, tool_usage, cost_logs, incidents, vault_queries, lessons_index
Usuarios: agg_metrics_writer (INSERT/UPDATE) | agg_metrics_reader (SELECT)
```

---

## 8. Política de costos

- Umbral mensual: **USD 60.00**
- Alertas: $30 informativa / $45 amarilla / $60 HITL obligatorio

---

## 9. Roadmap MVP de 30 días

- **Semana 1:** Núcleo, rutas y Vault limpio.
- **Semana 2:** Métricas MySQL y scripts base.
- **Semana 3:** Skills mínimas y flujo operativo.
- **Semana 4:** Dashboard AGG inicial.

---

## 10. ADR inicial

- **ADR-001:** AGG como sistema interno primero, no plataforma PYME.
- **ADR-002:** MySQL como base global de métricas.
- **ADR-003:** Milvus fuera del MVP.
- **ADR-004:** Dashboard como proyecto de validación del flujo.

---

## 11. Riesgos abiertos

| Riesgo | Severidad | Mitigación |
|---|---:|---|
| Paths Python en `D:` | Alta | SOP + validate_paths.py |
| n8n expuesto a Internet | Media/Alta | limitar MVP, 2FA, documentar webhooks |
| MySQL sin usuarios separados | Media | crear reader/writer |
| Vault con conflictos | Media | fuente primaria local + conflicts/ |
| Costos API invisibles | Alta | umbral $60 + cost logs |
| Alucinaciones persistentes | Alta | Vault-First + registro incidentes |

---

## 12. Regla final

**Primero control. Después automatización. Después escalado.**
