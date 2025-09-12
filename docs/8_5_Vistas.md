---
title: 8.5. Vistas
description: Desarrollando máodulos en Odoo. Vistas
---

En este capítulo exploraremos cómo funcionan las vistas en Odoo, qué tipos de vistas existen y cómo personalizarlas mediante XML y acciones de servidor. Aprenderás a estructurar correctamente una interfaz de usuario, definir formularios, listas, kanban y gráficos, y gestionar la navegación entre ellos.

Las vistas son la forma en la que se representan los modelos. Si no declaramos vistas, Odoo generará automáticamente una vista de lista o formulario estándar para poder ver los registros de cada modelo. Sin embargo, casi siempre queremos personalizar las vistas, y en ese caso se referencian por un identificador.

Las vistas tienen una prioridad y, si no se especifica el identificador de la que queremos mostrar, se mostrará la de mayor prioridad.

```xml
<record model="ir.ui.view" id="view_id">
    <field name="name">view.name</field>
    <field name="model">object_name</field>
    <field name="priority" eval="16"/>
    <field name="arch" type="xml">
        <!-- contenido de la vista: <form>, <list>, <graph>, ... -->
    </field>
</record>
```

!!! tip
    Las vistas se guardan en el modelo `ir.ui.view`. Todos los elementos de interfaz tienen en su nombre `ir.ui` (Information Repository, User Interface). Los menús están en `ir.ui.menu` y las acciones en `ir.actions.window`.

### Ejemplo de vista formulario

```xml
<record model="ir.ui.view" id="course_form_view">
    <field name="name">course.form</field>
    <field name="model">openacademy.course</field>
    <field name="arch" type="xml">
        <form string="Course Form">
            <sheet>
                <group>
                    <field name="name"/>
                    <field name="description"/>
                </group>
            </sheet>
        </form>
    </field>
</record>
```

Aunque Odoo proporciona una lista y un formulario por defecto, normalmente es necesario mejorarlos. Todas las vistas tienen campos que pueden tener diferentes widgets. En los formularios, podemos adaptar mucho el aspecto con grupos de campos, pestañas, campos ocultos condicionalmente, etc.

## Vistas de lista

A partir de Odoo 18 ya no existen vistas `tree`, todas son `list`. Los ejemplos antiguos funcionarán cambiando la palabra.

Las vistas *list* permiten mostrar registros en formato de tabla, facilitando la visualización y gestión de grandes cantidades de datos. Son útiles para mostrar información resumida de un conjunto de registros, con columnas que muestran los campos más relevantes. Además, pueden incluir funcionalidades como ordenación, filtros y acciones rápidas.

**Ejemplo básico de vista list para el modelo de clientes (`res.partner`):**

```xml
<record id="view_partner_list" model="ir.ui.view">
    <field name="name">res.partner.list</field>
    <field name="model">res.partner</field>
    <field name="arch" type="xml">
        <list>
            <field name="name"/>
            <field name="phone"/>
            <field name="email"/>
            <field name="company_id"/>
        </list>
    </field>
</record>
```

**Elementos principales:**

- `<record>`: Define un nuevo registro en el modelo `ir.ui.view`.
- `<field name="name">`: Nombre único de la vista.
- `<field name="model">`: Modelo al que pertenece la vista.
- `<field name="arch" type="xml">`: Estructura XML de la vista.
- `<list>`: Define una vista de tipo lista.
- `<field name="..."/>`: Columnas que se mostrarán en la lista.

### Colores en listas

En las vistas de lista se puede modificar el **color** de las filas según el contenido de un campo usando la etiqueta **decoration**, que utiliza colores contextuales de Bootstrap:

- `decoration-bf`: Negrita
- `decoration-it`: Cursiva
- `decoration-danger`: Rojo claro
- `decoration-info`: Azul claro
- `decoration-muted`: Gris claro
- `decoration-primary`: Púrpura claro
- `decoration-success`: Verde claro
- `decoration-warning`: Marrón claro

```xml
<list decoration-info="state=='draft'" decoration-danger="state=='trashed'">
    <field name="name"/>
    <field name="state"/>
</list>
```

Para comparar un campo de tipo Date o Datetime se puede usar la variable global de QWeb `current_date`:

```xml
<list decoration-info="start_date==current_date">
    ...
</list>
```

También se puede aplicar **decoration** a campos individuales.

### Editable

Las listas pueden ser **editables** para no tener que abrir un formulario:  
`editable="[top | bottom]"`. Indica dónde se crearán los nuevos registros.  
El atributo `on_write` permite ejecutar un método al editar o crear un elemento.

