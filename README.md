# Desarrollo de acciones automatizadas en Odoo

Skill de agente (Cursor / [Agent Skills](https://agentskills.io)) para diseñar e implementar **acciones automatizadas en Odoo 19 Studio**: `base.automation`, `ir.actions.server` y **Execute Code** (Python `safe_eval`).

## Qué incluye

- Preguntas obligatorias antes de implementar (modelo, trigger, instancia, tipo de acción)
- Dónde configurar en Studio
- Tabla de triggers (`on_stage_set`, `on_create_or_write`, `on_create`, webhook, manual)
- Restricciones del harness Python en Odoo Online
- Flujo de trabajo del agente y consultas XML-RPC/MCP
- Plantillas de código y checklist de pruebas

## Instalación

### Cursor / Agent Skills (personal)

```bash
git clone https://github.com/saenzdf/desarrollo-acciones-automatizadas-odoo.git \
  ~/.agents/skills/odoo-studio-automations
```

### Proyecto (copia local)

```bash
cp -r SKILL.md reference.md /ruta/al/proyecto/.cursor/skills/odoo-studio-automations/
```

## Uso

En el chat del agente:

> Usa el skill **odoo-studio-automations** para crear una regla cuando…

El agente debe preguntar **modelo**, **trigger** e **instancia** antes de escribir código.

## Archivos

| Archivo | Contenido |
|---------|-----------|
| [SKILL.md](SKILL.md) | Instrucciones principales |
| [reference.md](reference.md) | Plantillas, checklist, errores |

## Skills complementarios

- **odoo-development** — ORM, módulos, arquitectura
- **python-odoo-cursor-rules** — convenciones Python/Odoo

## Licencia

[MIT](LICENSE)

## Origen

Extraído del flujo real Life Deportes (Odoo Online 19): automatización «Eliminar .cdr en Empacado» (ex Cobro y entrega) con `on_stage_set` y Execute Code.
