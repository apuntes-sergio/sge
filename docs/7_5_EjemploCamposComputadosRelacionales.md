---
title: 7.5. Campos computados relacionales
description: Ejemplo guiado de desarrollo de módulo de gestión de proyectos en Odoo
---

En este apartado se abordará el desarrollo de campos computados relacionales en Odoo, utilizando como ejemplo un módulo de gestión de proyectos de software que estamos desarrollando. 

El objetivo es mostrar, de manera estructurada y formal, cómo modelar entidades y relaciones, así como implementar campos computados tanto de tipo `Many2one` como `Many2many`.

Los campos computados relacionales resultan fundamentales en Odoo cuando se requiere que ciertos datos se mantengan actualizados de forma automática en función de la información relacionada en otros modelos. Por ejemplo, permiten que un campo refleje siempre el estado más reciente de una relación, sin necesidad de intervención manual.

En el contexto de la gestión de proyectos, estos campos facilitan la automatización de procesos como la asignación de *sprints* a *tareas* o la recopilación de *tecnologías* utilizadas en una historia de usuario. Así, se reduce el riesgo de errores y se mejora la coherencia de los datos, ya que la información se actualiza dinámicamente conforme cambian las relaciones entre los distintos modelos.

Además, los campos computados relacionales contribuyen a simplificar la lógica de negocio y la experiencia de usuario, ya que presentan información relevante de manera centralizada y siempre actualizada, sin requerir acciones adicionales por parte de los usuarios.

## Diseño y situación actual: Modelos y relaciones

Hasta este punto, el módulo cuenta con los siguientes modelos principales:

- **Tarea**
- **Sprint**
- **Tecnología**

Las relaciones existentes son:

- Una **tarea** está relacionada con un **sprint** mediante una relación `Many2one`.
- Una **tarea** puede estar asociada a varias **tecnologías** mediante una relación `Many2many`.
- Un **sprint** puede contener varias **tareas** (`One2many`).
- Una **tecnología** puede estar asociada a varias **tareas** (`Many2many`).

Además también tenemos campos computados como: 

- El campo `codigo` en **tareas** en generado automáticamente mediante la función `_get_codigo`.
- El campo `fecha_fin` en **sprints**, calculado a partir de la función `_compute_fecha_fin`.

---

## Nuevos modelos y relaciones, permisos y vistas

Vamos a avanzar en la creación de un módulo para gestionar el desarrollo de software mediante una metodología *Scrum*