### Campos invisibles

Para ocultar un campo en la lista pero seguir usándolo:

```xml
<list decoration-info="duration==0">
    <field name="name"/>
    <field name="course_id"/>
    <field name="duration" invisible="1"/>
    <field name="taken_seats" widget="progressbar"/>
</list>
```

### Botones en listas

Las listas pueden tener **botones** con los mismos atributos que los de los formularios.

!!! tip
    En listas dentro de formularios (X2many), el botón se ejecuta en el modelo de la lista, no en el del formulario padre. Para acceder al padre, usa el atributo `parent`.

### Totales

Se pueden calcular totales en listas:

```xml
<field name="amount" sum="Total Amount"/>
```

### Ordenar por un campo

Para ordenar por defecto por un campo no calculado:

```xml
<list default_order="sequence,name desc">
```

Si quieres que siempre se ordene así, añade el atributo `_order` al modelo.

### Agrupar por un campo

Con `default_group_by` (solo funciona con campos almacenados en la base de datos):

```xml
<list default_group_by="born_year">
    <field name="name"/>
    <field name="born_year"/>
    <field name="age"/>
</list>
```

### banner_route

Desde Odoo 12, puedes añadir un banner HTML en listas, formularios, etc., usando `banner_route`:

```xml
<list banner_route="/negocity/city_banner">
```

Debes crear un controlador web que devuelva el HTML para esa ruta.

```python
from odoo import http

class BannerCityController(http.Controller):
    @http.route('/negocity/city_banner', auth='user', type='json')
    def banner(self):
        return {
            'html': """
                <div class="negocity_banner" style="height: 200px; background-size:100%; background-image: url(/negocity/static/src/img/negocity_city.jpg)">
                    <div class="negocity_button" style="position: static; color:#fff;"><a>Generate Cities</a></div>
                </div>
            """
        }
```

Para dar funcionalidad al enlace, usa un `<a>` con `type="action"` y los atributos necesarios:

```xml
<a class="banner_button" type="action" data-reload-on-close="true" 
   role="button" data-method="action_generate_cities" data-model="negocity.city">Generate Cities</a>
```

## Vistas formulario

Para que un formulario quede bien, incluye la etiqueta `<sheet>`, que limita el ancho. Dentro de `<sheet>`, usa `<group>` para organizar los campos. Puedes usar varios grupos y añadir títulos con el atributo `string`.

Para pestañas temáticas, usa `<notebook>` y `<page string="Título">`.  
Para separar grupos, usa `<separator string="Descripción"/>`.

Si un campo One2many muestra una lista no adecuada, puedes modificar la vista de lista por defecto:

```xml
<field name="subscriptions" colspan="4">
   <list>...</list>
</field>
```

O especificar la vista que quieres:

```xml
<field name="subscriptions" context="{'list_view_ref': 'modulo.view_subscriptions_tree'}"/>
```

También puedes especificar la vista formulario para el One2many:

```xml
<field name="m2o_id" context="{'form_view_ref': 'module_name.form_id'}"/>
```

!!! warning
    Las vistas tree embebidas tienen limitaciones respecto a las llamadas con un action. Por ejemplo, no pueden ser agrupadas.

### Valores por defecto en One2many

Para que al crear un registro en un One2many el campo Many2one apunte al padre:

```xml
context="{'default_<campo_many2one>': active_id}"
```

O en un action:

```xml
<field name="context">{"default_doctor": True}</field>
```

!!! tip
    Esta sintaxis sirve para pasar valores por defecto a un formulario llamado desde un action, One2many, botón o menú.

!!! tip
    `active_id` apunta al id del elemento activo. En creación, aún no existe en la base de datos, pero funciona internamente.

!!! tip
    En Odoo 14 ya no es necesario, pero sigue siendo útil para otros Many2one o valores por defecto.

### Domains en Many2one

Puedes filtrar los campos Many2one:

```xml
<field name="hotel" domain="[('ishotel', '=', True)]"/>
```

Funciona también para Many2many.

### Widgets

Algunos campos pueden mostrarse con un **widget** diferente al predeterminado:

```xml
<field name="image" widget="image" class="oe_left oe_avatar"/>
<field name="taken_seats" widget="progressbar"/>
<field name="country_id" widget="selection"/>
<field name="state" widget="statusbar"/>
```

#### Integer y Float

