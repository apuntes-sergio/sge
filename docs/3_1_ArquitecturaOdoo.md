---
title: 3.1. Arquitectura de Odoo
description: Pasos para la instalación y configuración de Odoo
---

Antes de comenzar ha desarrollar en ***Odoo*** necesitamos conocer un poco de su arquitectura para comprender cómo trabaja y la forma en la que debemos programar y cómo poder utilizar el framework que nos facilitar para poder implementar nuestros propios módulos

# Arquitectura


***Odoo*** es un ERP con una arquitectura [de Tenencia Múltiple](http://es.wikipedia.org/wiki/Tenencia_M%C3%BAltiple). Es decir, tiene una base de datos y un servidor común para todos los clientes. Lo contrario sería tener un servidor o base de datos por cliente o virtualizar.

El *framework* de ***Odoo*** facilita diversos componentes que permiten construir la aplicación:

-   La capa **ORM** (Object Relational Mapping) entre los objetos Python y la base de datos **PostgreSQL**. El diseñador-programador no realiza el diseño de la base de datos; únicamente diseña clases, y la capa *ORM* de ***Odoo*** se encarga de mapearlas sobre el SGBD **PostgreSQL**. Mas info en [An overview of ORM Methods in Odoo](https://sdlccorp.com/post/an-overview-of-orm-methods-in-odoo-18/)

<figure markdown="span" align="center">
  ![Image title](./imgs/desarrollo/odoo_ORM.png){ width="85%"  }
  <figcaption>Diagrama ***Odoo*** ORM (Object Relational Mapping)</figcaption>
</figure>

-   Una arquitectura **MVC** ([modelo-vista-controlador](http://es.wikipedia.org/wiki/Modelo_Vista_Controlador)), en la que el ***modelo*** reside en los datos de las clases diseñadas con Python, la ***vista*** reside en los formularios, listas, calendarios, gráficos... definidos en archivos *XML*, y el ***controlador*** reside en los métodos definidos en las clases que proporcionan la lógica de negocio.

<figure markdown="span" align="center">
  ![Image title](./imgs/desarrollo/Diagrama_MVC.jpg){ width="65%"  }
  <figcaption>Diagrama Modelo - Vista - Controlador.</figcaption>
</figure>

-   Diseñadores de informes.
-   Facilidades para la traducción de la aplicación a varios idiomas.

El **servidor** ***Odoo*** proporciona acceso a la base de datos mediante **ORM**. El servidor necesita tener instalados **módulos**, ya que comienza vacío.

Por otro lado, el **cliente** se comunica con el servidor mediante **XML-RPC**, y los clientes web por **JSON-RPC**. El cliente solo tiene que mostrar lo que proporciona el servidor o solicitar correctamente los datos. Por tanto, un cliente puede ser muy simple y desarrollarse en cualquier lenguaje de programación. ***Odoo*** proporciona un cliente web, aunque se puede crear un cliente en cualquier plataforma.

Los datos están almacenados en una **base de datos relacional** como es *PostgreSQL* y gracias al **ORM**, no es necesario realizar consultas SQL directamente. El **ORM** proporciona una serie de métodos para trabajar de manera más rápida y segura. En lugar de hablar de tablas, se habla de **modelos**. Estos son "mapeados" por el ORM en tablas. Sin embargo, un modelo es más que datos en una tabla. Un modelo se comporta como un objeto al tener campos funcionales, restricciones y campos relacionales que dejan la normalización de la base de datos en manos de ***Odoo***.

El acceso del cliente a los datos se realiza mediante un servicio. Este puede ser **[WSGI](http://en.wikipedia.org/wiki/Web_Server_Gateway_Interface)**. WSGI es una solución estándar para crear servidores y clientes HTTP en Python. En el caso de Odoo, este utiliza `Werkzeug`, que interpreta las peticiones y las transforma en funciones que ***Odoo*** puede entender.

Otro concepto dentro de ***Odoo*** son los **Business Objects**. Se implementan con `models` y contienen tanto datos como lógica de negocio.

Odoo también proporciona los llamados **Wizards**, que funcionan como asistentes para introducir datos de una manera más sencilla para el usuario.

El cliente web es fácil de desarrollar gracias a los **Widgets** o Window GaDGETS. Estos proporcionan un comportamiento y visualización adecuados para cada tipo de dato. Por ejemplo, si el campo es para definir una fecha, mostrará un calendario. Algunos tienen diferentes visualizaciones según el tipo de vista y se pueden definir Widgets personalizados. Los Widgets y demás interfaz web funcionan gracias a `OWL`, un framework de cliente web desarrollado en el proyecto `Odoo` y similar a `React`.

Resumiendo todos estos conceptos: 

| Componente         | Descripción |
|------|-----------------|
| Base de Datos (PostgreSQL)   | Donde se almacenan los datos de forma relacional.   |
| ORM   | Traduce modelos Python a tablas SQL, gestionando relaciones y lógica.      |
| Business Objects    | Modelos que contienen datos y lógica de negocio.                           |
| WSGI/Werkzeug      | Intermediario que convierte peticiones HTTP en funciones de ***Odoo***.          |
| Wizards   | Asistentes que facilitan la introducción de datos.                         |
| Cliente Web (OWL + Widgets)    | Interfaz interactiva que adapta la visualización según el tipo de dato.    |

## La base de datos de Odoo

En ***Odoo*** no existe un diseño explícito de la base de datos, si no que la base de datos de una empresa en ***Odoo*** es el resultado del mapeo del diseño de clases del ERP hacia el SGBD PostgreSQL, que es el que proporciona la persistencia necesaria para los objetos. Esto es el **ORM**.

En consecuencia, ***Odoo*** no facilita ningún diseño entidad-relación sobre la base de datos de una empresa ni tampoco ningún diagrama del modelo relacional.

***Odoo*** permite, mediante el cliente web, recuperar el nombre de la clase Python que define la información que se introduce a través de un formulario y el nombre del dato miembro de la clase correspondiente a cada campo del formulario. Esta información permite llegar a la tabla y columna afectadas, teniendo en cuenta dos cuestiones:

- El nombre de las clases Python de ***Odoo*** siempre está en minúsculas (se utiliza el guion bajo para hacer legibles las palabras compuestas) y sigue la nomenclatura `nombre_del_modulo.nom1.nom2.nom3...` en la cual se utiliza el punto para indicar un cierto nivel de jerarquía. Cada clase Python de ***Odoo*** se mapea en una tabla de *PostgreSQL* con muchas posibilidades de que su nombre coincida con el de la clase, sustituyendo los puntos por guiones bajos.

- Los nombres de los atributos de una clase Python también están siempre en minúsculas (se utiliza el guion bajo para hacer legibles las palabras compuestas). Cada dato miembro de una clase Python de ***Odoo*** que sea persistente (una clase puede tener datos miembros calculados no persistentes) se mapea como un atributo en la correspondiente tabla de PostgreSQL con el mismo nombre.

!!! Ejemplo

    La clase Python `sale.order` de ***Odoo*** está pensada para describir la cabecera de los pedidos de venta y la correspondiente tabla en PostgreSQL es `sale_order`.

De esta manera, conociendo el nombre de la clase y el nombre del dato miembro, es muy posible conocer el nombre de la tabla y de la columna correspondientes. Se puede configurar el cliente web para que informe del nombre de la clase y del dato miembro al situar el ratón sobre las etiquetas de los campos de los formularios.


## Estructura de carpetas de Odoo

La estructura de carpetas está organizada para facilitar el desarrollo, mantenimiento y extensión de funcionalidades. 

De esta forma, a continuación tenemos el listado de las carpetas más importantes dentro del código fuente de Odoo:

- **`addons/`** : Contiene todos los módulos estándar y personalizados de Odoo. Cada módulo tiene su propia carpeta con modelos, vistas, controladores, datos, etc. Ejemplos: `sale`, `account`, `crm`, `inventory`.

- **`config/`**: Contiene archivos de configuración del servidor Odoo, como `odoo.conf`. Es esencial para iniciar Odoo correctamente en entornos de producción o desarrollo.

- **`data/`**: Carpeta opcional usada para almacenar datos persistentes o archivos cargados por el usuario. Aunque no es parte del core, es común en despliegues personalizados o cuando se configura `filestore` fuera de la base de datos.

- `odoo/`:  Núcleo del framework. Aquí se encuentran los componentes esenciales como el ORM, controladores, herramientas de seguridad, y utilidades. Subcarpetas importantes:
     * `odoo/models/`: Definición del ORM y clases base.
     * `odoo/addons/`: Módulos integrados directamente en el core.
     * `odoo/tools/`: Funciones auxiliares (utilidades, validaciones, etc.).
     * `odoo/http.py`: Manejo de peticiones HTTP y controladores web.

- `setup/`: Archivos de configuración para la instalación del entorno de desarrollo y despliegue.

- `tests/`: Pruebas automatizadas para asegurar la calidad del código. Incluye pruebas unitarias y de integración.

- `doc/`: Documentación técnica del framework y sus componentes.

- `static/`:  Archivos estáticos como CSS, JS e imágenes utilizados por la interfaz web.

- `web/`: Módulo base para la interfaz web de Odoo. Incluye componentes JavaScript, vistas web, y widgets.

- `custom_addons/` *(opcional)*: Carpeta comúnmente usada para módulos personalizados fuera del core. Se configura en el archivo de inicio (`odoo.conf`).

Si observamos, las tres primera carpetas son las que hemos definido en el `docker-compose.yml` del punto anterior, por lo que ahora ya conocemos su cometido: 

```yml
services:
  odoo:
    container_name: odoo
    image: odoo:18.0
    depends_on:
      - db
    ports:
      - "8069:8069"
    volumes:
      - odoo-web-data:/var/lib/odoo
      - ./config:/etc/odoo
      - ./addons:/mnt/extra-addons
    environment:
      - HOST=db
      - USER=odoo
      - PASSWORD=odoo
    command: --dev=all
    tty: true
```


## Módulos

Tanto el servidor como los clientes son módulos. Todos están guardados en una base de datos. Todo lo que se pueda hacer para modificar ***Odoo*** se hace en módulos.

### Composición de un módulo

Los módulos de Odoo amplían o modifican partes del *Modelo-Vista-Controlador*. De esta manera, un módulo puede tener:

-   **Objetos de negocio**: Son la parte del modelo, definidos en clases de Python según una sintaxis propia del ORM de ***Odoo***.
-   **Archivos de datos**: Son archivos XML que pueden definir datos, vistas o configuraciones.
-   **Controladores web**: Gestionan las peticiones de los navegadores web.
-   **Datos estáticos**: Imágenes, CSS o JavaScript utilizados por la interfaz web. Es necesario que los datos estáticos se guarden en el directorio **static**. Por ejemplo, el icono del módulo va en `static/description/icon.png`.

### Estructura de archivos de un módulo

-   Todos los módulos están en un directorio definido en la opción **--addons-path** o en el archivo de configuración, normalmente `addons`. Cada módulo esta un un subdirectorio dentro de esta carpeta y por lo que puede haber más de un directorio.
-   Un módulo de Python se declara en un archivo de **manifest** que da información sobre el módulo, lo que hace, los módulos de los que depende y cómo se debe instalar o actualizar. [Manifest in Odoo](https://www.odoo.com/documentation/8.0/reference/module.html#reference-module-manifest)
-   Un módulo es un paquete de Python que necesita un archivo **\_\_init\_\_.py** para instanciar todos los archivos Python.

### Creación de módulos

Para ayudar al programador, ***Odoo*** contiene un comando para crear módulos vacíos. Este crea la estructura de archivos necesaria para empezar a trabajar:

     $ odoo scaffold <nombre_del_módulo> <dónde_crearlo>

Posteriormente veremos más sobre `scaffold`, pero si quieres más información puedes recurrir al [Manual oficial Scaffolding](https://www.odoo.com/documentation/8.0/reference/cmdline.html#scaffolding)

El parámetro **scaffold** puede tener la opción **-t** para indicar el directorio de la *plantilla*. Esta está hecha utilizando **jinja2**, que es un motor de plantillas para Python.

Las *plantillas* están en el directorio de instalación de Odoo, en el directorio **cli**. En nuestro caso: **cli/templates/** dentro del directorio de instalación de ***Odoo***.

Podemos crear una *plantilla* copiando el directorio default o theme y modificando los archivos. Esto puede ser útil si siempre hacemos módulos con la misma plantilla. Por ejemplo, para poner nuestro logo, copyright, etc.

## ORM

***Odoo*** mapea sus objetos en una base de datos mediante **ORM**, evitando al programador la mayoría de consultas SQL. De esta manera, el desarrollo de los módulos es muy rápido y se evitan errores de programación.

Los modelos se crean como clases de Python que extienden la clase **[models.Model](https://www.odoo.com/documentation/master/developer/reference/backend/orm.html)**, que contiene los campos y métodos útiles para trabajar con el ORM.

!!! Nota

    Los modelos, al heredar de **models.Model**, necesitan obligatoriamente asignar valores a algunas variables, como por ejemplo **\_name**.

***Odoo*** considera que un modelo es la referencia a una o más tablas en la base de datos. Un modelo no es una fila en la tabla, es toda la tabla.

> En programación, el **Modelo** es una forma de relacionar el programa con la base de datos. Es de más alto nivel que las consultas directas a la base de datos y que las **clases y objetos** respecto a la programación orientada a objetos. El modelo une en un único concepto las **estructuras de datos**, las **restricciones de integridad** y las opciones de **manipulación** de los datos.

Los modelos en ***Odoo*** pueden

 - Heredar de **models.Model** y ser los normales, mapeados y permanentes en la base de datos. 
 - Pueden ser **models.TransientModel**, que son iguales pero no tienen persistencia definitiva en la base de datos. Estos son los recomendados para crear `wizards`. 
 - También pueden ser **models.AbstractModel** para definir modelos abstractos que luego serán heredados.

En Odoo, existen tres tipos principales de modelos que heredan de clases base diferentes según su propósito, se trata de `models.Model`, `models.TransientModel` y `models.AbstractModel`:

- `models.Model` se trata de Modelos persistentes. Representa una tabla real en la base de datos. Los datos almacenados en este modelo son permanentes. Por ejemplo Clientes, productos, facturas, etc.
- `models.TransientModel` son datos temporales (como asistentes o formularios rápidos), por lo tanto es un modelo no persistente. Los registros se eliminan automáticamente después de un tiempo (por defecto, 7 días). Se utilizan para Wizards, asistentes de importación/exportación, formularios temporales.
- `models.AbstractModel` que son modelos base reutilizables, o sea, se trata de modelos abstractos que no crea una tabla en la base de datos. Se utiliza para definir funcionalidades comunes que pueden ser heredadas por otros modelos.

Comparativa rápida

| Característica         | `models.Model` | `models.TransientModel` | `models.AbstractModel` |
|------------------------|----------------|--------------------------|-------------------------|
| Persistencia           | Sí             | No (temporal)            | No                      |
| Crea tabla en BD       | Sí             | Sí                       | No                      |
| Uso típico             | Datos reales   | Formularios/Wizards      | Funcionalidad común     |
| Eliminación automática | No             | Sí                       | No                      |

### Inspeccionar los modelos

Para ver los modelos existentes, se puede acceder a la base de datos PostgreSQL o consultar en *Configuración > Estructura de la base de datos > Modelos* dentro del modo desarrollador.

Cabe destacar el campo *modules*, donde se indican los módulos instalados en los que se define o hereda el modelo observado.

### Fields

Las "columnas" del modelo son los fields (campos). Estos pueden ser de datos normales como Integer, Float, Boolean, Date, Char... o especiales como Many2one, One2many, Related...

Hay algunos fields reservados:

- **id** (Id): identificador único para un registro en su modelo.
- **create_date** (Datetime): fecha de creación del registro.
- **create_uid** (Many2one): usuario que creó el registro.
- **write_date** (Datetime): fecha de la última modificación del registro.
- **write_uid** (Many2one): usuario que realizó la última modificación.

Hay otros fields que podemos declarar y que tienen propiedades especiales. Estos son los más importantes:

- **name**: es el campo utilizado para el **Identificador Externo** o cuando se hace referencia en los many2one en la vista.
- **active**: indica si el registro está activo. Permite ocultar productos que ya no se necesitan, por ejemplo.
- **sequence**: permite definir el orden de los registros a mostrar en una lista.

Los fields se declaran con un constructor:

```python
from openerp import models, fields

class LessMinimalModel(models.Model):
    _name = 'test.model2'

    name = fields.Char()
```

Tienen unos atributos comunes:

- **string** (unicode, por defecto: el nombre del field): la etiqueta que verán los usuarios en la vista.
- **required** (bool, por defecto: False): si es *True*, el campo no puede quedar vacío.
- **help** (unicode, por defecto: ''): en los formularios proporciona ayuda al usuario para rellenar el campo.
- **index** (bool, por defecto: False): pide a ***Odoo*** que sea el índice de la base de datos. En otro caso, el ORM crea un campo **id**.

Y algunos, sobre todo los especiales, tienen atributos particulares.

Ejemplo completo:

```python
class AModel(models.Model):

    _name = 'a_name'

    name = fields.Char(
        string="Name",                   # Etiqueta opcional del campo
        compute="_compute_name_custom",  # Convierte el campo en calculado
        store=True,                      # Si es calculado, almacena el resultado
        select=True,                     # Fuerza índice en el campo
        readonly=True,                   # El campo será solo lectura en las vistas
        inverse="_write_name",           # Al actualizar, dispara esta función
        required=True,                   # Campo obligatorio
        translate=True,                  # Habilita traducción
        help='blabla',                   # Texto de ayuda
        company_dependent=True,          # Convierte columnas a ir.property
        search='_search_function',       # Búsqueda personalizada, usado sobre todo con compute
        copy=True                        # Si se puede copiar con el método copy()
    )

    # La clave string no es obligatoria
    # Por defecto usará el nombre de la propiedad capitalizado

    name = fields.Char()  # Definición válida
```

Si queremos **valores por defecto**, se pueden indicar como un atributo del field.

```python
name = fields.Char(default='Alberto')
```
o
```python
name = fields.Char(default=a_fun)
...
def a_fun(self):
    return self.do_something()
```

### Fields normals 

Aquests són els fields per a dades normals que proporciona Odoo:

-   Integer
-   Char
-   Text
-   Date : Mostra un calendari en la vista.
-   Datetime
-   Float
-   Boolean
-   Html : Guarda un text, però es representa de manera especial en el
    client.
-   Binary : Per guardar, per exemple, imatges. Utilitza codificació
    base64 al enviar els fitxers al client. En realitat les guarda en
    **/var/lib/odoo/.local/share/Odoo/filestore** i la ruta als fitxers
    la diu la taula **ir_attachment** junt amb el id, nom del field i el
    model.
-   Image (Odoo13) : En el cas d\'imatges, accepta els atributs
    **max_width** i **max_height** on es pot dir en píxel que ha de
    redimensionar la imatge a eixa mida màxima.
-   Selection : Mostra un select amb les opcions indicades.

``` python
     type = fields.Selection([('1','Basic'),('2','Intermediate'),('3','Completed')])
     aselection = fields.Selection(selection='a_function_name') # Es pot cridar a una funció que defineix les opcions.
```

### Fields Relacionals 

Les relacions entre els models (en definitiva, entre les taules de la
base de dades) també les simplifica l\'ORM. D\'aquesta maneram les
relacions 1 a molts es fan en el ***Odoo*** anomena Many2one i les relacions
Mols a Molts es fan el el Many2Many. Les relacions molts a molts, en una
base de dades relacional, impliquen una tercera taula en mitg, però en
Odoo no tenim que preocupar-nos d\'aquestes coses si no volem, el mapat
dels objectes el detectarà i farà les taules, claus i restriccions
d\'integritat necessaries. Anem a repasar un a un aquests camps:

#### Reference

Una referència arbitrària a un model i un camp.
[2](http://www.zbeanztech.com/blog/reference-fields-odoo)

``` python
 
 aref = fields.Reference([('model_name', 'String')])
 aref = fields.Reference(selection=[('model_name', 'String')])
 aref = fields.Reference(selection='a_function_name')

# Fragment de test_new_api:
    reference = fields.Reference(string='Related Document', selection='_reference_models')
    @api.model
    def _reference_models(self):
        models = self.env['ir.model'].search([('state', '!=', 'manual')])
        return [(model.model, model.name)
                for model in models
                if not model.model.startswith('ir.')]
```

Els fields reference no són molt utilitzats, ja que normalment les
relacions entre models són sempre les mateixes.

#### Many2one

Relació amb un altre model

``` python
 arel_id = fields.Many2one('res.users')
 arel_id = fields.Many2one(comodel_name='res.users')
 an_other_rel_id = fields.Many2one(comodel_name='res.partner', delegate=True)
```

En aquest cas:

    ----------            -----------
    | Pais   |  one       |  Ciutat | 
    ---------- -----      -----------
    | * id   |     |      | * id    |
    | * name |     |  many| * name  |
    ----------     |------| * pais  |
                          -----------

El codi resultant sería:

``` python
class ciutat(models.Model):
    _name = 'mon.ciutat'
    pais = fields.Many2one("mon.pais", string='Pais', ondelete='restrict')
```

**delegate** està en True per a fer que els fields del model apuntat
siguen accessibles des del model actual.

Accepta també **context** i **domain** com en la vista. D\'aquesta
manera queda disponible per a totes les possibles vistes.

Un altre argument addicional és **ondelete** que permet definir el
comportament al esborrar l\'element referenciat a **set null**,
**restrict** o **cascade**.

> ondelete cascade esborra els fills a nivel de PostgreSQL, però no elimina en External Id, això es fa en unlink(), però no executa unlink() dels fills. Per tant, si volem que s'eliminen per complet, cal heretar el unlink del pare i afegir la cridada al dels fills.

#### One2many

Inversa del Many2one. Necessita de la existència d\'un Many2one en
l\'altre:

``` python
 arel_ids = fields.One2many('res.users', 'arel_id')
 arel_ids = fields.One2many(comodel_name='res.users', inverse_name='arel_id')
```

Un One2many funciona perquè hi ha un many2one en l\'altre model.
D\'aquesta manera, sempre has de especificar el nom del model i el nom
del camp Many2one del model que fa referencia a l\'actual, com es pot
veure en l\'exemple.

En l\'exemple anterior, quedaria com:

``` python
class pais(models.Model):
    _name = 'mon.pais'
    ciutats = fields.One2many('mon.ciutat', 'pais', string='Ciutats')
```

```{admonition} Important
:class: danger
És important entendre que el One2many no implica dades addicionals en la base de dades i sempre és calculat com un ''select'' en la base de dades on el id del model actual coincidisca amb el Many2one (clau aliena) de l'altre model. Això fa que no tinga sentit fer One2many computed o ficar un domain per restringit els que es poden afegir.
```
```{admonition} Domains
:class: tip
Els One2many poden tindre domain per no mostrar els que no compleixen una condició, això no significa que no existeixi aquesta relació.
```
#### Many2many

Relació molts a molts.

``` python
 arel_ids = fields.Many2many('res.users')
 arel_ids = fields.Many2many(comodel_name='res.users', # El model en el que es relaciona
                            relation='table_name', # (opcional) el nom del la taula en mig
                            column1='col_name', # (opcional) el nom en la taula en mig de la columna d'aquest model
                            column2='other_col_name')  # (opcional) el nom de la columna de l'altre model.
```

El primer exemple sol funcionar directament, però si volem tindre més
d\'una relació Many2many entre els dos mateixos models, cal utilitzar la
sintaxi completa on especifiquem el nom de la relació i el nom de les
columnes que identifiquem els dos models. Pensem que una relació
Many2many implica una taula en mig i estem especificant les seues claus
alienes.

```{tip}
També és precís especificar la taula en mig si es fa una relació Many2many al propi model.
```
```{tip}
Un Many2many implica una taula en mig. Si volem afegir atributs a aquesta relació, cal crear explícitament el model del mig. 

El many2many pot ser ''computed'' i en el còmput es pot ordenar o filtrar. Un Many2many computed no crea la taula en mig.
```
#### Related

Un camp d\'un altre model, necessita una relació Many2one. D\'aquesta
manera es poden aprofitar les funcionalitats de guardar, com ara les
búsquedes o referències en funcions. En termes de bases de dades, un
camp related trenca la tercera forma normal. Això sol ser problemàtic,
però ***Odoo*** té mecanismes per a que no passe res. De totes maneres, si ens
preocupa això, amb store=False no guarda res en la taula.

``` python
participant_nick = fields.Char(string='Nick name',
                               store=True,
                               related='partner_id.name'
```

Un camp related pot ser de qualsevol tipus. Per exemple, many2one:

``` python
sala = fields.Many2one('cine.sala',related='sessio.sala',store=True,readonly=True)
```

#### Many2oneReference

Un Many2one on es guardar també el model al qual fa referència amb l'atribut: **model_field**.

#### One2one

Els camps **One2one** no existeixen en ***Odoo***. Però si volem aquesta
funcionalitat podem utilitzar varies tècniques:

-   Fer dos camps Many2many i restringir amb constrains que sols pot
    existir una relació. Problemes:
    -   En la vista no podem ficar un widget com en el Many2one i és
        complicat evitar relacions creuades.
    -   Es pot fer un **limit** en la vista, però es continuarà
        comportant com un Many2many.
-   Fer dos Many2one i restringit amb contrains o sql constrains que
    sols pot existir una relació mútua. (Cal sobreescriure els mètodes
    create i write per a que es cree l\'associació automàticament).
    Problemes:
    -   Si sobreescribim el write de els dos, es pot produir una cridada
        recursiva sense fi i és molt complicat aconseguir que no tingam
        referències creuades.
-   Fer un Many2one i en l\'altre model un Many2one computed que busque
    en els del primer model. Per poder editar en els dos cal fer una
    funció inversa per al camp computed. Aquesta és una de les opcions
    més elegants. Exemple:


``` python
class orderline(models.Model):
    _name = 'sale.order.line'
    _inherit = 'sale.order.line'
    booking = fields.Many2one('reserves.bookings')
      
    _sql_constraints = [
    ('booking_uniq', 'unique(booking)', 'There is another order line for this booking'),
    ]

class bookings(models.Model):
    _name = 'reserves.bookings'

    name = fields.Char()
    order_line = fields.Many2one('sale.order.line',compute='_get_order_line',inverse='_set_order_line')

    @api.multi
    def _get_order_line(self):
        for b in self:
            b.order_line=self.env['sale.order.line'].search([('booking.id','=',b.id)]).id

    @api.one
    def _set_order_line(self):
        o = self.order_line.id
        self.env['sale.order.line'].search([('id','=',o)]).write({'booking':self.id})
```
-   Fer un Many2one i un One2many i restringir el màxim del One2many (
    [3](https://stackoverflow.com/questions/32801526/how-to-create-a-one2one-relationship-in-odoo-8)
    ). Problemes:
    -   Els mateixos que en els dos many2manys. És més simple restringir
        les relacions creuades.
-   Fer una herència múltiple.
    [4](http://blog.odoobiz.com/2014/10/openerp-one2one-relational-field-example.html).
    Problemes:
    -   Esta és, en teoría, la forma més oficial de fer-ho, però obliga
        a crear sempre la relació i els models en un ordre determinat.

#### Filtres (Domains) 

En ocasions és necessari afegir un filtre en el codi python per fer que
un camp **relacional** no puga tindre certes referències. El
comportament del domain és diferent depen del tipus de field.

-   **Domain en Many2one**: Filtra els elements del model referenciat
    que poden ser elegits per al field:

``` python
parent = fields.Many2one('game.resource', domain="[('template', '=', True)]")
```

-   **Domain en Many2many**: La llista d\'elements a triar es filtra
    segons el domain:

``` python
allowed_value_ids = fields.Many2many(
    comodel_name="x",
    compute="_compute_allowed_value_ids"
)

def _compute_allowed_value_ids(self):
    for record in self:
        record.allowed_value_ids = self.env["x"].search(...)

value_id = fields.Many2many(
    comodel_name="x",
    domain="[('id', 'in', allowed_value_ids)]",
)
```

-   **Domain en One2many**: Al ser una relació que depen d\'altre
    Many2one, no es pot filtrar, si fiquem un domain, sols deixarà de
    mostrar els que no compleixen el domain, però no deien d\'existir:




### Fields Computed 

Moltes vegades volem que el contingut d\'un camp siga calculat en el
moment en que anem a veure-lo. Tots els tipus de fields poden ser
computed. Anem a veure alguns exemples:

``` python
   taken_seats = fields.Float(string="Taken seats", compute='_taken_seats')   # Aquest camp no es guarda a la base de dades 
                                                                #i sempre es recalcula quan executem un action que el mostra

   @api.depends('seats', 'attendee_ids')  # El decorador @api.depends() indica que es cridarà a la funció 
                                          # sempre que es modifiquen els camps seats i attendee_ids. 
                                          #Si no el posem, es recalcula sols al recarregar el action.
   def _taken_seats(self):          
      for r in self:  # El for recorre self, que és un recordset amb tots els elements del model mostrats 
                      # per la vista, si és un tree, seran tots els visibles i si és un form, serà un singleton.
          if not r.seats: # r és un singleton i es pot accedir als fields com a variables de l'objecte.      
              r.taken_seats = 0.0 
          else:
              r.taken_seats = 100.0 * len(r.attendee_ids) / r.seats
```

En aquest exemple es veu cóm el camp *float* taken seats es calcula en
una funció privada \_taken_seats. És interessant observar el *for*
perquè recorre totes les instancies a les que fa referència el model.
Aquesta funció sols s\'executarà una vegada encara que tinga que
calcular tots els elements d\'una llista. Per això, la propia funció és
la que té que iterar els elements de **self**. **self** és un
`recordset`, per tant, és com una llista
en la que cada element és un registre del model. Si el computed és
cridat al entrar a un formulari, el recordset tindrà sols un element,
però si el camp computed es veu en una llista (tree), pot ser que siguen
més d\'un registre. És important recordar fer el **for record in self:**
encara que pensem que el camp computed sols l\'utilitzarem en un
formulari.

Exemples de **computed** de tots els tipus de fields:

``` python
# -*- coding: utf-8 -*-

from openerp import models, fields, api, tools
from datetime import date, datetime

class proves_computed(models.Model):
     _name = 'proves_computed.proves_computed'

     name = fields.Char()
     value = fields.Integer()
     image = fields.Binary(String="Image original")
     computedfloat = fields.Float(compute="_value_pc", store=True)
     computedchar = fields.Char(compute="_value_pc", store=False)
     medium_image = fields.Binary(compute="_redimensionar", store=True)
     small_image = fields.Binary(compute="_redimensionar", store=True)
     computedm2o = fields.Many2one('res.partner',compute="_value_pc", store=False)
     computedm2m = fields.Many2many(comodel_name='product.template',compute="_value_pc", store=False)
     computeddate = fields.Date(compute="_value_pc", store=False)
     computeddatetime = fields.Datetime(compute="_value_pc", store=False)
     
     description = fields.Text()

     @api.depends('value')
     def _value_pc(self):
      for r in self:
        r.computedfloat = float(r.value) / 100 
        r.computedchar = "("+str(r.value)+")"
        r.computedm2o = self.env['res.partner'].search([('id','=',r.value)]).id # Many2one espera un id, que és un camp Integer. 
        print '\033[93m'+str(self.env['product.product'].search([('id','>',r.value)]).ids)+'\033[0m'
        r.computedm2m = self.env['product.template'].search([('id','>',r.value)]).ids #Many2many espera un array d'ids o un recordset. 
        # El codi comentat a continuació fa el mateix, per si volem fer altres coses dins del for.
        #ids = []
        #for t in self.env['product.template'].search([('id','>',r.value)]):
        # ids.append(t.id)
        #r.computedm2m = ids
 
        #r.computeddate = date.today() # Aquest depen de Python
        r.computeddate = fields.date.today() # Recomanem aquest, ja que és propi de la classe fields d'Odoo
        #r.computeddate = datetime.now()
        r.computeddatetime = fields.datetime.now()
        

     @api.depends('image')
     def _redimensionar(self):
       for r in self:
         image_original = r.image
         if image_original:
            images = tools.image_get_resized_images(image_original)
            r.medium_image = images['image_medium']                        
            r.small_image = images['image_small']                
         else:
            r.medium_image = ""                        
            r.small_image = ""
```

([Codi
complet](https://github.com/xxjcaxx/SGE-Odoo-2016-2017/tree/master/proves_computed))


> En l\'apartat del `controlador` s\'expliquem més detalls de les funcions en python-odoo.

#### Buscar i escriure en camps computed 

Amb el **api.depends** podem fer que camps calculats puguen ser buscats
o referenciats des d\'uns altres models, ja que podem dir que sí se
guarden en la base de dades. Si es guarda en la base de dades no es
recalcula fins que no canvia el contingut del *field* del que depèn.
Però si el camp calculat no depèn de valors estàtics d\'altres *fields*
i/o necessitem que sempre es calcule, no tenim moltes opcions elegants.
Una d\'elles pot ser fer dos camps, un calculat **store=False** i altre
no i fer un *write* en la funció. L\'altra possibilitat és fer una
funció pública que puga ser cridada des d\'un altre model. La més
elegant que no sempre funciona és utilitzar l\'opció **search** i
assignar-li una funció que ha de retornar un domini de búsqueda. El
problema és que no accepta molta complexitat, ja que suposa una cerca
per tota la base de dades i pot ser molt ineficient.

Per defecte no es pot escriure en un camp *computed*. No té massa sentit
la majoria dels casos, ja que és un camp que depèn d\'altres. Però pot
ser que, de vegades volem escriure el resultat i que modifique el camp
origen. Imaginem, per exemple, que sabem el preu final i volem que
calcule el preu sense IVA. Per fer-ho, la millor manera és crear una
funció i fer que estiga en l\'opció **inverse**.

Exemple:

``` python
 preu = fields.Float('Price',compute="_get_price",search='_search_price',inverse='_set_price')

 @api.depends('pelicula','descompte')
      def _get_price(self):
        for r in self:
          price = r.pelicula.preu
          price = price - (price*r.descompte/100)
          r.preu = price

      def _search_price(self,operator,value): # De moment aquest search sols és per a ==
       preus = self.search([]).mapped(lambda e: [e.id , e.pelicula.preu - (e.pelicula.preu*e.descompte/100)]) # Un bon exemple de mapped en lambda
       print preus
       p = [ num[0] for num in preus if num[1] == value]  # condició if en una llista python sense fer un for (list comprehension)
       # també es pot provar en un filter() de python
       print p
       # p és una llista de les id que ja compleixen la condició, per tant sols cal fer que la id estiga en la llista.
       return [('id','in',p)]

      def _set_price(self):
       self.pelicula.preu = self.preu  # Açò és un exemple, però està mal, ja que modifiques el preu de la peli en totes les sessions
```

Documentació oficial: https://www.odoo.com/documentation/master/developer/reference/backend/orm.html

### Valors per defecte 

En ***Odoo*** és molt fàcil fer valors per defecte, ja que és un argument més
en el constructor dels fields:

``` python
name = fields.Char(default="Unknown")
user_id = fields.Many2one('res.users', default=lambda self: self.env.user)
start_date = fields.Date(default=fields.Date.today())
active = fields.Boolean(default=True)
def compute_default_value(self):
    return self.get_value()
a_field = fields.Char(default=compute_default_value)
```

Si volem, per exemple, ficar la data del moment de crear, no podem fer
això:

``` python
start_date = fields.Date(default=fields.Date.today())  # MAL
```

Perquè calcula la data del moment d\'actualitzar el mòdul, no el de
crear l\'element en el model. Cal fer:

``` python
start_date = fields.Date(default=lambda self: fields.Date.today())  # CORRECTE
```

o

``` python
start_date = fields.Datetime(default=lambda self: fields.Datetime.now()) # CORRECTE
```

El valor per defecte no pot dependre d\'un field que està creant-se en
eixe moment. En eixe cas es pot utilitzar un **on_change**.

En cas de tindre molts valors per defecte o que depenen del context, es
pot utilitzar la funció **default_get** que ja tenen els models.

``` python
    @api.model
    def default_get(self, default_fields):
        result = super(SelectSalePrice, self).default_get(default_fields)
        if self._context.get('default_picking_id') is not None:
            result['picking_id'] = self._context.get('default_picking_id')
        return result
```

El que fa aquesta funció és un poc avançat de moment, ja que fa ús del `context` i l\'herencia per afegir un valor per
defecte al diccionari que retorna aquesta funció en la classe *Model*

### Restriccions (constrains) 

Els objectes poden incorporar, de forma opcional, restriccions
d'integritat, addicionals a les de la base de dades. ***Odoo*** valida
aquestes restriccions en les modificacions de dades i, en cas de
violació, mostra una pantalla d'error.

``` python
from ***Odoo***.exceptions import ValidationError

@api.constrains('age')
def _check_something(self):
    for record in self:
        if record.age > 20:
            raise ValidationError("Your record is too old: %s" % record.age)
    # all records passed the test, don't return anything
```

En ocasions, quan tenim clar cóm faríem aquesta restricció en SQL, tal
vegada ens resulte més interessant fer una restricció de la base de
dades amb una **sql constraint**. Aquestes es defineixen amb 3 strings
**(name, sql_definition, message)**. Per exemple:

``` python
_sql_constraints = [
    ('name_uniq', 'unique(name)', 'Custom Warning Message'),
    ('contact_uniq', 'unique(contact)', 'Custom Warning Message')
]
```

En aquest cas és una restricció d\'unicitat, la qual és més simple que
fer una búsqueda en python.

## Fitxers de dades 

Quan fem un mòdul d\'Odoo, es poden definir dades que es guardaran en la
base de dades. Aquestes dades poden ser necessàries per al funcionament
del mòdul, de demostració o inclús part de la vista.

```{tip}
Alguns mòduls sols estan per clavar dades en Odoo
```
Tots els fitxers de dades són en XML i tenen una estructura com esta:

``` xml
<odoo>
        <record model="{model name}" id="{record identifier}">
            <field name="{a field name}">{a value}</field>
        </record>
<odoo>
```

Dins de les etiquetes **odoo** poden trobar una etiqueta **record** per cada
fila en la taula que volem introduir. Cal especificar el model i el id.
El **id** és un identificador extern, que no te perquè coincidir amb la
clau primària que l\'ORM utilitzarà després. Cada **field** tindrà un
nom i un valor.

### External Ids 

Tots els records de la base de dades tenen un identificador únic en la
seua taula, el **id**. És un número auto incremental assignat per la
base de dades. No obstant, si volem fer referència a ell en fitxers de
dades o altres llocs, no sempre tenim perquè saber el id. La solució
d\'odoo són els **External Identifiers**. Això és una taula que
relaciona cada id de cada taula en un nom. Es tracta del model
**ir.model.data**. Per trobar-los cal anar a:

`Settings > Technical > Sequences & identifiers > External Indentifiers`

Ahí dins trobem la columna **Complete ID**.

Per trobar les *id* al fer fitxers de demostració o de dades podem anar
al menú, però eixes ids canvien d\'una instal·lació a un altra. Per
tant, cal utilitzar les **external id**. Per aconseguir-lo podem obrir
el mode desenvolupador i obrir el menú **View Metadata**.

En les dades de demo, els **external Ids** s\'utilitzen per no utilitzar
les id, que poden canviar al ser auto incrementals. Per a que funcione
cal utilitzar l\'atribut **ref**:

``` xml
<field name="product_id" ref="product.product1"/>
```

```{tip}
Es recomana fer l'atribut '''id''' en el record, encara que no sobreescriu el id real, serveix per declarar el External Id i és més fàcil després fer referència a ell.
```
> Veure també la funció **ref()** de l\'ORM

### Expressions

De vegades volem que els fields es calculen cada vegada que
s\'actualitza el mòdul. Això es pot fer en l\'atribut **eval** que
avalua una expressió de Python.

``` xml
<field name="date" eval="(datetime.now()+timedelta(-1)).strftime('%Y-%m-%d')"/>
<field name="product_id" eval="ref('product.product1')"/> # Equivalent a l'exemple anterior
<field name="price" eval="ref('product.product1').price"/>
<field name="avatar" model="school.template" eval="obj().env.ref('school.template_student1').image" ></field>  # Com que utilitza obj() necessita model="...
```

Per al x2many, es pot utilitzar el eval per assignar una llista
d\'elements.

``` xml
<field name="tag_ids" eval="[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_compact'),
                                   ref('fleet.vehicle_tag_senior')] )]" />
```

Observem que hem passat una tripleta amb un 6, un 0 i una llista de
refs. Les tripletes poden ser:

-   (0,\_ ,{\'field\': value}): Crea un nou registre i el vincula
-   (1,id,{\'field\': value}): Actualtiza els valors en un registre ja
    vinculats
-   (2,id,\_): Desvincula i elimina el registre
-   (3,id,\_): Desvincula però no elimina el registre de la relació.
-   (4,id,\_): Vincula un registre ja existent
-   (5,\_,\_): Desvincula pero no elimina tots els registres vinculats
-   (6,\_,\[ids\]): Reemplaça la llista de registres vinculats.

### Dades per als Binary i Image

Algunes dades com les imatges o fitxers es poden posar en records. Hi ha dos maneres: 

* Convertir en Base64 el fitxer i copiar i pegar el resultat dins del field.
* Afegir el atribut `type="base64"` i el atribut `file="modul/demo/fitxer"`

```xml
<field name="image_1920" type="base64" file="exemple/demo/cares/1000.jpg"/>
```
Observem la ruta que inicia des del directori del mòdul.

### Esborrar

Amb l\'etiqueta **delete** es pot especificar els elements a esborrar
amb el external ID o amb un search:

``` xml
<delete model="cine.sessio" id="sessio_cine1_1"></delete>
```

```{danger}
Si falla l'actualització amb dades de demo, és possible que ***Odoo*** deshabilite la possibilitat de tornar-les a instal·lar. Això és el field demo de ir.module.module que és readonly, per tant, cal modificar-lo a ma en la base de dades:

`update ir_module_module set demo = 't' where name='school';`
```

Més informació: https://www.odoo.com/documentation/master/developer/reference/backend/data.html

## Accions i menús 

Si vols conèixer en més detall cóm funcionen les accions en Odoo, llig
l\'article **Accions i menús en Odoo**

El client web de ***Odoo*** conté uns menús dalt i a l\'esquerra. Aquests
menús, al ser accionats mostren altres menús i les pantalles del
programa. Quant pulsem en un menú, canvia la pantalla perquè hem fet una
[acció](https://www.odoo.com/documentation/master/developer/reference/backend/actions.html).

Una acció bàsicament té:

-   **type**: El tipus d\'acció que és i cóm l\'acció és interpretada.
    Quan la definim en el XML, el type no cal especificar-lo, ja que ho
    indica el model en que es guarda.
-   **name**: El nom, que pot ser mostrat en la pantalla o no. Es
    recomana que siga llegible per els humans.

Les accions i els menús es declaren en fitxers de dades en XML o
dirèctament si una funció retorna un diccionari que la defineix. Les
accions poden ser cridades de tres maneres:

-   Fent clic en un menú.
-   Fent clic en botons de les vistes (han d\'estar connectats amb
    accions).
-   Com accions contextuals en els objectes.

D\'aquesta manera, el client web pot saber quina acció ha d\'executar si
rep alguna d\'aquestes coses:

-   **false**: Indica que s\'ha de tancar el diàleg actual.
-   **Una string**: Amb l\'etiqueta de **l\'acció de client** a
    executar.
-   **Un número**: Amb el ID o external ID de l\'acció a trobar a la
    base de dades.
-   **Un diccionari**: Amb la definició de l\'acció, aquesta no està ni
    en XML ni en la base de dades. En general, és la manera de cridar a
    un action al finalitzar una funció.

### Accions tipus *window* 

Les accions *window* són un record més (**ir.actions.act_window**). No
obstant, els menús que les criden, tenen una manera més ràpida de ser
declarats amb una etiqueta **menuitem**:

``` xml
<record model="ir.actions.act_window" id="action_list_ideas">
    <field name="name">Ideas</field>
    <field name="res_model">idea.idea</field>
    <field name="view_mode">tree,form</field>
</record>
<menuitem id="menu_ideas" parent="menu_root" name="Ideas" sequence="10"
          action="action_list_ideas"/>
```

```{tip}
 Les accions han de ser declarades al XML abans que els menús que les accionen. 
```

Exemple:

``` xml

<?xml version="1.0" encoding="UTF-8"?>
<openerp>
    <data>
        <!-- window action -->
        <!--
            The following tag is an action definition for a "window action",
            that is an action opening a view or a set of views
        -->
        <record model="ir.actions.act_window" id="course_list_action">
            <field name="name">Courses</field>
            <field name="res_model">openacademy.course</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
                <p class="oe_view_nocontent_create">Create the first course
                </p>
            </field>
        </record>

        <!-- top level menu: no parent -->
        <menuitem id="main_openacademy_menu" name="Open Academy"/>
        <!-- A first level in the left side menu is needed
             before using action= attribute -->
        <menuitem id="openacademy_menu" name="Open Academy"
                  parent="main_openacademy_menu"/>
        <!-- the following menuitem should appear *after*
             its parent openacademy_menu and *after* its
             action course_list_action -->
        <menuitem id="courses_menu" name="Courses" parent="openacademy_menu"
                  action="course_list_action"/>
        <!-- Full id location:
             action="openacademy.course_list_action"
             It is not required when it is the same module -->
    </data>
</openerp>
```
Sols el tercer nivell de menús pot tindre associada un action. El primer
és el menú de dalt i el segon no es \'clicable\'.

```{tip}
El que hem vist en esta secció és la definició d'una acció en un XML com a part de la vista, però una acció no és més que una forma còmoda d'escriure moltes coses que farà el client en javascript per demanar alguna cosa al servidor. Els actions separen i simplifiquen el desenvolupament de la interfície d'usuari que és el client web. Un menú o botó en html acciona una funció javascript que en principi no sap el que fer. Aquesta demana que es carregue la definició del seu action. Una vegada carregada la definició, queda clar tot el que ha de demanar (les vistes, context, dominis, vistes search, lloc on carregar-ho tot...) aleshores demana les vistes i amb ajuda de les vistes i els fields, demana els records que són les dades a mostrar. Per tant, un action és la definició sense programar javascript de coses que ha de fer el javascript. ***Odoo*** permet declarar actions com a resposta de funcions. Aquestes actions no estan en la base de dades, però són enviades igualment al client i el client fa en elles el mateix que en un action que ell ha demanat. Un exemple d'això són els actions que retornen els botons dels wizards. De fet, podem fer que un botó torne un action i, per tant, obrir una vista diferent. 
```

## La vista 

Per saber més sobre les vistes i cómo millorar-les, consulta l\'article
de **La vista en Odoo**


## Herència

El framework d\'Odoo facilita el mecanisme de l'herència per tal que els
programadors puguin adaptar mòduls existents i garantir a la vegada que
les actualitzacions dels mòduls no destrossin les adequacions
desenvolupades.

L'herència es pot aplicar en els tres components del patró MVC:

-   En el model: possibilita ampliar les classes existents o dissenyar
    noves classes a partir de les existents.
-   En la vista: possibilita modificar el comportament de vistes
    existents o dissenyar noves vistes.
-   En el controlador: possibilita sobreescriure els mètodes existents o
    dissenyar-ne de nous.

OpenObject proporciona tres mecanismes d'herència: l'herència de classe,
l'herència per prototip i l'herència per delegació.

| **Mecanisme**     | **Característiques** | **Com es defineix** |
|------------------|----------------------|---------------------|
| **De classe** | - Herència simple. <br>- La classe original queda substituïda o ampliada.  <br>- Afegeix noves funcionalitats (atributs i/o mètodes) a la classe original.  <br>- Les vistes definides sobre la classe original continuen funcionant.  <br>- Permet sobreescriure mètodes de la classe original.  <br>- En PostgreSQL, continua mapada en la mateixa taula que la classe original, ampliada amb els nous atributs que pugui incorporar. | - S'utilitza l'atribut `_inherit` en la definició de la nova classe Python: `_inherit = 'obj'`.  <br>- El nom de la nova classe ha de continuar sent el mateix que el de la classe original: `_name = 'obj'`. |
| **Per prototip** | - Herència simple.  <br>- Aprofita la definició de la classe original (com si fos un «prototipus»).  <br>- La classe original continua existint.  <br>- Afegeix noves funcionalitats (atributs i/o mètodes) a les aportades per la classe original.  <br>- Les vistes definides sobre la classe original no existeixen (cal dissenyar-les de nou).  <br>- Permet sobreescriure mètodes de la classe original.  <br>- En PostgreSQL, queda mapada en una nova taula. | - S'utilitza l'atribut `_inherit` en la definició de la nova classe Python: `_inherit = 'obj'`.  <br>- Cal indicar el nom de la nova classe: `_name = 'nou_nom'`. |
| **Per delegació** | - Herència simple o múltiple.  <br>- La nova classe «delega» certs funcionaments a altres classes que incorpora a l'interior.  <br>- Els recursos de la nova classe contenen un recurs de cada classe de la que deriven.  <br>- Les classes base continuen existint.  <br>- Afegeix les funcionalitats pròpies (atributs i/o mètodes) que correspongui.  <br>- Les vistes definides sobre les classes bases no existeixen a la nova classe.  <br>- En PostgreSQL, queda mapada en diferents taules: una taula per als atributs propis, mentre que els recursos de les classes derivades resideixen en les taules corresponents a les dites classes. | - S'utilitza l'atribut `_inherits` en la definició de la nova classe Python: `_inherits = {'obj': 'field_id'}`.  <br>- Cal indicar el nom de la nova classe: `_name = 'nou_nom'`. |


    ```{figure} imgs/Inheritance_methods.png
    :scale: 100 %
    :alt: Herència

    Diferents modes d'herència
    ```

### Herència en el Model 

El disseny d'un model d'Odoo heretat és paregut al disseny d'un no heretat; únicament hi ha dues diferències:

-   Apareix l'atribut **\_inherit** o **\_inherits** per indicar
    l'objecte (herència simple) o els objectes (herència múltiple) dels
    quals deriva el nou objecte. La sintaxi a seguir és:

`_inherit = 'nom.objecte.del.que.es.deriva'`\
`_inherits = {'nom.objecte1':'nom_camp_FK1', ...}`

-   En cas d'herència simple, el nom (atribut \_name) de l'objecte
    derivat pot coincidir o no amb el nom de l'objecte pare. També és
    possible no indicar l'atribut \_name, fet que indica que el nou
    objecte manté el nom de l'objecte pare.

L'herència simple (\_inherit) amb atribut \_name idèntic al de l'objecte
pare, s'anomena herència de classe i en ella el nou objecte substitueix
l'objecte pare, tot i que les vistes sobre l'objecte pare continuen
funcionant. Aquest tipus d'herència, la més habitual, s'utilitza quan es
vol afegir *fields* i/o modificar propietats de dades existents i/o
modificar el funcionament d'alguns mètodes. En cas d'afegir dades,
aquestes s'afegeixen a la taula de la base de dades en la qual estava
mapat l'objecte pare.


**Exemple d\'herència de classe** L'herència de classe la trobem en
molts mòduls que afegeixen dades i mètodes a objectes ja existents, com
per exemple, el mòdul comptabilitat (account) que afegix dades i mètodes
a l'objecte res.partner. Fixem-nos en el contingut del mòdul account:

``` python
    class res_partner(Model.model):
    _inherit = 'res.partner'
    debit_limit = fields.float('Payable limit')
    ...
```

Podeu comprovar que la taula res_partner d'una empresa sense el mòdul
account instal·lat no conté el camp debit_limit, que en canvi sí que hi
apareix una vegada instal·lat el mòdul.

Odoo té molts mòduls que deriven de l'objecte res.partner per afegir-hi
característiques i funcionalitats.

L'herència simple (\_inherit) amb atribut **\_name** diferent al de
l'objecte pare, s'anomena **herència per prototip** i en ella es crea un
nou objecte que aglutina les dades i mètodes que tenia l'objecte del
qual deriva, juntament amb les noves dades i mètodes que pugua
incorporar el nou objecte. En aquest cas, sempre es crea una nova taula
a la base de dades per mapar el nou objecte.

**Exemple d\'herència per prototip** L'herència per prototip és difícil
de trobar en els mòduls que incorpora ***Odoo***. Un exemple el tenim en el
mòdul base_calendar en el qual podem observar el mòdul comptabilitat
(account) que afegix dades i mètodes a l'objecte res.partner. Fixem-nos
en el contingut del mòdul account:

``` python
    class res_alarm(Model.model):
    _name = 'res.alarm'
    ...
    class calendar_alarm(Model.model):
    _name = 'calendar.alarm'
    _inherit = 'res.alarm'
    ...
```

En una empresa que tingui el mòdul base_calendar instal·lat podeu
comprovar l'existència de la taula res_alarm amb els camps definits a
l'apartat \_atributs de la classe res_alarm i la taula calendar_alarm
amb camps idèntics als de la taula res_alarm més els camps definits a
l'apartat \_atributs de la classe calendar_alarm.

```{tip}
L'herència per prototip és la tradicional en els llenguatges orientats a objectes, ja que crea una nova classe vinculada
```
L'herència múltiple (\_inherits) s'anomena herència per delegació i
sempre provoca la creació d'una nova taula a la base de dades. L'objecte
derivat ha d'incloure, per cada derivació, un camp many2one apuntant
l'objecte del qual deriva, amb la propietat **ondelete=\'cascade**\'.
L'herència per delegació obliga que cada recurs de l'objecte derivat
apunte a un recurs de cadascun dels objectes dels quals deriva i es pot
donar el cas que hi hagi diversos recursos de l'objecte derivat que
apunten a un mateix recurs per algun dels objectes dels quals deriva.

``` python
    class res_alarm(Model.model):
    _name = 'res.alarm'
    ...
    class calendar_alarm(Model.model):
    _name = 'calendar.alarm'
    _inherits = {'res.alarm':'alarm_id'}
    ...
```

### Herència en la vista 

L'herència de classe possibilita continuar utilitzant les vistes
definides sobre l'objecte pare, però en moltes ocasions interessa
disposar d'una versió retocada. En aquest cas, és molt millor heretar de
les vistes existents (per afegir, modificar o eliminar camps) que
reemplaçar-les completament.

``` xml
 <field name="inherit_id" ref="id_xml_vista_pare"/>
```

En cas que la vista id_xml_vista_pare estiga en un mòdul diferent del
que estem dissenyant, cal afegir el nom del mòdul al davant:

``` xml
 <field name="inherit_id" ref="modul.id_xml_vista_pare"/>
```

El motor d'herència d'OpenObject, en trobar una vista heretada, processa
el contingut de l'element arch. Per cada fill d'aquest element que
tingui algun atribut, OpenObject cerca a la vista pare una etiqueta amb
atributs coincidents (excepte el de la posició) i, a continuació,
combina els camps de la vista pare amb els de la vista heretada i
estableix la posició de les noves etiquetes a partir dels següents
valors:

-   inside (per defecte): els valors s'afegeixen "dins" de l'etiqueta.
-   after: afegeix el contingut després de l'etiqueta.
-   before: afegeix el contingut abans de l'etiqueta.
-   replace: reemplaça el contingut de l'etiqueta.
-   attributes: Modifica [els
    atributs](https://www.odoo.com/es_ES/forum/ayuda-1/question/xpath-how-to-replace-attributes-only-and-not-the-full-field-38192).

**Reemplaçar**

``` xml
 <field name="arch" type="xml">
   <field name="camp" position="replace">
     <field name="nou_camp" ... />
   </field>
 </field>
```

**Esborrar**

``` xml
 <field name="arch" type="xml">
   <field name="camp" position="replace"/>
 </field>
```

**Inserir nous camps**

``` xml
 <field name="arch" type="xml">
    <field name="camp" position="before">
       <field name="nou_camp" .../>
    </field>
 </field>

 <field name="arch" type="xml" style="font-family:monospace">
    <field name="camp" position="after">
       <field name="nou_camp" .../>
    </field>
 </field>
```

**Fer combinacions**

``` xml
 <field name="arch"type="xml">
   <data>
     <field name="camp1" position="after">
       <field name="nou_camp1"/>
     </field>
     <field name="camp2" position="replace"/>
     <field name="camp3" position="before">
        <field name="nou_camp3"/>
     </field>
   </data>
 </field>
```

Per definir la posició dels elements que afegim, podem utilitzar una
expresió **xpath**:

``` xml
 <xpath expr="//field[@name='order_line']/tree/field[@name='price_unit']" position="before">
 <xpath expr="//form/*" position="before">
  <header>
    <field name="status" widget="statusbar"/>
  </header>
 </xpath>
```

És posssible que necessitem una vista totalment nova de l\'objecte
heredat. Si fem un action normal en l\'XML es veuran els que més
prioritat tenen. Si volem especificar quina vista volem en concret hem
d\'utilitzar **view_ids**, observem aquest exemple:

``` xml
        <record model="ir.actions.act_window" id="terraform.player_action_window">
            <field name="name">Players</field>
            <field name="res_model">res.partner</field>
            <field name="view_mode">tree,form,kanban</field>
            <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'tree', 'view_id': ref('terraform.player_tree')}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('terraform.player_form')}),]" />
        </record>
```

En **(0,0,{registre_a\_crear})** li diguem que a eixe Many2many hi ha
que afegir un nou registre amb eixes dades en concret. El que necessita
és el **view_mode** i el **view_id**, com en els records anteriors.

Si es vol especificar una vista search es pot inclourer la etiqueta
**search_view_id**:

``` xml
 <field name="search_view_id" ref="cine.pos_order_line_search_view"/>  
```

**Domains**

Si volem que el action heredat sols mostre els elements que volem, s\'ha
de ficar un domain en el action:

``` xml
<field name="domain"> [('isplayer','=',True)]</field> 
```

Amés, es pot dir que, per defecte, quan es crea un nou registre a través
d\'aquest action, tinga el field a True:

``` xml
<field name="context">{'default_is_player': True}</field>
```

**Filtre per defecte**

El problema en la solució anterior és que lleva la possibilitat de veure
el que no tenen aquest field a True i cal anar per un altre action a
modificar-los. Si volem poder veure tots, podem crear un filtre en la
vista search i en l\'action dir que volem aquest filtre per defecte:

``` xml
<!--   En la vista search -->
...
    <search>
        <filter name="player_partner" string="Is Player" domain="[('is_player','=',True)]" />
    </search>
...
<!-- En l'action -->
            <!--  <field name="domain"> [('is_player','=',True)]</field> -->
            <field name="domain"></field>
            <field name="context">{'default_is_player': True, 'search_default_player_partner': 1}</field>
```

Per tant, un action complet per a vistes personalitzades i amb filtres quedarà com aquest:

``` xml
        <record model="ir.actions.act_window" id="terraform.player_action_window">
            <field name="name">Players</field>
            <field name="res_model">res.partner</field>
            <field name="view_mode">tree,form,kanban</field>
            <field name="domain"></field>
            <field name="context">{'default_is_player': True, 'search_default_player_partner': 1}</field>
            <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'tree', 'view_id': ref('terraform.player_tree')}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('terraform.player_form')}),]" />
        </record>
```

### Herència en el controlador 

L'herència en el controlador és un mecanisme conegut, ja que l'apliquem
de forma inconscient quan ens veiem obligats a sobreescriure els mètodes
de la capa ORM d'OpenObject en el disseny de molts mòduls.

```{tip}
Funció super()

El llenguatge Python recomana utilitzar la funció super() per invocar el mètode de la classe base quan s’està sobreescrivint en una classe derivada, en lloc d’utilitzar la sintaxi nomClasseBase.metode(self…).
```
L'efecte de l'herència en el controlador es manifesta únicament quan cal
sobreescriure algun dels mètodes de l'objecte del qual es deriva i per a
fer-ho adequadament cal tenir en compte que el mètode sobreescrit en
l'objecte derivat:

-   De vegades vol substituir el mètode de l'objecte base sense
    aprofitar-ne cap funcionalitat: el mètode de l'objecte derivat no
    invoca el mètode sobreescrit.
-   De vegades vol aprofitar la funcionalitat del mètode de l'objecte
    base: el mètode de l'objecte derivat invoca el mètode sobreescrit.

Exemples:

[Sobreescriure el mètode
**create**](http://www.odoo.yenthevg.com/override-create-functions-odoo/):

``` python
class res_partner(models.Model):
    _inherit = 'res.partner'
    passed_override_write_function = fields.Boolean(string='Has passed our super method')
 
    @api.model
    def create(self, values):
        # Override the original create function for the res.partner model
        record = super(res_partner, self).create(values)
 
        # Change the values of a variable in this super function
        record['passed_override_write_function'] = True
        print 'Passed this function. passed_override_write_function value: ' + str(record['passed_override_write_function'])
 
        # Return the record so that the changes are applied and everything is stored.
    return record
```

## El controlador 

Part del controlador l\'hem mencionat al parlar dels camps **computed**.
No obstant, cal comentar les facilitats que proporciona ***Odoo*** per a no
tindre que accedir dirèctament a la base de dades.

La capa **ORM** d'Odoo facilita uns mètodes que s'encarreguen del
mapatge entre els objectes Python i les taules de PostgreSQL. Així,
disposem de mètodes per crear, modificar, eliminar i cercar registres a
la base de dades.

En ocasions, pot ser necessari alterar l'acció automàtica de cerca --
creació -- modificació -- eliminació facilitada per ***Odoo*** i haurem de
sobreescriure els corresponents mètodes en les nostres classes.

Els programadors en el framework d\'Odoo hem de conèixer els mètodes
subministrats per la capa ORM i hem de dominar el disseny de mètodes
per:

-   Poder definir camps funcionals en el disseny del model.
-   Poder definir l'acció que cal executar en modificar el contingut
    d'un field d'una vista form (@api.onchange)
-   Poder alterar les accions automàtiques de cerca, creació,
    modificació i eliminació de recursos.

Una darrera consideració a tenir en compte en l'escriptura de mètodes i
funcions en ***Odoo*** és que els textos de missatges inclosos en mètodes i
funcions, per poder ser traduïbles, han de ser introduïts amb la sintaxi
\_(\'text\') i el fitxer .py ha de contenir from tools.translate import
\_ a la capçalera.

### API de l\'ORM 

```{tip}
**Interactuar en la terminal**
    $ ***Odoo*** shell -d castillo -u containers

Observa cóm hem ficat el paràmetre '''shell'''. Les coses que se fan en la terminal no són persistents en la base de dades fins que no s'executa '''self.env.cr.commit()'''. Dins de la terminal podem obtindre ajuda dels mètodes d'Odoo amb help(), per exemple: help(tools.image)
Amb el següent exemple, podem arrancar ***Odoo*** sense molestar a l'instància que està en marxa redefinint els ports:

    $ ***Odoo*** shell -c /path/to/odoo.conf --xmlrpc-port 8888 --longpolling-port 8899

https://asciinema.org/a/123126 (Asciinema amb alguns exemples)
```

Un mètode creat dins d\'un model actua sobre tots els elements del model
que estiguen actius en el moment de cridar al mètode. Si és un tree,
seran molts i si és un form sols un. Però en qualsevol cas és una
\'llista\' d\'elements i es diu **recordset**.

Bàsicament la interacció amb els models en el controlador es fa amb els
anomenats **recordsets** que són col·leccions d\'objectes sobre un
model. Si iterem dins dels recordset , obtenim els **singletons**, que
són objectes individuals de cada línia en la base de dades.

``` python
def do_operation(self):
    print self # => a.model(1, 2, 3, 4, 5)
    for record in self:
        print record # => a.model(1), then a.model(2), then a.model(3), ...
```

Podem accedir a tots els fields d\'un model sempre que estem en un
singleton, no en un recordset:

``` python
>>> record.name
Example Name
>>> record.company_id.name
Company Name
>>> record.name = "Bob"
```

Intentar llegir o escriure un field en un recordset donarà un error.
Accedir a un **many2one, one2many o many2many** donarà un recordset.

#### Set operations 

Els recordsets es poden combinar amb operacions específiques que són les
típiques dels conjunts:

-   **record in set** retorna si el record està en el set
-   **set1 \| set2** Unió de sets
-   **set1 & set2** Intersecció de sets
-   **set1 - set2** Diferència de sets

Amés, un recordset no té elements repetits i permet accedir a recordsets
dins d\'ell. Per exemple:

``` python
>>> record.students.classrooms
```

Dona la llista de totes les classes de tots els estudiants i sense
repetir cap.

#### Programació funcional en l\'ORM 

Python té una serie de funcions que permeten iterar una llista i aplicar
una funció als elements. Les més utilitzades són map(), filter(),
reduce(), sort(), zip()\... ***Odoo*** treballa en recordsets, no llistes, i
té les seues funcions pròpies per a imitar aquestes:

-   **filtered()** Filtra el recordset de manera que sols tinga els
    records que complixen una condició.

``` python
records.filtered(lambda r: r.company_id == user.company_id)
records.filtered("partner_id.is_company")
```

-   **sorted()** Ordena segons uns funció, se defineix una funció lambda
    (key) que indica que s\'ordena per el camp name:

``` python
# sort records by name
records.sorted(key=lambda r: r.name)
records.sorted(key=lambda r: r.name, reverse=True)
```

-   **mapped()** Li aplica una funció a cada recordset i retorna un
    recordset amb els canvis demanats:

``` python
# returns a list of summing two fields for each record in the set
records.mapped(lambda r: r.field1 + r.field2)
# returns a list of names
records.mapped('name')
# returns a recordset of partners
record.mapped('partner_id')
# returns the union of all partner banks, with duplicates removed
record.mapped('partner_id.bank_ids')
```

Aquestes funcions són útils per a fer tècniques de [programació
funcional](https://docs.python.org/3.7/howto/functional.html)

#### Enviroment

L\'anomenat enviroment o **env** guarda algunes dades contextuals
interessants per a treballar amb l\'ORM, com ara el cursor a la base de
dades, l\'usuari actual o el context (que guarda algunes metadades).

Tots els recordsets tenen un enviroment accesible amb env. Quant volem
crear un recordset dins d\'un altre, podem usar env:

``` python
>>> self.env['res.partner']
res.partner
>>> self.env['res.partner'].search([['is_company', '=', True], ['customer', '=', True]])
res.partner(7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74)
```

El primer cas crea un recordset buit però que fa referència a
res.partner i es poden fer les funcions de l\'ORM que necessitem.

##### Context

El context és un diccionari de python que conté dades útils per a totes
les vistes i els mètodes. Les funcions d\'Odoo reben el context i el
consulten si cal. Context pot tindre de tot, però quasi sempre té al
menys el user ID, l\'idioma o la zona temporal. Quant ***Odoo*** va a
renderitzar una vista XML, consulta el context per veure si ha
d\'aplicar algun paràmetre.

``` python
print(self.env.context)
```

Al llarg de tot aquest manual utilitzem sovint paràmetres del context.
Aquests són els paràmetres que hem utilitzat en algun moment:

-   active_id : self.\_context.get(\'active_id\') es tracta de l\'id de
    l\'element del model que està en pantalla.
-   active_ids : Llista de les id seleccionats en un tree.
-   active_model : El model actual.
-   default\_`<field>`{=html} : En un action o en un one2many es pot
    assignar un valor per defecte a un field.
-   search_default\_`<filter>`{=html} : Per aplicar un filtre per
    defecte a la vista en un **action**.
-   group_by : Dins d\'un camp **filter** per a crear agrupacions en les
    vistes **search**.
-   graph_mode : En les vistes **graph**, aquest paràmetre canvia el
    **type**
-   context.get : En les vistes es pot treure algunes dades del context
    per a mostrar condicionalment o per als *domains*

El context va passant d\'un mètode a un altre o a les vistes i, de
vegades volem modificar-lo.

Imaginem que volem fer un botó que obriga un
wizard, però volem passar-li **paràmetres**
al wizard. En els botons i fields relacionals es pot especificar un
context:

``` xml
<button name="%(reserves.act_w_clients_bookings)d" type="action" string="Select bookings" context="{'b_fs':bookings_fs}"/>
```

Eixe action obre un wizard, que és un model transitori en el que podem
definir un field amb els continguts del context:

``` python
def _default_bookings(self):
         return self._context.get('b_fs')
bookings_fs = fields.Many2many('reserves.bookings',readonly=True, default=_default_bookings)
```

Aquest many2many tindrà els mateixos elements que el form que l\'ha
cridat. (Això és com el
**default\_** en els
One2many, però fet a mà)

També es pot utilitzar aquesta manera d\'enviar un recordset per un
context per al **domain** d\'un field Many2one o Many2many:

``` python
def _domain_bookings(self):
         return [('id','=',self._context.get('b_fs').ids)]
bookings_fs = fields.Many2many('reserves.bookings',readonly=True, domain=_default_bookings)
```

En ocasions necessitem especificar valors per defecte i filtres per
defecte en un **action**. Per exemple, quan implementem l\'herència,
volem que els nous registres que es facen en el nostre **action**
tinguem un valor per defecte. En el següent exemple, en la primera línia
és el que es sol fer en la Herència i en la segona estem
especificant un External ID amb **ref()**
dins d\'un eval.

``` python
        <field name="context">{'default_is_player': True, 'search_default_player_partner': 1}</field>
        <field name="context" eval="{'default_partner_id':ref('base.main_partner'), 'company_hide':False, 'default_company_id':ref('base.main_company'), 'search_default_my_bank':1}"/>
```

El context és un diccionari inmutable (frozendict) que no pot ser
alterat en funcions. no obstant, si volem modificar el context actual
per enviar-lo a un action o cridar a una funció d\'un model amb un altre
context, es pot fer amb
**[with_context](https://www.odoo.com/documentation/11.0/reference/orm.html#odoo.models.Model.with_context)**:

``` python
# current context is {'key1': True}
r2 = records.with_context({}, key2=True)
# -> r2._context is {'key2': True}
r2 = records.with_context(key2=True)
# -> r2._context is {'key1': True, 'key2': True}
```

Si és precís modificar el context es pot fer:

``` python
 self.env.context = dict(self.env.context)
 self.env.context.update({'key': 'val'})
```
o

``` python
 self = self.with_context(get_sizes=True)
 print self.env.context
```

Però no funciona més enllà del recordset actual. És a dir, no modifica
el context en el que s\'ha cridat.

Si el que volem és passar el valor d\'un field per context a un botó
dins d\'una \'subvista\', podem utilitzar el paràmetre **parent**, que
funciona tant en en **domain**, **attr**, com en context. Ací tenim un
exemple de tree dins d\'un field amb botons que envíen per context coses
del pare:

``` xml
 <field name="movies" >
    <tree>
        <field name="photo_small"/>
        <field name="name"/>
        <field name="score" widget='priority'/>
        <button name="book_it" string="Book it" type="object" context="{'b_client':parent.client,'b_day':parent.day}"/>
     </tree>
```

Podem passar el context per un action i el podem utilitzar en la vista,
ja que tenim l\'objecte **context** disponible en QWeb. Si, per exemple,
volem retornar un action que cride a una vista i un field tinga un
domain passat per context:

``` python
     return {
            'name': 'Travel wizard action',
            'type': 'ir.actions.act_window',
            'res_model': self._name,
            'res_id': self.id,
            'view_mode': 'form',
            'target': 'new',
            'context': dict(self._context, cities_available_context= (self.cities_available.city).ids),
        }
```

``` xml
           <field name="destiny"
                  domain = "[('id','in',context.get('cities_available_context',[]))]"
                  />
```

#### Mètodes de l\'ORM 

##### search()

A partir d\'un **domain** de Odoo, proporciona un recordset amb tots els
elements que coincideixen:

``` python
>>> # searches the current model
>>> self.search([('is_company', '=', True), ('customer', '=', True)])
res.partner(7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74)
>>> self.search([('is_company', '=', True)], limit=1).name
'Agrolait'
```

```{tip}
 Es pot obtindre la quantitat d'elements amb el mètode '''search_count()'''
```
``` python
Parameters
    args -- A search domain. Use an empty list to match all records.
    offset (int) -- number of results to ignore (default: none)
    limit (int) -- maximum number of records to return (default: all)
    order (str) -- sort string
    count (bool) -- if True, only counts and returns the number of matching records (default: False)
```

##### create()

Te dona un recordset a partir d\'una definició de varis fields:

``` python
>>> self.create({'name': "New Name"})
res.partner(78)
```

El mètode **create** s\'utilitza sovint per a ser sobreescrit en
herència per fer coses en el moment de la creació. Ací tenim un exemple
en el que modifiquem el **create** d\'un model per crear una instància
associada amb una imatge predefinida:

``` python
     @api.model
     def create(self, values):
        new_id = super(player, self).create(values)
        print values
        name_player = new_id.name
        img = self.env['mmog.fortress'].search([('name','=','f1')])[0].icon
        self.env['mmog.fortress'].create({'name':name_player+"-fortress",'level':1,'soldiers':100,'population':10,'food':1000,'integrity':100,'id_player':new_id.id,'icon':img})
        return new_id
```

##### write()

Escriu uns fields dins de tots els elements del recordset, no retorna
res:

``` python
self.write({'name': "Newer Name"})
```

**Escriure en un many2many**:

La manera més senzilla és passar una llista d\'ids. Però si ja
existeixen elements abans, necessitem uns codis especials (vegeu
[Odoo#Expressions](Odoo#Expressions "wikilink")):

Per exemple:

``` python
 self.sessions = [(4,s.id)] 
 self.write({'sessions':[(4,s.id)]})
 self.write({'sessions':[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_compact'),ref('fleet.vehicle_tag_senior')] )]})
```

##### browse()

A partir d\'una llista de ids, retorna un recordset.

``` python
>>> self.browse([7, 18, 12])
res.partner(7, 18, 12)
```

##### exists()

Retorna si un record en concret encara està en la base de dades.

``` python
if not record.exists():
    raise Exception("The record has been deleted")
o:
records.may_remove_some()
# only keep records which were not deleted
records = records.exists()
```

En el segon exemple, refresca un recordset amb aquells que encara
existixen.

##### ref()

Retorna un singleton a partir d\'un **External ID**

``` python
>>> env.ref('base.group_public')
res.groups(2)
```

##### ensure_one()

S\'asegura de que el record en concret siga un singleton.

``` python
records.ensure_one()
# is equivalent to but clearer than:
assert len(records) == 1, "Expected singleton"
```

##### unlink()

Esborra de la base de dades els elements del recordset actual.

Exemple de cóm sobreescriure el mètode unlink per a esborrar en cascada:

``` python
    def unlink(self):
        for x in self:
            x.catid.unlink()
        return super(product_uom_class, self).unlink()
```

**read()** Es tracta d\'un mètode de baix nivell per llegir un field en
concret dels records. És preferible emprar browse()

*\'name_search(name=*, args=None, operator=\'ilike\', limit=100)\'\'\' →
records Search for records that have a display name matching the given
name pattern when compared with the given operator, while also matching
the optional search domain (args).

This is used for example to provide suggestions based on a partial value
for a relational field. Sometimes be seen as the inverse function of
name_get(), but it is not guaranteed to be.

This method is equivalent to calling search() with a search domain based
on display_name and then name_get() on the result of the search.

**ids** Llista dels ids del recordset actual.

**sorted(key=None, reverse=False)** Retorna el recordset ordenat per un
criteri.

**display_name**. Aquest atribut, per defecte, mostra el field **name**
si està. Es pot sobreescriure `_compute_display_name` per mostrar un altre camp o mescla
d\'ells. També es pot canviar `_rec_name` per indicar un field distint de `name`.

**copy()** Crea una còpia del singleton i permet aportar nous valors per
als fields de la copia.

En els fields **One2many** no es pot copiar per defecte, però es pot dir
**copy=True**.

##### onchange

Si volem que un valor siga modificat en temps real quant modifiquem el
valor d\'un altre field sense encara haver guardat, podem usar els
mètodes **on_change**.

```{tip}
 Els camps '''computed''' ja tenen el seu propi onchange, per tant, no cal fer-lo
```

```{tip}
 Ha quedat "deprecated" retornar un domain https://github.com/odoo/odoo/pull/41918#issuecomment-824946980
```

En onchange es modifica el valor d\'un o més camps dirèctament i, si cal
un filtre o un missatge, es fa en el return:

``` python
return {
    'warning': {'title': "Warning", 'message': "What is this?", 'type': 'notification'},
}
```

Si el **type** és **notification** es mostrarà en una notificació, en un
altre cas, en un dialog. (Odoo 13)

Exemples:

``` python
# onchange handler
@api.onchange('amount', 'unit_price')
def _onchange_price(self):
    # set auto-changing field
    self.price = self.amount * self.unit_price
    # Can optionally return a warning and domains
    return {
        'warning': {
            'title': "Something bad happened",
            'message': "It was very bad indeed",
        }
    }

@api.onchange('seats', 'attendee_ids')
def _verify_valid_seats(self):
     if self.seats < 0:
         return {
             'warning': {
                 'title': "Incorrect 'seats' value",
                 'message': "The number of available seats may not be negative",
             },          }
     if self.seats < len(self.attendee_ids):
          return {
             'warning': {
                 'title': "Too many attendees",
                 'message': "Increase seats or remove excess attendees",
             },
         }
```

```{tip}
Si l'usuari s'equivoca introduint algunes dades, ***Odoo*** proporciona varies maneres d'evitar-lo: 
* Constraints
* onchange amb missatge d'error i restablint els valors originals
* Sobreescriptura del mètode write o create per comprovar coses abans de guardar 
```


##### Cron Jobs 

Cal crear un record en el model ir.cron, per exemple:

``` xml
        <record model="ir.cron" forcecreate="True" id="game.cron_update">
            <field name="name">Game: Cron Update</field>
            <field name="model_id" ref="model_game_player"/>
            <field name="state">code</field>
            <field name="code">model.update_resources()</field>
            <field name="user_id" ref="base.user_root"/>
            <field name="interval_number">1</field>
            <field name="interval_type">minutes</field>
            <field name="numbercall">-1</field>
            <field name="activity_user_type">specific</field>
            <field name="doall" eval="False" />
        </record>
```

I un mètode amb el \@api.model i aquests arguments:

``` python
    @api.model
    def update_resources(self):
        ...
```

**ir.cron** té un many2one amb **ir.actions.server** i, al ser creat,
crea l\'acció de servidor corresponent. És important ficar en el
manifest que depén de **mail**, ja que és un mòdul preinstal·lat que
hereta i afegeix camps a **ir.actions.server**.

<https://poncesoft.blogspot.com/2018/05/creacion-metodos-automatizados-en-odoo.html>
<https://webkul.com/blog/creating-cron-server-action-odoo-11/>
<https://odoo-development.readthedocs.io/en/latest/odoo/models/ir.cron.html>

### Els Decoradors 

Com es veu, abans de moltes funcions es fica \@api.depends,
\@api.multi\...

Els decoradors modifiquen la forma en la que és cridada la funció. Entre
altres coses, modifiquen el contingut de **self**, les vegades que se
crida i quant se crida.

-   **\@api.depends()** Aquest decorador crida a la funció sempre que el
    camp del que depén siga modificat. Encara que el camp diga
    *store=True*. Per defecte, **self** és un recordset, per tant, cal
    fer un for.
-   **\@api.model** S\'utilitza per a funcions que afecten al model i no
    als recordsets.
-   **\@api.constrains()** S\'utilitza per a comprovar les *constrains*.
    Self és un recordset. Com que quasi sempre es crida en un form,
    funciona si utilitzem self directament. Però cal fer for, ja que pot
    ser cridat en un recordset quant modifiquem camps en grup.
-   **\@api.onchange()** S\'executa cada vegada que modifiquem el field
    indicat en la vista. En aquest, com que es crida quant es modifica
    un form, sempre **self** serà un singleton. Però si fiquem un for no
    passa res.

### Càlculs en dates 

Odoo gestiona les dates com a strings. Per una altra banda, python té el
seu propi tipus de dades anomenat datetime, date i timedelta entre
altres. Això pot provocar dificultats per a fer cálculs en dates. Odoo
proporciona algunes ferramentes mínimes per facilitar aquesta tasca.

Primer de tot, anem a importar datetime:

``` python
from ***Odoo*** import models, fields, api
from datetime import datetime, timedelta
```

El primer que necessitem saber és cóm transformar de date o datetime
d'Odoo a python. En definitva, passar de string a datetime.

Tenim un field datetime declarat de la següent manera:

``` python
start_date = fields.Datetime()
```

En la base de dades guardarà un string amb el format: \'%Y-%m-%d
%H:%M:%S\'. Per tant, si volem transformar aquesta data en string a un
objecte datetime.datetime tenim que ejecutar el constructor de la classe
amb aquests paràmetres:

``` python
fmt = '%Y-%m-%d %H:%M:%S'
data = datetime.strptime(self.start_date,fmt)
```

És a dir, transforma un string en aquest format al tipus de dades
datetime.datetime oficial de python.

Per no tindre que especificar el format cada vegada, ***Odoo*** dona una
ferramenta més facil. La classe **fields.Datetime** té un mètode per
generar un datetime.datetime de un string:

``` python
data = fields.Datetime.from_string(self.start_date)
```

De la mateixa manera passa al contrari:

``` python
fmt = '%Y-%m-%d %H:%M:%S'
self.start_date = data.strftime(fmt)
vs
self.start_date = fields.Datetime.to_string(data)
```

**A continuació, anem a veure cóm incrementar una data en un temps:**

En el format d'Odoo (fields.Datetime) no es pot, cal passar a
datetime.datetime per sumar el temps i després tornar a passar a
fields.Datetime. Per sumar o restar temps a un datetime.datetime cal
utilitzar una classe anomenada datetime.timedelta. Aquesta classe
representa una duració o la diferència entre dues dates. Per exemple,
aquest constructor representa molt bé les opcions que es poden ficar per
crear un timedelta:

``` python
un_any = timedelta(weeks=40, days=84, hours=23, minutes=50, seconds=600) 
```

Aquest exemple d'Odoo mostra cóm afegir 3 dies a un field:

``` python
data=fields.Datetime.from_string(self.start_date)
data=data+timedelta(hours=3)
self.end_date=fields.Datetime.to_string(data)
```

O si es vol fer sols en mètodes python:

``` python
fmt = '%Y-%m-%d %H:%M:%S'
data = datetime.strptime(self.start_date,fmt)
data=data+timedelta(hours=3)
self.end_date=data.strftime(fmt)
```

**Ara anem a veure cóm calcular el temps que ha passat entre dues
dates:**

Solució amb **relativedelta**:

``` python
from datetime import datetime, timedelta
from dateutil.relativedelta import relativedelta

start=fields.Datetime.from_string(self.start_date)
end=fields.Datetime.from_string(self.end_date)

relative=relativedelta(start,end)
print r.years
print r.months
print r.days
print r
```

El problema és que dona la data per separat. No sol ser molt útil per a
Odoo on necessitem la diferència sols en dies, hores o minuts.

Solució sols amb **Datetime**:

``` python
from datetime import datetime
 
start=fields.Datetime.from_string(self.start_date)
end=fields.Datetime.from_string(self.end_date)
 
print (end-start).days * 24 * 60
print (end-start).total_seconds()/60/60/24
```

Solució amb **Unix timestamp**

``` python
d1_ts = time.mktime(d1.timetuple())
d2_ts = time.mktime(d2.timetuple())

print int(d2_ts-d1_ts) / 60
```

La solució és la mateixa, sols és per si necessiteu algun càlcul
intermedi que necessite la data en un Integer.

El resultat de restar dos datetime és un timedelta. Podem demanar els
dies i segons com en el relative delta, però amés té una funció per
traure els segons totals i després fer els càlculs que necessitem.

**Consultar si una data és anterior a una altra:**

Les dates en format Datetime o Date es poden comparar:

``` python
d3=fields.Datetime.from_string(self.d3)
d4=datetime.now()
if d3 < d4:
   print "La data és anterior"
```

També es pot calcular si és del mateix dia, sols cal transformar de
datetime a date:

``` python
d3=d3.date()
d4=d4.date()

if d3 == d4 :
   ….
```

Si volem saber si són del mateix més o any, es pot calcular la
diferència i veure si en dies és major o menor de 30, per exemple. Però
si volem major precisió, en aquest cas es recomana utilitar
relativedelta.


## Misc.

-   Si volem fer un print en colors, podem ficar un caracter de escape:
    \\033\[93m i \\033\[0m al final
-   Traure la menor potència de 2 major o igual a un número:
    <http://stackoverflow.com/a/14267557>

Distintes alertes:


Odoo pot mostrar distintes alertes en funció del que necessitem. Totes
estan en openerp.exceptions

Si entrem en el mode shell del debug podem executar aquest comandament:

    >>> help(openerp.exceptions)

Una vegada dins podem detectar:

`AccessDenied`\
`DeferredException`\
`QWebException`\
`RedirectWarning`\
`except_orm`\
`        AccessError`\
`        MissingError`\
`        UserError`\
`        ValidationError`

Normalment són utilitzats pel ***Odoo*** sense necessitat de que els cridem
nosaltres. Però en ocasion pot ser útil.

Per exemple, si volem mostrar un Warning perquè úsuari ha fet alguna
cosa mal. (Normalment es fa un onchange que ja pot tornar el warning)

``` python
from openerp import _
from openerp.exceptions import Warning
[...]
raise Warning(_('Alguna cosa ha fallat!'))
```

O si volem Donar opcions a l\'usuari amb RedirectWarning:

``` python
 action = self.env.ref('base.action_res_users')
 msg = _("You cannot create a new user from here.\n To create new user please go to configuration panel.")
 raise openerp.exceptions.RedirectWarning(msg, action.id, _('Go to the configuration panel'))
```

En aquest exemple, per al missatge, utilitza la barra baixa **\_()** per
a obtindre la traducció en cas de que existisca. **self.env.ref()**
retorna l\'objecte referit amb una id externa. En aquest cas, un action.

En el cas de les Constrains també s\'ha de llançar un Validation error.

Funcions lambda:


En moltes ocasions, cal cridar a alguna funció de l\'ORM o similar
passant com a paràmetre una funció lambda. La raó és que si passem una
variable, esta queda establerta en temps de càrrega i no es modifica. La
funció sempre recalcula.

La sintaxi de la funció lambda és:

``` python
a = lambda x,y: x*y
a(2,3)
6
```

On les primeres x,y són els arguments que rep la funció, després el que
calcula.

Cal recordar que les funcions lambda són de una sola línia de codi. Si
volem alguna cosa més sofisticada hem de cridar a una funció normal.


Si volem que el nostre mòdul tinga configuració podem afegir-la com a un
field més del model **res.control.settings**. Aquest ja s\'encarrega de
centralitzar opcions de configuració. Per a que aparega en el menú de
configuració també podem afegir-lo heretant en la vista:

``` python
class config(models.TransientModel):
    _inherit = 'res.config.settings'
    players = fields.Char(string='players',
                             config_parameter="expanse.players")


    def reset_universe(self):
        print("reset",self)
```

``` xml
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
                            <button type="object" name="reset_universe" string="Reset Universe"  class="btn-primary"/>
                        </div>

                    </div>
                </xpath>

            </field>
        </record>
```

Si en data-key posem el nom del mòdul, afegirà l\'icona al menú de
settings.
