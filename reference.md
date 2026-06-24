# Referencia — Odoo Studio Automations

## Checklist de pruebas

- [ ] Regla dispara solo cuando cumple dominio
- [ ] Execute Code compatible con safe_eval (sin import/lambda)
- [ ] Idempotente si se guarda dos veces el registro
- [ ] Datos no objetivo intactos
- [ ] Log (`log()`) o chatter si aplica
- [ ] Probado en **test** antes de **prod**

## Plantilla documentación

```markdown
# [Nombre] — Odoo 19 Studio

## Trigger de negocio
…

## Configuración
| Modelo | … |
| Trigger | … |
| Apply on | … |
| Acción | Execute Code / Webhook / Manual |

## Código / payload
…

## Ids por instancia
| Test | base.automation | ir.actions.server |
| Prod | … | … |
```

## Plantillas Execute Code

### Borrar adjuntos por extensión

```python
for task in records:
    attachments = env['ir.attachment'].search([
        ('res_model', '=', 'project.task'),
        ('res_id', '=', task.id),
        ('name', 'ilike', '%.cdr'),
    ])
    for att in attachments:
        if (att.name or '').lower().endswith('.cdr'):
            log('Eliminado: %s (task %s)' % (att.name, task.id))
            att.unlink()
```

### Mensaje en chatter

```python
for task in records:
    task.message_post(
        body='<p>Mensaje</p>',
        subtype_xmlid='mail.mt_note',
        body_is_html=True,
    )
```

## Errores frecuentes

| Error | Causa | Solución |
|-------|-------|----------|
| safe_eval SyntaxError | `import`, `lambda` | Bucles `for` |
| Regla no dispara | Trigger incorrecto | `on_stage_set` vs `on_create_or_write` |
| Id etapa wrong | Copiar ids entre BDs | Consultar `project.task.type` en cada instancia |
| M2M link falla | `create` devuelve `[id]` | Usar entero en `(4, 0, id)` |