- `widget="integer"`: Muestra el número sin decimales.
- `widget="char"`: Muestra el número como texto.
- `widget="id"`: Muestra el número, no editable.
- `widget="float"`: Muestra el número con decimales.
- `widget="percentpie"`: Gráfico circular de porcentaje.
- `widget="float_time"`: Muestra floats como tiempo.
- `widget="progressbar"`: Barra de progreso (en tree y form).
- `widget="monetary"`: Número con 2 decimales.
- `widget="gauge"`: Gráfico semicircular (solo kanban).

**Ejemplo de Gauge:**

```xml
<field name="current" widget="gauge" options="{'max_field': 'target_goal', 'label_field': 'definition_suffix', 'style': 'width:160px; height: 120px;'}" />
```

#### Char y Text

- `widget="char"`: Editor de una línea.
- `widget="text"`: Editor multilínea.
- `widget="email"`: Enlace para enviar correo.
- `widget="url"`: Enlace http.
- `widget="date"`: Guarda fechas como texto.
- `widget="html"`: Editor WYSIWYG.
- `dashboard_graph`: Gráfico pequeño con datos en JSON.

**Ejemplo de dashboard_graph:**

```xml
<field name="week_ocupation" widget="dashboard_graph" graph_type="bar"/>
```

El valor debe ser un JSON generado en Python, por ejemplo con `json.dumps()`.

#### Boolean

- `web_ribbon`: (Odoo 13) Muestra una cinta en el formulario.
- `boolean_toggle`: Permite activar un booleano en una lista.

#### Date

- `daterange`: Muestra un rango de fechas.

#### Many2one

- `widget="many2one"`: Selector con opción de crear nuevos.
- `widget="many2onebutton"`: Botón para abrir el formulario.

#### Many2many

- `widget="many2many"`: Lista con opción de añadir/eliminar.
- `widget="many2many_tags"`: Etiquetas.
- `widget="many2many_checkboxes"`: Lista de checkboxes.
- `widget="many2many_kanban"`: Kanban de los asociados.
- `widget="x2many_counter"`: Solo muestra la cantidad.
- `many2many_tags_avatar`: Muestra avatares.

#### One2many

- `widget="one2many"`: Por defecto.
- `widget="one2many_list"`: Igual, se mantiene por compatibilidad.

#### Modificar el tree del One2many

Puedes especificar el tree a mostrar:

```xml
<field name="fortress">
   <tree>
     <field name="name"/><field name="level"/>
   </tree>
</field>
```

O forzar un kanban:

```xml
<field name="gallery" mode="kanban,tree" context="{'default_hotel_id':active_id}">
    <kanban>
        <field name="name"/>
        <field name="image"/>
        <templates>
            <t t-name="kanban-box">
                <div class="oe_product_vignette">
                    <a type="open">
                        <img class="oe_kanban_image" style="width:300px; height:auto;"
                        t-att-src="kanban_image('marsans.hotel.galley', 'image', record.id.value)" />
                    </a>
                    <div class="oe_product_desc">
                        <h4>
                            <a type="edit">
                                <field name="name"></field>
                            </a>
                        </h4>
                    </div>
                </div>
            </t>
        </templates>
    </kanban>
</field>
```

#### Binary o Image

- `signature`: Permite firmar en pantalla.
- `image`: Permite opciones como zoom o previsualización.

```xml
<field name="image" widget="image" options='{"zoom": true, "preview_image": "image_128"}'/>
```

#### Selection

```xml
<field name="state" decoration-success="state == 'sale' or state == 'done'" decoration-info="state == 'draft' or state == 'sent'" widget="badge" optional="show"/>
```

#### Fields en trees

- `handle`: Permite ordenar manualmente (el campo debe ser el criterio de ordenación).

### Redimensionar imágenes

Desde Odoo 13, el campo Image permite tener diferentes resoluciones con varios related.

### Botones en formularios

Puedes añadir un botón en el formulario:

```xml
<button name="update_progress" type="object" string="Actualizar" class="oe_highlight"/>
```

El atributo `name` debe coincidir con la función a la que llama.  
El tipo puede ser: `object`, `action`, `url`, `client`.

Para llamar a un action externo:

```xml
<button name="%(launch_mmog_fortress_wizard)d" type="action" string="Lanzar ataque" class="oe_highlight"/>
```