!!! Note "Metodología ágil scrum" 

    En la metodología **Scrum**, un **proyecto** se organiza en ciclos cortos llamados **sprints**, durante los cuales se desarrollan incrementos funcionales del producto. El trabajo a realizar se descompone en **historias de usuario**, que representan funcionalidades o requisitos desde la perspectiva del usuario final. Cada **historia de usuario** puede dividirse en **tareas** más pequeñas y manejables, que son asignadas al **equipo** durante el **sprint**. 
    
    De este modo, Scrum facilita la planificación iterativa, la adaptación continua y la entrega incremental de valor, asegurando que el equipo mantenga el enfoque en los objetivos del proyecto y pueda responder rápidamente a los cambios en los requisitos.

    Más info: [Flowlu: Guía Paso a Paso para Proyectos y Sprints Ágiles](https://www.flowlu.com/es/blog/project-management/agile-planning-step-by-step-guide-for-agile-projects-and-sprints/)


### Nuevos modelos y sus relaciones

Para enriquecer el modelo de datos y adaptarlo a una gestión de proyectos basada en metodologías ágiles (por ejemplo, Scrum), se añaden los siguientes modelos:

- **Proyecto (`Proyextos`)**: Representa un proyecto de software. Incluye campos como nombre y descripción, y está asociado a un conjunto de historias de usuario.

    - Nombre (`name`)
    - `Descripción`
    - `Historias`: (Se define a continuación qué es una historia). Cada historia tiene un proyecto, pero un proyecto tiene muchas historias. (`One2many`)

!!! Tip "Relación Proyecto - Historias (One2Many)"

                      +-------------------+      (One2many)      +-----------------------+
                      |     Proyecto      |--------------------->|   Historia Usuario    |
                      +-------------------+                      +-----------------------+

    Si tenemos una relación One2many del modelo que estamos definiendo con otro modelo, a la hora de impementar la relación, ponemos lo mismo: One2many

    ```py
    historias = fields.One2many(
        'gestion_tareas_sergio.proyectos_sergio',   # Modelo con el que nos relacionamos
        'proyectos')                                # Campo del modelo que tendrá el Many2one
    ```

- **Historia de Usuario (`Historias`)**: Cada historia de usuario pertenece a un proyecto y puede estar asociada a varias tareas.

    - Nombre (`name`)
    - `descripción`
    - `proyecto`: Relación con proyectos. Un proyecto tiene muchas historias y cada historia tiene un proyecto (`Many2one`)
    - `tareas`: Relación con tareas. Una historia tiene muchos tareas y cada tarea tiene un proyecto (`One2many`)

- **Tareas**: Añadimos la relación inversa con Historias:

    - `historia`: Una tarea pertenece a una historia de usuario y una historia tiene muchas tareas (`Many2one`).


Vamos a crear estas tablas y establecer las relaciones.

??? Example "models.py"

    ```python
    # *******************************************************
    # PROYECTOS
    class proyectos_sergio(models.Model):
        _name = 'gestion_tareas_sergio.proyectos_sergio'
        _description = 'gestion_tareas_sergio.proyectos_sergio'

        name = fields.Char(string="Nombre", readonly=False, required=True, help="Introduzca el nombre del proyecto")
        descripcion = fields.Text(string="Descripción", help="Breve descripción del proyecto")
        historias_om = fields.One2many( 'gestion_tareas_sergio.historias_sergio', 'proyecto_mo', string='Historias de Usuario')



    # *******************************************************
    # HISTORIAS DE USUARIO
    class historias_sergio(models.Model):
        _name = 'gestion_tareas_sergio.historias_sergio'
        _description = 'gestion_tareas_sergio.historias_sergio'

        name = fields.Char(string="Nombre", readonly=False, required=True, help="Introduzca el nombre de la historia de usuario")
        descripcion = fields.Text(string="Descripción", help="Breve descripción de la hisoria de usuario")
        proyecto_mo = fields.Many2one( 'gestion_tareas_sergio.proyectos_sergio', string='Poryecto')
        tareas_om = fields.One2many( 'gestion_tareas_sergio.tareas_sergio', 'historia_mo', string='Tareas')

    # *******************************************************
    # TAREAS
    class tareas_sergio(models.Model):
        _name = 'gestion_tareas_sergio.tareas_sergio'
        _description = 'gestion_tareas_sergio.tareas_sergio'

        name = fields.Char(string="Nombre", readonly=False, required=True, help="Introduzca el nombre")

        historia_mo = fields.Many2one( 'gestion_tareas_sergio.historias_sergio', string='Historias de usuario')
    ```

!!! Nota

    Ya que estamos modificando modelos, vamos a eliminar el campo `fecha_creacion` de los modelos, ya que Odoo crea automáticamente un campo de fecha de creación para cada registro.

### Seguridad, vistas, acciones y menú

Una vez realizadas todas las modificaciones y comprobado que todo funciona correctamente, se deben actualizar los **permisos** para los nuevos modelos y modificar y/o crear vistas de formulario si procede facilitar la gestión de proyectos e historias de usuario. Además, se añaden acciones de ventana y opciones de menú para acceder a estos modelos desde la interfaz de Odoo.

Intenta completar todos los cambios sin ver la solución que se va publicando.... 

??? Example "ir.model.access.csv"

    ```csv
    id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
    . . . 
    access_gestion_tareas_sergio_proyectos_sergio,acceso_proyectos_sergio,model_gestion_tareas_sergio_proyectos_sergio,base.group_user,1,1,1,1
    access_gestion_tareas_sergio_historias_sergio,acceso_historias_sergio,model_gestion_tareas_sergio_historias_sergio,base.group_user,1,1,1,1
    ```


??? Example "views.xml"

    ```xml
    <!-- explicit FORM view definition -->
    <record model="ir.ui.view" id="tareas_form">
      <field name="name">gestion_tareas_sergio.tareas_sergio.form</field>
      <field name="model">gestion_tareas_sergio.tareas_sergio</field>
      <field name="arch" type="xml">
        <form>
          <sheet>
            <group>
              <field name="codigo"/>
              <field name="name"/>
              <field name="descripcion"/>
              <field name="fecha_ini"/>
              <field name="fecha_fin"/>
              <field name="finalizado"/>
              <field name="sprint"/>
              <field name="rel_tecnologias" widget="many2many_tags"/>
              <field name="historia_mo"/>
            </group>
          </sheet>
        </form>
      </field>
    </record>

    <!-- actions opening views on models -->
    <record model="ir.actions.act_window" id="gestion_tareas_sergio.action_proyectos">
      <field name="name">Gestion Tareas Sergio Proyectos</field>
      <field name="res_model">gestion_tareas_sergio.proyectos_sergio</field>
      <field name="view_mode">list,form</field>
    </record>

    <record model="ir.actions.act_window" id="gestion_tareas_sergio.action_historias">
      <field name="name">Gestion Tareas Sergio Historias de Usuarios</field>
      <field name="res_model">gestion_tareas_sergio.historias_sergio</field>
      <field name="view_mode">list,form</field>
    </record>

    <!-- actions -->
    <menuitem name="Proyectos" id="gestion_tareas_sergio.gestion_proyectos" 
      parent="gestion_tareas_sergio.gestion" 
      action="gestion_tareas_sergio.action_proyectos"/>

    <menuitem name="Historias de Usuarios" id="gestion_tareas_sergio.gestion_historias" 
      parent="gestion_tareas_sergio.gestion" 
      action="gestion_tareas_sergio.action_historias"/>

    ```

    Ten en cuenta que en este código se omite la parte que no se ha modificado


## Campos computados relacionales

Una vez ya tenemos ampliado nuestro módulo con nuevos modelos, ahora vamos a platear una realaciones mas avanzadas entre estos modelos.

En concreto vamos a crear vamos a crear dos:

  - Teniendo en cuenta que los sprints son intervalos de tiempo en los que dividimos nuestro proyecto, y que además vamos a tener la precaución de que que en cada momento **solo tenemos activo un sprint**, una tarea puede pertenecer a un sprint en un momento determinado y un tiempo más tarde, si la tarea no se ha finalizado puede pertenecer a otro sprint, esto se puede conseguir calculando en cada momento el sprint que tenemos abierto en un proyecto determinado y las tareas que tenemos asociado a ese proyecto mediante las diferentes historias de usuario.
  - Por otra parte, podemos asociar las diferentes tecnologías que se usan en cada Historia de Usuario teniendo en cuenta que las tecnologías estan asociadas a las tareas, y a su vez, las tareas a las historias. De esta forma, podemos calcular qué tecnologías se utilizan en las diferentes tareas asociadas a cada Historia de Usuario de forma automática.

Veamos cómo implementamos todo esto a continuación.

### Campo computado relacional `Many2one` para Sprint

Se implementa un campo computado que asocia automáticamente una tarea al sprint abierto correspondiente, en función de la fecha actual y el proyecto al que pertenece la historia de usuario.

#### Ejemplo de implementación

```python
class tareas_sergio(models.Model):
    _name = 'gestion_tareas_sergio.tareas_sergio'
    . . .
    sprint_moc = fields.Many2one('gestion_tareas_sergio.sprints_sergio', string='Sprint', compute='_compute_sprint', store=True)

    # @api.depends('codigo')
    @api.depends('fecha_ini', 'fecha_fin', 'historia_mo')
    def _compute_sprint(self):
        for tarea in self:
            tarea.sprint_moc = False
            if tarea.historia_mo and tarea.historia_mo.proyecto_mo:
                sprints = self.env['gestion_tareas_sergio.sprints_sergio'].search([('proyecto_om.id', '=', tarea.historia_mo.proyecto_mo.id)])
                if sprints: 
                    for sprint in sprints:
                        if isinstance(sprint.fecha_fin, datetime) and sprint.fecha_fin > datetime.now():
                            tarea.sprint_moc = sprint.id
```

El campo `sprint_moc` es un campo relacional de tipo `Many2one` que enlaza cada tarea con un sprint del modelo `gestion_tareas_sergio.sprints_sergio`. Este campo es **computado**, lo que significa que su valor se calcula automáticamente mediante el método `_compute_sprint` y se almacena en la base de datos (`store=True`).

El decorador `@api.depends('fecha_ini', 'fecha_fin', 'historia_mo')` indica que el cálculo de `sprint_moc` depende de los campos `fecha_ini`, `fecha_fin` y `historia_mo`. Cuando alguno de estos campos cambia, se ejecuta el método `_compute_sprint`. También podemos decidir que el cálculo dependa del código, con lo cual si cambiamos las fechas de la tarea, no cambiará el sprint. 

Dentro del método, para cada tarea, primero se inicializa `sprint_moc` en `False`. Luego, si la tarea tiene asociada una historia de usuario (`historia_mo`) y esta historia está vinculada a un proyecto (`proyecto_mo`), se buscan todos los sprints relacionados con ese proyecto.

Para cada sprint encontrado, se verifica si la fecha de finalización (`fecha_fin`) es un objeto `datetime` y si es posterior al momento actual. Si se cumple esta condición, se asigna el identificador del sprint a `sprint_moc`. De este modo, la tarea queda asociada automáticamente al sprint activo más próximo, facilitando la organización y planificación de las tareas dentro de los sprints del proyecto.

Este enfoque automatiza la asignación del sprint adecuado a cada tarea, facilitando la gestión y organización del trabajo en función de la planificación temporal de los sprints.

---

### Campo computado relacional `Many2many` para tecnologías usadas

Se crea un campo computado en el modelo de historia de usuario que recopila todas las tecnologías utilizadas en las tareas asociadas a dicha historia.

#### Ejemplo de implementación

```python
class historias_sergio(models.Model):
    _name = 'gestion_tareas_sergio.historias_sergio'
    . . .
    tecnologias_mmc = fields.Many2many( "gestion_tareas_sergio.tecnologias_sergio", compute="_compute_tecnologias", string="Tecnologías")

    def _compute_tecnologias(self):
        for historias in self:
            tecnologias = None
            for tareas in historias.tareas_om:
                if not tecnologias:
                    tecnologias = tareas.tecnologias_mm
                else:
                    tecnologias = tecnologias + tareas.tecnologias_mm
            historias.tecnologias_mmc = tecnologias
```

El campo `tecnologias_mmc` es un campo relacional de tipo `Many2many` que enlaza cada historia con múltiples tecnologías del modelo `gestion_tareas_sergio.tecnologias_sergio`. Este campo es **computado**, es decir, su valor se calcula automáticamente mediante el método `_compute_tecnologias`.

El método `_compute_tecnologias` recorre cada registro de historia y, para cada uno, inicializa la variable `tecnologias` en `None`. Luego, itera sobre todas las tareas asociadas a la historia (`tareas_om`). Para cada tarea, si `tecnologias` aún no tiene valor, se le asignan las tecnologías de la primera tarea; en caso contrario, se suman (concatenan) las tecnologías de la tarea actual a las ya acumuladas. Finalmente, el campo `tecnologias_mmc` de la historia se actualiza con el conjunto total de tecnologías recopiladas de todas sus tareas.

Este enfoque permite que el campo de tecnologías de la historia de usuario refleje automáticamente todas las tecnologías utilizadas en sus tareas asociadas, facilitando la consulta y gestión de la información relacionada.

---

### Related. Campos relacionales no almacenados en base de datos

Hasta ahora, los campos relacionales que hemos tratado han sido de tipo `Many2one`, `One2many` o `Many2many`, y en ocasiones, campos relacionales computados, es decir, calculados automáticamente mediante código Python.

Sin embargo, existe una situación particular en la que podemos necesitar un campo relacional que **no requiera almacenamiento en la base de datos** y que **tampoco sea calculado mediante código Python**. Esto ocurre cuando, en un modelo, queremos añadir un campo relacional que ya existe en otro modelo y simplemente queremos mostrarlo, sin duplicar la información ni crear una nueva relación en la base de datos.

#### Ejemplo de implementación

Supongamos que tenemos el siguiente escenario:

- El modelo **Proyecto** está relacionado con las **Historias de Usuario**.
- Cada **Historia de Usuario** pertenece a un **Proyecto** y está asociada a un conjunto de **Tareas**.
- Las **Tareas** están relacionadas con un **Sprint**, con las **Tecnologías** utilizadas y con la **Historia de Usuario** a la que pertenecen.

Ahora, imaginemos que queremos que, en el formulario de **Tarea**, además de la historia de usuario, se muestre también el **Proyecto** al que pertenece la tarea. Podríamos crear un campo `Many2one` directamente en el modelo de tareas, pero esto generaría una columna adicional en la base de datos, lo cual no es necesario, ya que la relación ya existe a través de la historia de usuario.

Para resolver esto, Odoo permite definir un campo relacional utilizando el atributo `related`. De este modo, el campo en el modelo de tareas simplemente referencia el campo `proyecto` del modelo de historia de usuario, sin necesidad de almacenamiento adicional ni lógica de cálculo.

Por ejemplo:

```python
class tareas_sergio(models.Model):
  _name = 'gestion_tareas_sergio.tareas_sergio'
  # ...
  historia_mo = fields.Many2one( 'gestion_tareas_sergio.historias_sergio', string='Historias de usuario')
  # Nuevo campo que relaciona directamente con el proyecto a partir de la historia de usuario
  proyecto_no = fields.Many2one(
      'gestion_tareas_sergio.proyectos_sergio',
      string='Proyecto',
      related='historia_mo.proyecto_mo',
      readonly=True)
```

En este caso, el campo `proyecto_no` en el modelo de tareas obtiene automáticamente el proyecto asociado a la historia de usuario seleccionada. Además, al establecer `readonly=True`, evitamos que el usuario pueda modificar este valor manualmente.

Para que este campo sea visible en el formulario de tareas, basta con añadirlo a la vista correspondiente:

```xml
<field name="proyecto_no"/>
```

De este modo, cuando se seleccione una historia de usuario en una tarea, el campo de proyecto se actualizará automáticamente mostrando el valor correspondiente, sin que el usuario pueda modificarlo.

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/17_EjemploTarelaRelatedProyecto.png){ width="75%"  }
  <figcaption>Asignación automática de proyecto</figcaption>
</figure>

Este tipo de campos relacionales son útiles cuando queremos mostrar información relacionada sin duplicar datos ni realizar cálculos adicionales. Utilizando el atributo `related`, podemos mantener la integridad y simplicidad del modelo de datos, facilitando la navegación y consulta de información en la interfaz de Odoo.








## Conclusiones

En este apartado se ha mostrado cómo crear campos computados relacionales en Odoo, tanto para relaciones `Many2one` como `Many2many`. Además, se ha ampliado la estructura del módulo para incluir proyectos e historias de usuario, permitiendo una gestión más completa y alineada con metodologías ágiles como Scrum.

La estructura resultante permite:

- Gestionar múltiples proyectos, cada uno con sus historias de usuario.
- Desglosar historias de usuario en tareas, que a su vez se asocian a sprints y tecnologías.
- Automatizar la asignación de sprints y la recopilación de tecnologías mediante campos computados relacionales.

Este enfoque facilita la escalabilidad y el mantenimiento del módulo, y sienta las bases para futuras ampliaciones, como la gestión de usuarios y permisos avanzados.

