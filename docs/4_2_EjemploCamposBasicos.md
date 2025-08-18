---
title: 4.2. Campos básicos y relacionales
description: Ejemplo guiado de desarrollo de módulo de gestión de proyectos en Odoo
---

Seguimos con el ejemplo creado en la sección anterior y en este apartado veremos cómo añadir distintos [campos básicos y relaciones](https://www.odoo.com/documentation/18.0/es/applications/studio/fields.html) a nuestros modelos en Odoo:. No nos centraremos en vistas avanzadas, sino en ver la relación entre modelos y cómo Odoo toma vistas por defecto, además de cómo podemos modificarlas un poquito.

Tenemos un modelo `tareas_sergio` con dos campos propios y dos campos de tipo carácter llamado `name` y `descripcion`. 

!!! Note "Documentación de desarrollo ORM"

    Puedes consultar la [documentación de desarrollo de Odoo](https://www.odoo.com/documentation/18.0/developer/reference/orm.html) para más información sobre el ORM y los campos disponibles.

## Campos básicos y parámetros

Al crear un campo de tipo carácter, puedes añadir parámetros como:

- **label**: Si no se indica, la etiqueta será el nombre del campo.
- **readonly**: Para hacerlo de solo lectura.
- **required**: Para hacerlo obligatorio.
- **help**: Mensaje de ayuda, por ejemplo: "Introduzca el nombre".

Cuando modificas el modelo, debes reiniciar Docker y actualizar el módulo para aplicar los cambios.

### Ejemplo de campos básicos

Supongamos que añadimos los siguientes campos al modelo `tareas_sergio`, pero añadiendo etiquetas, ayuda y haciendo que algún campo sea requerido:

- `creation_date`: tipo `Date`
- `start_date`: tipo `Datetime`
- `end_date`: tipo `Datetime`
- `is_done`: tipo `Boolean`

Vamos a añadir estos campos, si es necesario revisando la documentación de Odoo sobre los [tipos de campos](https://www.odoo.com/documentation/18.0/es/developer/reference/backend/orm.html#basic-fields). Añadimos también características indicadas y modificamos los campos ya existente con estos parámetros


??? Note "models.py"


    ```py
    from odoo import models, fields, api
    
    class tareas_sergio(models.Model):
        _name = 'gestion_tareas_sergio.tareas_sergio'
        _description = 'gestion_tareas_sergio.tareas_sergio'

        name = fields.Char(string="Nombre", readonly=False, required=True, help="Introduzca el nombre")
        descripcion = fields.Text(string="Descripción", help="Breve descripción de la tarea")
        fecha_creacion = fields.Date(string="Fecha Creación Tarea", readonly=False, required=True, help="Fecha en la que se dió de alta la tarea")
        fecha_ini = fields.Datetime(string="Fecha Inicio", readonly=False, required=True, help="Fecha en la que se inicia la tarea")
        fecha_fin = fields.Datetime(string="Fecha Final", readonly=False, required=False, help="Fecha en la que se finaliza la tarea")
        finalizado = fields.Boolean(string="Finalizado", readonly=False, required=False, help="Indica si la tarea ha sido finalizada o no")
    ```

Una vez añadidos, comprobamos que todo funciona bien reinstalando el módulo. Recuerda reiniciar el servicio y actualizar el módulo tras modificar los modelos.

Actualizamos y comprobamos que tenemos los nuevos campos. Comprobamos que sale la ayuda y que los campos requeridos funcionan como tal.

### Actualizando la vista en Odoo

Por defecto, Odoo crea una vista de tipo formulario con todos los campos. 

Si entramos en esta vista, veremos que es muy pobre y que normalmente no esta ajustada de forma coherente. Para personalizar la vista, vamos a editar el archivo de vistas *XML* y añade los campos dentro de una etiqueta `<group>`.

Así pues, para crear la vista, **vamos a copiar y pegar** la vista que ya tenemos en formato **`list`** y la vamos a renombrar como tipo **`form`** y donde tenemos las etiquetas `<list>` vamos a tener `<form>`, `<sheet>` y `<group>` 

Observa el siguiente código y verás que no todos los campos aparecen en las definiciones. Si todo esta correcto, este ejemplo en el formato listado, aparecerán 3 campos y en el formato formulario aparecerán 5 campos.

Como siempre el nombre del modelo puede ser el que quieras, pero se recomienda seguir una misma nomenclatura durante todo el proyecto y el modelo, debemos indicar correctamente el nombre del modelo definido. Si nos equivocamos en el nombre del modelo, mostrará los formularios por defecto

```xml
<!-- explicit list view definition -->
  <record model="ir.ui.view" id="tareas_list">
  <field name="name">gestion_tareas_sergio.tareas_sergio.list</field>
  <field name="model">gestion_tareas_sergio.tareas_sergio</field>
  <field name="arch" type="xml">
    <list>
      <field name="name"/>
      <field name="descripcion"/>
      <field name="finalizado"/>
    </list>
  </field>
</record>

<!-- explicit FORM view definition -->
<record model="ir.ui.view" id="tareas_form">
  <field name="name">gestion_tareas_sergio.tareas_sergio.form</field>
  <field name="model">gestion_tareas_sergio.tareas_sergio</field>
  <field name="arch" type="xml">
    <form>
      <sheet>
        <group>
          <field name="name"/>
          <field name="descripcion"/>
          <field name="fecha_creacion"/>
          <field name="fecha_ini"/>
          <!-- <field name="fecha_fin"/> -->
          <field name="finalizado"/>
        </group>
      </sheet>
    </form>
  </field>
</record>
```

Comprueba que todo funciona correctamente y que puedes introducir valores en todos los campos.

## Relaciones

Las relaciones entre los modelos (en definitiva, entre las tablas de la base de datos) también se simplifican gracias al **ORM**. De esta manera, las relaciones uno a muchos se implementan en Odoo mediante el campo **`Many2one`** y las relaciones muchos a muchos mediante **`Many2many`**. Las relaciones muchos a muchos, en una base de datos relacional, implican una tercera tabla intermedia, pero en Odoo no tenemos que preocuparnos por estos detalles si no queremos, ya que el mapeo de objetos lo detectará y creará las tablas, claves y restricciones de integridad necesarias. 

### Campos relacionales: Many2one

Vamos a comprobar cómo podemos establecer una relación **`Many2one`**, para ello vamos a crear un nuevo modelo llamado **sprint** que después relacionaremos con las tareas.

!!! Nota "Sprints y tareas"

    Un *sprint* es un ciclo de trabajo corto y repetitivo, típicamente de 1 a 4 semanas, en el que:

    - Se planifican tareas específicas (historias de usuario, bugs, mejoras).
    - Se ejecutan esas tareas con un objetivo claro.
    - Se revisa el trabajo realizado al final del sprint.
    - Se reflexiona sobre cómo mejorar en el siguiente sprint (retrospectiva).

    Un *sprint* contiene varias *tareas* (también llamadas issues, tickets o historias de usuario) que son las  unidades de trabajo que deben completarse dentro del sprint.

Así pues, vamos a crear un nuevo modelo `sprint` con los siguientes campos:

- nombre
- descripción
- fecha_inicio
- fecha_fin

Intenta generar el código, es muy sencillo, puesto que la mayoría de los campos los tenemos en el modelo de tareas, y antes de introducir la relación, asegura que todo es correcto reiniciando y reinstalando.

??? Note "modelo sprint en models.py"

```py
class spints_sergio(models.Model):
    _name = 'gestion_tareas_sergio.sprints_sergio'
    _description = 'gestion_tareas_sergio.sprints_sergio'

    name = fields.Char(string="Nombre", readonly=False, required=True, help="Introduzca el nombre")
    descripcion = fields.Text(string="Descripción", help="Breve descripción de la tarea")
    fecha_ini = fields.Datetime(string="Fecha Inicio", readonly=False, required=True, help="Fecha en la que se inicia la tarea")
    fecha_fin = fields.Datetime(string="Fecha Final", readonly=False, required=False, help="Fecha en la que se finaliza la tarea")
```

Una vez tenemos el nuevo modelo, para relacionar tareas con sprints, añadimos un campo `sprint_id` en el modelo `tareas_sergio` de tipo `Many2one` de la siguiente forma:

```python
sprint = fields.Many2one('gestion_tareas_sergio.sprints_sergio', string='Sprint relacionado', ondelete='set null', help='Sprint relacionado')
```

  Este nuevo campo significa:

  - `sprint`: es el nombre del campo en el modelo actual (`tareas_sergio`).
  - `fields.Many2one(...)`: define una relación de muchos a uno con otro modelo.
  - `'gestion_tareas_sergio.sprints_sergio'`: **indica el modelo** destino de la relación (en este caso, el modelo de sprints).
  - `string='Sprint relacionado'`: etiqueta que se mostrará en la interfaz de usuario para este campo.
  - `ondelete='set null'`: si se elimina el sprint relacionado, este campo se establecerá a `null` (la tarea no se elimina, solo pierde la relación). Otros valores podrían ser: `cascade`, `restrict` (Impide eliminar el registro relacionado si hay registros que lo usan), `set default` y `no action`	(No hace nada automáticamente. Puede causar errores si hay restricciones SQL). 
  - `help='Sprint relacionado'`: texto de ayuda que aparece al pasar el cursor sobre el campo.

De esta manera, con esta simple líneas por medio de **ORM** le estamos informado a Odoo de la relación entre ambos modelos.

Es el momento de probar que todo funciona bien aunque para ello como mínimo debemos:

- Añadir el nuevo campo en la vista formulario, por ejemplo
- Añadir nnuevos permisos correspondientes en el archivo de seguridad para poder acceder al nuevo modelo sprints

??? Note "ir.model.access.csv"

    ```csv
    id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
    access_gestion_tareas_sergio_tareas_sergio,acceso_tareas_sergio,model_gestion_tareas_sergio_tareas_sergio,base.group_user,1,1,1,1
    access_gestion_tareas_sergio_sprints_sergio,acceso_sprints_sergio,model_gestion_tareas_sergio_sprints_sergio,base.group_user,1,1,1,1
    ```


Probamos y observamos que ya tenemos el nuevo campo, ya podemos seleccionar e incluso crear nuevos *sprints* desde aqui aunque no tenemos vistas del modelo.

Vamos a configurar para que aparezca en el menú el nuevo modelo al igual que lo hace el modelo de tareas observaremos que si no definimos vistas entonces Odoo mostrará las vistas por defecto, y si quieremos cambiar algo en estas vistas, simplemetne las definimos..


### Campos relacionales: One2many

La relación **One2many** (uno a muchos) es un concepto fundamental en bases de datos y frameworks como Odoo. Este tipo de relación indica que un registro de un modelo puede estar vinculado a múltiples registros de otro modelo. Por ejemplo, una factura (modelo padre) puede tener varias líneas de factura (modelo hijo). En la práctica, esto nos permite a partir de un registro detalle, obtener todos los registros que apunta a este., o sea, que se crea un un campo especial que permite asociar una lista de registros hijos al registro padre.

En Odoo, el campo `One2many` se define especificando el modelo relacionado y el campo inverso que conecta ambos modelos. Esto facilita la navegación y gestión de datos relacionados, permitiendo mostrar, agregar o eliminar registros hijos directamente desde el formulario del registro padre. Esta relación es clave para estructurar datos complejos y mantener la integridad referencial dentro de la aplicación.

Vamos a crear la relación inversa a la creada anteriormente, mediante la creación de un nuevo campo en el modelo `sprint` que se llame por ejemplo `tareas` que será de tipo  `One2many` como sigue:

```python
tareas = fields.One2many('gestion_tareas_sergio.tareas_sergio', 'sprins', string='Tareas')
```

La línea anterior define un campo llamado `tareas`  de tipo `One2many`, lo que significa que permite asociar múltiples registros del modelo `'gestion_tareas_sergio.tareas_sergio'`, utilizando el campo inverso `'sprins'` en el modelo de tareas para establecer la relación. El campo inverso es necesario e imprescidible y es el campo que ha creado la relación `Many2one` en el modelo indicado.

El parámetro `string='Tareas'` especifica la etiqueta que se mostrará en la interfaz de usuario para este campo. 

Gracias a esta definición, desde el formulario del modelo principal (por ejemplo, un sprint), se pueden visualizar y gestionar todas las tareas relacionadas, facilitando la administración de la información y la navegación entre registros vinculados.

Veamos a continuación un ejemplo de uso de esta relación: 

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/13_EjemploOne2Many.png){ width="75%"  }
  <figcaption>Ejemplo de relación One2many</figcaption>
</figure>


### Campos relacionales: Many2many

Los **campos relacionales Many2many** permiten establecer una relación de muchos a muchos entre dos modelos en una base de datos o framework como Odoo. Esto significa que un registro de un modelo puede estar vinculado a varios registros de otro modelo, y viceversa. Por ejemplo, un estudiante puede estar inscrito en varios cursos, y cada curso puede tener varios estudiantes asociados.

En Odoo, el campo `Many2many` se utiliza para crear este tipo de relación, facilitando la gestión y visualización de datos complejos donde la asociación entre registros es múltiple en ambos sentidos. Este campo suele representarse en la interfaz de usuario mediante listas o tablas donde se pueden seleccionar y gestionar fácilmente los registros relacionados.

Para nuestro utilizar este tipo de relaciones en nuestro ejemplo, supongamos que definimos un modelo que sea **Tecnologías** de forma que podamos establecer un listado de las diferentes tecnologías que pueden aplicarse a un proyecto, por ejemplo *python*, *java*, *javascript*, etc... De esta forma un proyecto puede implementarse usando muchas tecnologías y cada tecnología puede ser usada en muchos proyectos.

Vamos pues a crar un modelo **"Tecnología"** que tenga los siguientes campos:

- nombre
- descripción
- logo: donde incluiremos el logotipo de la tecnología, y así comprobamos como funciona un tampo de tipo imagen. Busca información y crea un campo imagen de 200x200 pixel máximo.

??? Note "nuevo modelo tecnología en models.py"

    ```py
    class tecnologias_sergio(models.Model):
      _name = 'gestion_tareas_sergio.tecnologias_sergio'
      _description = 'gestion_tareas_sergio.tecnologias_sergio'

      name = fields.Char(string="Nombre", readonly=False, required=True, help="Introduzca el nombre de la tecnología")
      descripcion = fields.Text(string="Descripción", help="Breve descripción de la tecnología")
      logo = fields.image(string="Logo", help="Logo de la tecnología", max_width=256, max_height=256)
    ```

y ahora creamos campos en tareas y tecnologías para generar la relación **muchos a muchos**.

Para asociar tareas y tecnologías, añadiimos en `tareas`:

```python
    rel_tecnologias = fields.Many2many( 
        comodel_name='gestion_tareas_sergio.tecnologias_sergio',
        relation='relacion_tareas_tecnologias',
        column1='rel_tareas',
        column2='rel_tecnologias',
        string='Tecnologías')

```

donde se define un campo llamado `rel_tecnologias` utilizando el tipo de campo `Many2many` siendo los parametros utilizados los siguientes:

- **comodel_name**: indica el modelo con el que se realiza la relación muchos a muchos, en nuestro caso `'gestion_tareas_sergio.tecnologias_sergio'` .
- **relation**, `'relacion_tareas_tecnologias'`, indica el nombre de la tabla intermedia que se utilizará en la base de datos para gestionar la relación entre tareas y tecnologías. 
- **column1** y **column1**: `'rel_tareas'` y `'rel_tecnologias'`, especifican los nombres de las columnas en la tabla intermedia que referencian a cada uno de los modelos relacionados.
- El parámetro `string='Tecnologías'` define la etiqueta que se mostrará en la interfaz de usuario para este campo. 

Para comprobar que funciona, realizamos los pasos de siempre:

- Incluimos el nuevo modelo en el fichero de seguridad
- Añadimos el nuevo campo en la vista
- Añadimos un menú y un action para poder acceder directamente. 
- Reiniciamos y actualizamos.

!!! Tip "widgets"

    Al añadir el campo en la vista, podemos hacerlo de forma habitual

    ```py
    <field name="rel_tecnologias"/>
    ```

    o podemos indicar que queremos un [*widget*](https://www.odoo.com/documentation/18.0/es/applications/studio/fields.html?highlight=widgets#many2many-many2many) especifico de visualización, en este uno que nos permite ver cada tecnología como un *tag*

    
    ```py
    <field name="rel_tecnologias" widget="many2many_tags"/>
    ```

    Comprueba la diferencia entre ambos

    **PONER ENLACE DE WIDGETS A LOS APUNTES DE CASTILLO QUE EXPLICAN MUY BIEN LOS WIDGETS**

Y en `tecnologías` hacemos exactamente lo mismo, pero apuntanto a tareas:

```python
    rel_tareas = fields.Many2many( 
        comodel_name='gestion_tareas_sergio.tareas_sergio',
        relation_name='relacion_tareas_tecnologias',
        column1='rel_tecnologias',
        column2='rel_tareas',
        string='Tareas')
```

Gracias a esta configuración, es posible seleccionar y gestionar fácilmente las tecnologías asociadas a una tarea desde el formulario correspondiente, permitiendo una administración flexible y eficiente de las relaciones entre tareas y tecnologías.


