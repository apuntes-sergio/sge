---
---
title: 4.8. Mejorando vistas
description: Ejemplo guiado de desarrollo de módulo de gestión de proyectos en Odoo
---


**POR REVISAR**

En este apartado vamos a aprender cómo crear y personalizar una vista tipo *tree* en Odoo, aprovechando todas las posibilidades que nos brinda el framework. 

## ¿Qué es una vista List?

La vista *tree* (o lista) es la más sencilla y también la más fácil de personalizar en Odoo. Por defecto, si no definimos una vista *tree* para un modelo, Odoo genera una automáticamente mostrando el campo `name` y otros campos básicos. Por ejemplo, en el módulo que estamos desarrollando, llamado `manage`, aparecen varias vistas *tree* generadas por defecto para los modelos que hemos creado, como `sprint`, `task` y `project`.

Si revisamos el código de nuestra aplicación, veremos que la mayoría de las vistas personalizadas que hemos creado hasta ahora son de tipo formulario (*form*), y apenas hemos definido vistas *tree*. Cuando no añadimos una vista *tree* personalizada, Odoo muestra la predeterminada, que suele ser muy básica.

Por ejemplo, en el modelo `devs`, que hereda de `res.partner` (contactos), la vista *tree* por defecto muestra varios campos heredados de contactos. Si queremos personalizarla, debemos definir nuestra propia vista *tree* en el archivo XML correspondiente.

## Codificación de una vista Tree

Para crear una vista *tree* personalizada, debemos definir un registro en la tabla `ir.ui.view` con un identificador único (`id`), un nombre descriptivo, el modelo al que aplica y la definición XML de la vista. Por ejemplo, para el modelo `task` ya tenemos una vista *tree* personalizada, aunque sea muy similar a la predeterminada.

Supongamos que queremos crear una vista *tree* personalizada para el modelo `sprint`. Copiamos la estructura de la vista *tree* de `task` y la adaptamos:

- Cambiamos el `id` por uno único, como `sprint_list`.
- El nombre también debe ser descriptivo, por ejemplo, `Sprint list`.
- El modelo será `sprint`.
- Añadimos los campos que queremos mostrar: `name`, `description`, `duration`, `start_date`, `end_date`, etc.

Una vez definida la vista y actualizado el módulo, veremos que la lista de sprints muestra los campos seleccionados. Si creamos un nuevo sprint, veremos cómo se actualizan los valores automáticamente, por ejemplo, la fecha de fin calculada a partir de la duración y la fecha de inicio.

## Decoraciones condicionales en vistas Tree

Odoo permite resaltar filas en la vista *tree* según condiciones usando el atributo `decoration`. Por ejemplo, si queremos que los sprints con una duración exacta de 15 días aparezcan en amarillo, podemos usar `decoration-warning` con la condición correspondiente.

```xml
<tree decoration-warning="duration == 15">
  ...
</tree>
```

En la documentación oficial de Odoo puedes consultar todas las opciones de decoración disponibles.

Si queremos resaltar los sprints que están actualmente en curso (es decir, que ya han comenzado pero aún no han terminado), podemos crear un campo computado booleano llamado `active` en el modelo `sprint`. Este campo se calculará automáticamente comprobando si la fecha de inicio es anterior o igual a la fecha actual y la fecha de fin es posterior o igual a hoy.

El método computado podría verse así (en Python):

```python
from odoo import models, fields, api
from datetime import date

class Sprint(models.Model):
  _name = 'sprint'

  start_date = fields.Date()
  end_date = fields.Date()
  duration = fields.Integer()
  active = fields.Boolean(compute='_compute_active')

  @api.depends('start_date', 'duration')
  def _compute_active(self):
    today = date.today()
    for sprint in self:
      if sprint.start_date and sprint.end_date:
        sprint.active = sprint.start_date <= today <= sprint.end_date
      else:
        sprint.active = False
```

Para poder usar este campo en la vista *tree* (aunque no lo mostremos), debemos incluirlo en la definición de la vista. Así, podemos aplicar una decoración, por ejemplo, en azul (`decoration-info`) cuando `active` sea `True`:

```xml
<tree decoration-info="active" ...>
  ...
  <field name="active" invisible="1"/>
</tree>
```

El orden de las decoraciones es importante: si un registro cumple varias condiciones, se aplicará la última que coincida.

## Uso de operadores en condiciones

Si queremos resaltar los sprints cuya duración sea menor que 15 días, debemos usar el operador `<`. Sin embargo, en XML debemos escribirlo como `&lt;` para evitar conflictos con las etiquetas:

```xml
<tree decoration-warning="duration &lt; 15">
  ...
</tree>
```

## Vistas Tree editables

Odoo permite hacer las vistas *tree* editables añadiendo el atributo `editable="top"` (o `bottom`). Esto permite modificar los registros directamente desde la lista, pero limita los campos editables a los que se muestran en la vista. Por lo general, no se recomienda usar esta opción salvo casos muy concretos, ya que se pierde la funcionalidad del formulario completo.

## Sumatorios en vistas Tree

Otra funcionalidad interesante es la posibilidad de mostrar sumatorios en la vista *tree*. Por ejemplo, podemos sumar la duración total de todos los sprints programados, lo cual es útil en contextos como presupuestos o informes de ventas. Para ello, basta con añadir el atributo `sum="Total"` en el campo correspondiente:

```xml
<field name="duration" sum="Total"/>
```

Esto mostrará el total de la columna al final de la lista.

## Recursos y documentación

En la documentación oficial de Odoo encontrarás información detallada sobre todas las opciones de personalización de vistas. Además, en el repositorio de GitHub del proyecto tienes disponibles todos los fuentes y los distintos commits con el progreso realizado.

