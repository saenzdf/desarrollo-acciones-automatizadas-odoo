---
name: odoo-studio-automations
description: >-
  Diseña e implementa acciones automatizadas de Odoo 19 Studio (base.automation,
  ir.actions.server, Execute Code). Pregunta modelo, trigger, instancia y tipo de
  acción antes de codificar. Consulta MCP/XML-RPC, safe_eval y skills odoo-development.
  Usar cuando el usuario pida automatización en Odoo, Studio, regla al cambiar etapa,
  server action, safe_eval o flujo de proyecto.
---

# Odoo Studio — Acciones automatizadas

Guía para crear o modificar **automatizaciones en Odoo Online/Studio**. El script **vive en Odoo** (`base.automation` + `ir.actions.server`), no en servicios externos salvo que la acción sea explícitamente un webhook.

## Antes de implementar: preguntas obligatorias

Si falta alguna respuesta, **preguntar al usuario** antes de escribir código o tocar la instancia.

| # | Pregunta | Por qué importa |
|---|----------|-----------------|
| 1 | **¿En qué modelo?** (`project.task`, `ir.attachment`, `sale.order`, …) | Define dónde se abre Studio y qué registros recibe `records` |
| 2 | **¿Cuál es el trigger?** (etapa, campo, creación, adjunto) | Determina `trigger` en `base.automation` |
| 3 | **¿Instancia test o producción?** | Credenciales y ids de etapa pueden variar |
| 4 | **¿La lógica es interna en Odoo o externa?** | Execute Code vs Send Webhook |
| 5 | **¿Dominio / condición exacta?** (nombre etapa, campo Studio, mimetype) | `filter_domain` — **nunca asumir ids sin consultar live** |
| 6 | **¿Acción destructiva?** (borrar adjuntos, escribir masivo) | Exigir inventario + confirmación antes de `unlink` |

Plantilla para el agente:

```
Para esta automatización necesito confirmar:
- Modelo: …
- Trigger: …
- Instancia: test / prod
- Acción: Execute Code / Webhook / botón manual
- Condición (dominio o etapa): …
```

## Dónde se configura (UI Studio)

1. Abrir la app del modelo (p. ej. **Proyecto**)
2. Abrir un registro representativo (p. ej. una **Task**)
3. **Studio** → pestaña **Automations** → **New**
4. Configurar trigger + dominio + acción
5. Guardar; documentar en el repo del cliente (`docs/odoo/` si existe)

Para **botón manual**: Studio → **Server Action** + botón en vista formulario.

## Triggers habituales (Odoo 19)

| Necesidad de negocio | `trigger` técnico | Notas |
|----------------------|-------------------|-------|
| Al **entrar a una etapa** | `on_stage_set` | Dominio `stage_id` — consultar ids en cada BD |
| Al **cambiar un campo** | `on_create_or_write` | *When updating field* |
| Al **crear** un registro | `on_create` | Dominio opcional en Apply on |
| Al **subir adjunto** | `on_create` en `ir.attachment` | Dominio `res_model`, `mimetype` |
| Solo cuando el **usuario decida** | Server Action manual | Sin `base.automation` |
| Lógica **fuera de Odoo** | Send Webhook | URL + headers + JSON body |

**Execute Code** para lógica interna (`unlink`, `write`, `message_post`). **Webhook** solo si el procesamiento es externo.

## Execute Code — harness Python (Odoo 19 safe_eval)

Leer también `python-odoo-cursor-rules` y docs Odoo 19 SaaS (Context7 `/websites/odoo_saas-19_1` → Automated Actions → Execute Code).

**Disponible:** `env`, `records`, `record`, `model`, `log(...)`, `UserError`, `time`, `datetime`, `dateutil`, `timezone`, `float_compare`, `Command`, ORM.

**Prohibido:** `import`, librerías externas, `re`, lambdas en `filtered()`.

```python
# Varias tareas
for task in records:
    ...

# Una tarea
if record:
    ...
```

Antes de escribir Python, revisar acciones similares en `ir.actions.server` (`state=code`) vía MCP o XML-RPC.

## Flujo de trabajo del agente

```
- [ ] 1. Leer skills: odoo-development, python-odoo-cursor-rules (y este skill)
- [ ] 2. Responder preguntas obligatorias (modelo, trigger, instancia, acción)
- [ ] 3. Read-only: listar base.automation + ir.actions.server del modelo
- [ ] 4. Resolver ids reales (project.task.type, campos x_studio_*)
- [ ] 5. Si destructivo: inventario → confirmación usuario → ejecutar
- [ ] 6. Crear/actualizar regla (Studio UI o API con cuidado)
- [ ] 7. Probar en test antes de prod
- [ ] 8. Documentar
```

## Consultas XML-RPC / MCP (read-only)

```python
base.automation.search_read(
    [('model_name', '=', 'project.task')],
    fields=['name', 'trigger', 'filter_domain', 'action_server_ids'],
)

ir.actions.server.search_read(
    [('model_id.model', '=', 'project.task'), ('state', '=', 'code')],
    fields=['name', 'code'],
)

project.task.type.search_read(
    [('name', 'ilike', 'Nombre etapa')],
    fields=['id', 'name', 'project_ids'],
)
```

Dominio como **primer argumento** de `search_read`, no lista anidada extra.

## Skills relacionados

| Skill | Ruta típica | Uso |
|-------|-------------|-----|
| `odoo-development` | `~/.agents/skills/odoo-development/` | ORM, módulos, security |
| `python-odoo-cursor-rules` | `~/.agents/skills/python-odoo-cursor-rules/` | Python/Odoo convenciones |
| `odoo-studio-automations` | Este skill | Studio, triggers, safe_eval |

## Crear regla vía API

1. `ir.actions.server` (`state='code'`, `model_id`)
2. `base.automation` (`trigger`, `filter_domain`, `action_server_ids`)
3. M2M: si `create()` devuelve `[id]`, enlazar con entero `id`

Preferir **Studio UI** para visibilidad del equipo.

## Fuera de alcance por defecto

- Asumir ids de etapa entre entornos sin verificar
- Scripts locales permanentes cuando la lógica debe vivir en Odoo
- Módulos custom si Studio + Execute Code alcanza (Odoo Online)

## Referencia extendida

[reference.md](reference.md) — plantillas, checklist, errores frecuentes.

## Caso de referencia (Life Deportes)

Ejemplo real: borrar `.cdr` al entrar a etapa «Cobro y entrega» en `project.task`, trigger `on_stage_set`, dominio `stage_id in (38, 39)`. Ver documentación en el repo del cliente.
