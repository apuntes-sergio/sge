---
title: 8.9. Miscelanea
description: Desarrollando máodulos en Odoo. Micelanea
---


## Miscelánea

- Para imprimir en color en consola: `\033[93m` para abrir color y `\033[0m` para cerrar.
- Para obtener la menor potencia de 2 mayor o igual a un número: [stackoverflow.com/a/14267557](http://stackoverflow.com/a/14267557)

### Alertas y excepciones en Odoo

Odoo puede mostrar diferentes tipos de alertas, todas en `openerp.exceptions` (o `odoo.exceptions` en versiones recientes):

- `AccessDenied`
- `DeferredException`
- `QWebException`
- `RedirectWarning`
- `AccessError`
- `MissingError`
- `UserError`
- `ValidationError`

Por ejemplo, para mostrar un aviso:

```python
from odoo import _
from odoo.exceptions import Warning
raise Warning(_('¡Algo ha fallado!'))
```

Para mostrar una advertencia con opción de redirección:

```python
action = self.env.ref('base.action_res_users')
msg = _("No puedes crear un nuevo usuario desde aquí.\nPara crear uno nuevo, ve al panel de configuración.")
raise odoo.exceptions.RedirectWarning(msg, action.id, _('Ir al panel de configuración'))
```

En las restricciones (*constrains*) se debe lanzar un `ValidationError`.

---

### Funciones lambda

Las funciones lambda permiten definir funciones anónimas de una sola línea, útiles para pasar como parámetro a métodos del ORM:

```python
a = lambda x, y: x * y
a(2, 3)  # Devuelve 6
```

Si se necesita más lógica, es mejor definir una función normal.

---

### Configuración de módulos

Para añadir opciones de configuración a un módulo, se puede heredar el modelo `res.config.settings`:

```python
class Config(models.TransientModel):
    _inherit = 'res.config.settings'
    players = fields.Char(string='Jugadores', config_parameter="expanse.players")

    def reset_universe(self):
        print("reset", self)
```

Y en la vista:

```xml
<record id="res_config_settings_view_form_inherit" model="ir.ui.view">
    <field name="name">res.config.settings.view.form.</field>
    <field name="model">res.config.settings</field>
    <field name="priority" eval="25" />
    <field name="inherit_id" ref="base.res_config_settings_view_form" />
    <field name="arch" type="xml">
        <xpath expr="//div[hasclass('settings')]" position="inside">
            <div class="app_settings_block" data-string="Expanse Settings" string="Expanse Settings" data-key="expanse">
                <div id="players">
                    <h2>Expanse</h2>
                    <button type="object" name="reset_universe" string="Reset Universe" class="btn-primary"/>
                </div>
            </div>
        </xpath>
    </field>
</record>
```

Si en `data-key` se pone el nombre del módulo, se añadirá el icono al menú de configuración.