Los botones pueden tener iconos (ver [Odoo Iconos](https://es.slideshare.net/TaiebKristou/odoo-icon-smart-buttons)):

```xml
<button name="test" icon="fa-star-o" confirm="¿Estás seguro?"/>
<button type="object" icon="fa-trash-o" name="unlink"/>
```

Se recomienda poner los botones en el `<header>`:

```xml
<header>
    <field name="state" widget="statusbar"/>
    <button name="accept" type="object" string="Aceptar" class="oe_highlight"/>
    <button special="cancel" string="Cancelar"/>
</header>
```

Los botones pueden tener contexto para enviar información extra al servidor.

#### Smart Buttons

Son botones que, además de ejecutarse, muestran información resumida y una icono. El texto y forma del botón se modifica dinámicamente.

```xml
<div class="oe_button_box">
    <button type="object" class="oe_stat_button" icon="fa-pencil-square-o" name="regenerate_password">
        <div class="o_form_field o_stat_info">
            <span class="o_stat_value">
                <field name="password" string="Password"/>
            </span>
            <span class="o_stat_text">Password</span>
        </div>
    </button>
</div>
```

### Formularios dinámicos

Puedes modificar el comportamiento de los campos según condiciones: ocultar con `invisible`, hacer solo lectura con `readonly` o requerir con `required`.

**Ejemplo de ocultar condicionalmente:**

```xml
<field name="boyfriend_name" invisible="married != False"/>
```

Mostrar solo en modo edición o lectura:

```xml
<field name="partit" class="oe_edit_only"/>
<field name="equip" class="oe_read_only"/>
```

Ocultar un grupo según el valor de un campo:

```xml
<group invisible="state in ['player', 'stats']">
    <field name="dia"/>
</group>
```

Ocultar una columna en una lista y usar el valor del padre:

```xml
<field name="lot_id" attrs="{'column_invisible': [('parent.state', 'not in', ['sale', 'done'])]}"/>
```

**Editar condicionalmente:**

```xml
<field name="name2" readonly="condition == False"/>
```

**Combinando atributos:**

```xml
<field name="name" invisible="condition1 == False" required="condition2 == True" readonly="condition3 == True"/>
<field name="suma" readonly="valor == 'calculat'" invisible="servicio in ['Reparaciones','Mantenimiento'] or cliente == 'Pepe'"/>
```

**readonly con force_save:**

```xml
<field name="salary" readonly="1" force_save="1"/>
```

## Vistas Kanban

Las vistas kanban muestran el modelo en forma de "tarjetas". Se declaran con una mezcla de XML, HTML y plantillas QWeb.

**Ejemplo básico:**

```xml
<record model="ir.ui.view" id="socio_kanban_view">
    <field name="name">cooperativa.socio</field>
    <field name="model">cooperativa.socio</field>
    <field name="arch" type="xml">
        <kanban>
            <field name="name"/>
            <field name="id"/>
            <field name="foto"/>
            <field name="arrobas"/>
            <templates>
                <t t-name="kanban-box">
                    <div class="oe_product_vignette">
                        <a type="open">
                            <img class="oe_kanban_image"
                                 t-att-alt="record.name.value"
                                 t-att-src="kanban_image('cooperativa.socio', 'foto', record.id.value)" />
                        </a>
                        <div class="oe_product_desc">
                            <h4>
                                <a type="edit">
                                    <field name="name"></field>
                                </a>
                            </h4>
                            <ul>
                                <li>Arrobas: <field name="arrobas"></field></li>
                            </ul>
                        </div>
                    </div>
                </t>
            </templates>
        </kanban>
    </field>
</record>
```

En el template QWeb, define `<t t-name="kanban-box">` que se renderiza una vez por cada registro. Puedes usar clases CSS de Odoo como `oe_kanban_vignette` o `oe_kanban_card`.

**Opciones avanzadas:**

- En `<kanban>`:
    - `default_group_by`: Agrupa por un campo.
    - `default_order`: Ordena por un campo.
    - `quick_create`: Permite crear elementos rápidamente.
- En cada `<field>`:
    - `sum`, `avg`, `min`, `max`, `count`: Funciones de agregación.
- En el template:
    - `<a type="open">`, `<a type="edit">`, `<a type="delete">`: Acciones sobre el registro.

**Funciones útiles en QWeb:**

- `kanban_image(model, field, id)`: Devuelve la URL de la imagen.
- `kanban_text_ellipsis(string, size=160)`: Acorta textos largos.
- `kanban_getcolor(raw_value)`: Devuelve un color predefinido.
- `kanban_color(raw_value)`: Usa el campo `color` para definir el color.

**Forms dentro de kanban:**

Desde Odoo 12 se puede incluir un formulario simple dentro de un kanban, útil con `quick_create` activado y agrupado por Many2one.

```xml
<kanban default_group_by="stage_id" class="o_kanban_small_column o_kanban_project_tasks" on_create="quick_create"
 quick_create_view="project.quick_create_task_form" examples="project">
...
</kanban>
```

El formulario referenciado debe ser sencillo:

```xml
<form>
  <group>
     <field name="name" string="Título de la tarea"/>
     <field name="user_id" options="{'no_open': True,'no_create': True}"/>
  </group>
</form>
```

**Imágenes en kanban:**

Puedes usar `kanban_image` o el widget `image` como en los formularios.

## Vistas de búsqueda (search)

Las vistas de búsqueda tienen tres tipos de elementos:

- **field**: Permite buscar en un campo.
- **filter**: Filtro por un valor predeterminado.
- **group**: Agrupa por un criterio.

**Ejemplo básico:**

```xml
<search>
    <field name="name"/>
    <field name="inventor_id"/>
</search>
```

!!! tip
    Los campos deben estar almacenados en la base de datos, aunque sean computed.

Puedes añadir un dominio para búsquedas personalizadas:

```xml
<field name="description" string="Nombre y descripción"
    filter_domain="['|', ('name', 'ilike', self), ('description', 'ilike', self)]"/>
```

**Filtros predefinidos:**

```xml
<filter name="my_ideas" string="Mis ideas" domain="[('inventor_id', '=', uid)]"/>
<filter name="more_100" string="Más de 100 cajas" domain="[('cajones','>',100)]"/>
<filter name="Today" string="Hoy" domain="[('date', '&gt;=', datetime.datetime.now().strftime('%Y-%m-%d 00:00:00')),
                                             ('date', '&lt;=',datetime.datetime.now().strftime('%Y-%m-%d 23:23:59'))]"/>
```

!!! tip
    Los filtros solo pueden comparar un campo con un valor específico. Si necesitas comparar dos campos, usa una función.

#### Operadores para domains

- `like`, `not like`, `=like`, `ilike`, `not ilike`, `=ilike`
- `=?`: Hace el término TRUE si el valor es None o False.
- `in`, `not in`
- `=`, `!=`
- `child_of`
- `<`, `<=`, `>`, `>=`

**Agrupar por un campo:**

```xml
<group string="Agrupar por">
    <filter name="group_by_inventor" string="Inventor" context="{'group_by': 'inventor_id'}"/>
</group>
```

Para agrupar por fecha por día:

```xml
<filter name="group_by_exit_day" string="Salida" context="{'group_by': 'exit_day:day'}"/>
```

**Filtro predefinido en un action:**

```xml
<field name="context">{'search_default_clients':1,"default_is_client": True}</field>
```

## Vistas calendario

Si el modelo tiene un campo date o datetime, puedes mostrar los registros ordenados por tiempo.

Atributos principales:

- `string`: Título de la vista.
- `date_start`: Campo de inicio (date/datetime).
- `date_delay`: Duración en horas.
- `date_stop`: Campo de fin (ignorado si existe `date_delay`).
- `day_length`: Duración de un día (por defecto 8 horas).
- `color`: Campo para distinguir con colores.
- `mode`: Vista inicial (day, week, month).

**Ejemplo:**

```xml
<record model="ir.ui.view" id="session_calendar_view">
    <field name="name">session.calendar</field>
    <field name="model">openacademy.session</field>
    <field name="arch" type="xml">
        <calendar string="Calendario de sesiones" date_start="start_date"
                  date_stop="end_date"
                  color="instructor_id">
            <field name="name"/>
        </calendar>
    </field>
</record>
```

## Vistas gráficas (graph)

Se usan para ver agregaciones sobre los datos.

Atributos:

- `string`: Título.
- `type`: Tipo de gráfico (bar, pie, line).
- `stacked`: Solo para bar, muestra datos apilados.

El primer campo es el eje X, el segundo el eje Y.

Atributos de los campos:

- `name`: Nombre del campo.
- `title`: Nombre en el gráfico.
- `invisible`: No aparece.
- `type`: `row` para agrupar, `col` para líneas, `measure` para datos agregados.

**Ejemplo:**

```xml
<record model="ir.ui.view" id="terraform.planet_changes_graph">
    <field name="name">Planet Changes graph</field>
    <field name="model">terraform.planetary_changes</field>
    <field name="arch" type="xml">
        <graph string="Historial de cambios" type="line">
            <field name="time" type="row"/>
            <field name="planet" type="col"/>
            <field name="greenhouse" type="measure"/>
        </graph>
    </field>
</record>
```

!!! tip
    Las vistas graph en Odoo son limitadas: solo aceptan un elemento en X y los campos deben estar almacenados en la base de datos.

