# vault-local — Knowledge Base AGG

Base de conocimiento local del proyecto AGG. Fuente primaria de edición durante MVP.

## Estructura

```
vault-local/
├── knowledge_base/
│   ├── strategy/        ← decisiones estratégicas, contexto de negocio
│   ├── lessons_learned/ ← aprendizajes registrados por sesión
│   ├── sops/            ← procedimientos operativos estándar
│   ├── tasks/           ← contexto y evidencia de tareas
│   └── agents_journals/ ← bitácoras de agentes por sesión
├── raw/                 ← material en bruto sin procesar
├── templates/           ← plantillas para lecciones, SOPs, journals
└── conflicts/           ← copias conflictivas pendientes de resolución manual
```

## Principio Vault-First

Antes de actuar, diagnosticar o recomendar, consultar en este orden:

1. `knowledge_base/lessons_learned/`
2. `knowledge_base/sops/`
3. `knowledge_base/strategy/`

Si no hay evidencia interna suficiente, declararlo explícitamente. Prohibido inventar.

## Flujo de sincronización

```
vault-local → GitHub push → webhook n8n → pull en Obsidian
```

Fuente primaria: este directorio. Obsidian actúa como espejo/consulta.

## Política de conflictos

Si hay conflicto: no sobrescribir. Mover copia a `conflicts/`, registrar incidente, resolver manualmente.
