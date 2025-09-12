---
title: 8.2. Definición de Campos (Fields)
description: Desarrollando módulos en Odoo. Fields
---

Hemos visto cómo se configura genéricamente un modelo, ahora vamos a profuncizan en la características de los campos que lo definen.

## Fields

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
from odoo import models, fields

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
### Campos normales

Estos son los campos para datos normales que proporciona Odoo:

- `Integer`
- `Char`
- `Text`
- `Date`: Muestra un calendario en la vista.
- `Datetime`
- `Float`
- `Boolean`
- `Html`: Guarda un texto, pero se representa de manera especial en el cliente.
- `Binary`: Para guardar, por ejemplo, imágenes. Utiliza codificación base64 al enviar los archivos al cliente. En realidad los guarda en **/var/lib/odoo/.local/share/Odoo/filestore** y la ruta a los archivos la indica la tabla **ir_attachment** junto con el id, nombre del campo y el modelo.
- `Image` (Odoo13): En el caso de imágenes, acepta los atributos **max_width** y **max_height** donde se puede indicar en píxeles que debe redimensionar la imagen a ese tamaño máximo.
- Sele`ction: Muestra un select con las opciones indicadas.

```python
type = fields.Selection([('1','Básico'),('2','Intermedio'),('3','Completado')])
aselection = fields.Selection(selection='a_function_name') # Se puede llamar a una función que define las opciones.
```

### Campos relacionales

Las relaciones entre los modelos (en definitiva, entre las tablas de la base de datos) también las simplifica el **ORM**. Así, las relaciones 1 a muchos se hacen en lo que Odoo llama ***Many2one*** y las relaciones Muchos a Muchos se hacen con ***Many2many***. Las relaciones muchos a muchos, en una base de datos relacional, implican una tercera tabla intermedia, pero en Odoo no tenemos que preocuparnos de estos detalles si no queremos, el mapeo de los objetos lo detectará y creará las tablas, claves y restricciones de integridad necesarias. Vamos a repasar uno a uno estos campos:

#### Reference

Una referencia arbitraria a un modelo y un campo.

```python
aref = fields.Reference([('model_name', 'String')])
aref = fields.Reference(selection=[('model_name', 'String')])
aref = fields.Reference(selection='a_function_name')

# Fragmento de test_new_api:
reference = fields.Reference(string='Documento relacionado', selection='_reference_models')
@api.model
def _reference_models(self):
    models = self.env['ir.model'].search([('state', '!=', 'manual')])
    return [(model.model, model.name)
            for model in models
            if not model.model.startswith('ir.')]
```

Los campos reference no son muy utilizados, ya que normalmente las relaciones entre modelos son siempre las mismas.

#### Many2one

Relación con otro modelo

```python
arel_id = fields.Many2one('res.users')
arel_id = fields.Many2one(comodel_name='res.users')
an_other_rel_id = fields.Many2one(comodel_name='res.partner', delegate=True)
```

En este caso:

    ----------              -----------
    | País   |  uno         |  Ciudad | 
    ---------- -----        -----------
    | * id   |     |        | * id    |
    | * name |     | muchos | * name  |
    ----------     ---------| * país  |
                            -----------

El código resultante sería:

```python
class ciudad(models.Model):
    _name = 'mon.ciutat'
    pais = fields.Many2one("mon.pais", string='País', ondelete='restrict')
```

**delegate** está en True para hacer que los campos del modelo apuntado sean accesibles desde el modelo actual.

También acepta **context** y **domain** como en la vista. Así queda disponible para todas las posibles vistas.

Otro argumento adicional es **ondelete** que permite definir el comportamiento al borrar el elemento referenciado: **set null**, **restrict** o **cascade**.

> `ondelete cascade` borra los hijos a nivel de PostgreSQL, pero no elimina en External Id, eso se hace en unlink(), pero no ejecuta unlink() de los hijos. Por tanto, si queremos que se eliminen por completo, hay que heredar el unlink del padre y añadir la llamada al de los hijos.

#### One2many

Inversa del Many2one. Necesita la existencia de un Many2one en el otro modelo:

```python
arel_ids = fields.One2many('res.users', 'arel_id')
arel_ids = fields.One2many(comodel_name='res.users', inverse_name='arel_id')
```

Un One2many funciona porque hay un many2one en el otro modelo. Así, siempre hay que especificar el nombre del modelo y el nombre del campo Many2one del modelo que hace referencia al actual, como se puede ver en el ejemplo.

En el ejemplo anterior, quedaría así:

```python
class pais(models.Model):
    _name = 'mon.pais'
    ciudades = fields.One2many('mon.ciutat', 'pais', string='Ciudades')
```

!!! Nota
        
    Es importante entender que el **One2many** no implica datos adicionales en la base de datos y siempre es calculado como un ''select'' en la base de datos donde el id del modelo actual coincida con el **Many2one** (clave foránea) del otro modelo. Esto hace que no tenga sentido hacer *One2many* computed o poner un domain para restringir los que se pueden añadir.

!!! Tip

    Los *One2many* pueden tener domain para no mostrar los que no cumplen una condición, esto no significa que no exista esa relación.

#### Many2many

Relación muchos a muchos.

```python
arel_ids = fields.Many2many('res.users')
arel_ids = fields.Many2many(
    comodel_name='res.users', # El modelo con el que se relaciona
    relation='table_name', # (opcional) el nombre de la tabla intermedia
    column1='col_name', # (opcional) el nombre en la tabla intermedia de la columna de este modelo
    column2='other_col_name')  # (opcional) el nombre de la columna del otro modelo.
```

El primer ejemplo suele funcionar directamente, pero si queremos tener más de una relación Many2many entre los dos mismos modelos, hay que usar la sintaxis completa donde especificamos el nombre de la relación y el nombre de las columnas que identifican los dos modelos. Recuerda que una relación Many2many implica una tabla intermedia y estamos especificando sus claves foráneas.

!!! Tip

    También es necesario especificar la tabla intermedia si se hace una relación **Many2many** al propio modelo.

    Un **Many2many** implica una tabla intermedia. Si queremos añadir atributos a esta relación, hay que crear explícitamente el modelo intermedio.

    El **many2many** puede ser ''computed'' y en el cálculo se puede ordenar o filtrar. Un Many2many computed no crea la tabla intermedia.


#### Related

Un campo de otro modelo, necesita una relación Many2one. Así se pueden aprovechar las funcionalidades de guardado, como búsquedas o referencias en funciones. En términos de bases de datos, un campo related rompe la tercera forma normal. Esto suele ser problemático, pero Odoo tiene mecanismos para que no pase nada. De todas formas, si nos preocupa esto, con store=False no guarda nada en la tabla.

```python
participant_nick = fields.Char(
    string='Nick name',
    store=True,
    related='partner_id.name'
```

Un campo related puede ser de cualquier tipo. Por ejemplo, many2one:

```python
sala = fields.Many2one(
    'cine.sala', 
    related='sessio.sala', 
    store=True, 
    readonly=True)
```

#### Many2oneReference

Un **Many2one** donde también se guarda el modelo al que hace referencia con el atributo: **model_field**. Más [info](https://www.odoo.com/documentation/18.0/developer/reference/backend/orm.html#pseudo-relational-fields)

#### One2one

Los campos **One2one** no existen en Odoo. Pero si queremos esta funcionalidad podemos usar varias técnicas:

-   Hacer dos campos Many2many y restringir con constrains que solo puede existir una relación. Problemas:
    -   En la vista no podemos poner un widget como en el Many2one y es complicado evitar relaciones cruzadas.
    -   Se puede poner un **limit** en la vista, pero seguirá comportándose como un Many2many.
-   Hacer dos Many2one y restringir con constrains o sql constrains que solo puede existir una relación mutua. (Hay que sobreescribir los métodos create y write para que se cree la asociación automáticamente). Problemas:
    -   Si sobreescribimos el write de ambos, se puede producir una llamada recursiva sin fin y es complicado evitar referencias cruzadas.
-   Hacer un Many2one y en el otro modelo un Many2one computed que busque en los del primer modelo. Para poder editar en ambos hay que hacer una función inversa para el campo computed. Esta es una de las opciones más elegantes. 

!!! Examle "One2one"

    ```python
    class orderline(models.Model):
        _name = 'sale.order.line'
        _inherit = 'sale.order.line'
        booking = fields.Many2one('reserves.bookings')
        
        _sql_constraints = [
        ('booking_uniq', 'unique(booking)', 'Ya existe otra línea de pedido para esta reserva'),
        ]

    class bookings(models.Model):
        _name = 'reserves.bookings'

        name = fields.Char()
        order_line = fields.Many2one('sale.order.line', compute='_get_order_line', inverse='_set_order_line')

        @api.multi
        def _get_order_line(self):
            for b in self:
                b.order_line = self.env['sale.order.line'].search([('booking.id','=',b.id)]).id

        @api.one
        def _set_order_line(self):
            o = self.order_line.id
            self.env['sale.order.line'].search([('id','=',o)]).write({'booking':self.id})
    ```

-   Hacer un Many2one y un One2many y restringir el máximo del One2many ([+info](https://stackoverflow.com/questions/32801526/how-to-create-a-one2one-relationship-in-odoo-8)). Problemas:
    -   Los mismos que en los dos many2many. Es más simple restringir las relaciones cruzadas.
-   Hacer una herencia múltiple (+info](http://blog.odoobiz.com/2014/10/openerp-one2one-relational-field-example.html)). Problemas:
    -   Esta es, en teoría, la forma más oficial de hacerlo, pero obliga a crear siempre la relación y los modelos en un orden determinado.

#### Filtros (Domains)

En ocasiones es necesario añadir un filtro en el código python para que un campo **relacional** no pueda tener ciertas referencias. El comportamiento del domain es diferente según el tipo de campo.

-   **Domain en Many2one**: Filtra los elementos del modelo referenciado que pueden ser elegidos para el campo:

```python
parent = fields.Many2one('game.resource', domain="[('template', '=', True)]")
```

-   **Domain en Many2many**: La lista de elementos a elegir se filtra según el domain:

```python
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

-   **Domain en One2many**: Al ser una relación que depende de otro Many2one, no se puede filtrar, si ponemos un domain, solo dejará de mostrar los que no cumplen el domain, pero no dejan de existir.


### Fields Computed 

Muchas veces queremos que el contenido de un campo sea calculado en el momento en que lo vamos a visualizar. Todos los tipos de campos pueden ser *computed* (calculados). Veamos algunos ejemplos:

!!! Example "Ejemplo"

    ```python
    # Este campo no se guarda en la base de datos 
    # y siempre se recalcula cuando ejecutamos una acción que lo muestra
    taken_seats = fields.Float(string="Plazas ocupadas", compute='_taken_seats')   

    # El decorador @api.depends() indica que se llamará a la función 
    # siempre que se modifiquen los campos seats y attendee_ids. 
    # Si no lo ponemos, solo se recalcula al recargar la acción.
    @api.depends('seats', 'attendee_ids')  
    def _taken_seats(self):          
        # El for recorre self, que es un recordset con todos los elementos del modelo mostrados 
        # por la vista. Si es un tree, serán todos los visibles y si es un form, será un singleton.
        for r in self:  
            # r es un singleton y se puede acceder a los campos como variables del objeto.      
            if not r.seats: 
                r.taken_seats = 0.0 
            else:
                r.taken_seats = 100.0 * len(r.attendee_ids) / r.seats
    ```

En este ejemplo se ve cómo el campo *float* `taken_seats` se calcula en una función privada `_taken_seats`. Es interesante observar el *for* porque recorre todas las instancias a las que hace referencia el modelo. Esta función solo se ejecutará una vez aunque tenga que calcular todos los elementos de una lista. Por eso, la propia función es la que debe iterar los elementos de **self**. **self** es un `recordset`, es decir, es como una lista en la que cada elemento es un registro del modelo. Si el campo *computed* se llama al entrar en un formulario, el recordset tendrá solo un elemento, pero si el campo *computed* se ve en una lista (tree), puede que sean varios registros. Es importante recordar hacer el **for record in self:** aunque pensemos que el campo *computed* solo lo utilizaremos en un formulario.

!!! Example "Ejemplo de campos **computed** de todos los tipos"

    ```python
    # -*- coding: utf-8 -*-

    from openerp import models, fields, api, tools
    from datetime import date, datetime

    class pruebas_computed(models.Model):
        _name = 'pruebas_computed.pruebas_computed'

        name = fields.Char()
        value = fields.Integer()
        image = fields.Binary(string="Imagen original")
        computedfloat = fields.Float(compute="_value_pc", store=True)
        computedchar = fields.Char(compute="_value_pc", store=False)
        medium_image = fields.Binary(compute="_redimensionar", store=True)
        small_image = fields.Binary(compute="_redimensionar", store=True)
        computedm2o = fields.Many2one('res.partner', compute="_value_pc", store=False)
        computedm2m = fields.Many2many(comodel_name='product.template', compute="_value_pc", store=False)
        computeddate = fields.Date(compute="_value_pc", store=False)
        computeddatetime = fields.Datetime(compute="_value_pc", store=False)
        
        description = fields.Text()

        @api.depends('value')
        def _value_pc(self):
            for r in self:
                r.computedfloat = float(r.value) / 100 
                r.computedchar = "(" + str(r.value) + ")"
                r.computedm2o = self.env['res.partner'].search([('id', '=', r.value)]).id # Many2one espera un id, que es un campo Integer. 
                print('\033[93m' + str(self.env['product.product'].search([('id', '>', r.value)]).ids) + '\033[0m')
                r.computedm2m = self.env['product.template'].search([('id', '>', r.value)]).ids # Many2many espera un array de ids o un recordset. 
                # El código comentado a continuación hace lo mismo, por si queremos hacer otras cosas dentro del for.
                # ids = []
                # for t in self.env['product.template'].search([('id','>',r.value)]):
                #     ids.append(t.id)
                # r.computedm2m = ids

                # r.computeddate = date.today() # Esto depende de Python
                r.computeddate = fields.date.today() # Recomendamos este, ya que es propio de la clase fields de Odoo
                # r.computeddate = datetime.now()
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

    ([Código completo](https://github.com/xxjcaxx/SGE-Odoo-2016-2017/tree/master/proves_computed))

!!! Note

    En el apartado del `controlador` se explican más detalles de las funciones en python-odoo.

#### Buscar y escribir en campos computed

Con **api.depends** podemos hacer que los campos calculados puedan ser buscados o referenciados desde otros modelos, ya que podemos indicar que sí se guarden en la base de datos. Si se guarda en la base de datos, no se recalcula hasta que no cambia el contenido del *field* del que depende. Pero si el campo calculado no depende de valores estáticos de otros *fields* y/o necesitamos que siempre se calcule, no tenemos muchas opciones elegantes. Una de ellas puede ser hacer dos campos, uno calculado **store=False** y otro no, y hacer un *write* en la función. Otra posibilidad es hacer una función pública que pueda ser llamada desde otro modelo. La más elegante, aunque no siempre funciona, es utilizar la opción **search** y asignarle una función que debe retornar un dominio de búsqueda. El problema es que no acepta mucha complejidad, ya que supone una búsqueda por toda la base de datos y puede ser muy ineficiente.

Por defecto no se puede escribir en un campo *computed*. No tiene mucho sentido en la mayoría de los casos, ya que es un campo que depende de otros. Pero puede ser que, a veces, queramos escribir el resultado y que modifique el campo origen. Imaginemos, por ejemplo, que sabemos el precio final y queremos que calcule el precio sin IVA. Para hacerlo, la mejor manera es crear una función y hacer que esté en la opción **inverse**.

!!! Example "Ejemplo:"

    ```python
    preu = fields.Float('Precio', compute="_get_price", search='_search_price', inverse='_set_price')

    @api.depends('pelicula', 'descuento')
    def _get_price(self):
        for r in self:
            price = r.pelicula.precio
            price = price - (price * r.descuento / 100)
            r.preu = price

    def _search_price(self, operator, value): # De momento este search solo es para ==
        precios = self.search([]).mapped(lambda e: [e.id, e.pelicula.precio - (e.pelicula.precio * e.descuento / 100)]) # Un buen ejemplo de mapped en lambda
        print(precios)
        p = [num[0] for num in precios if num[1] == value]  # condición if en una lista python sin hacer un for (list comprehension)
        # también se puede probar en un filter() de python
        print(p)
        # p es una lista de las id que ya cumplen la condición, por tanto solo hay que hacer que la id esté en la lista.
        return [('id', 'in', p)]

    def _set_price(self):
        self.pelicula.precio = self.preu  # Esto es un ejemplo, pero está mal, ya que modificas el precio de la peli en todas las sesiones
    ```

Documentación oficial: [https://www.odoo.com/documentation/master/developer/reference/backend/orm.html](https://www.odoo.com/documentation/master/developer/reference/backend/orm.html)

### Valores por defecto

En ***Odoo*** es muy fácil definir valores por defecto, ya que es un argumento más en el constructor de los campos:

```python
name = fields.Char(default="Desconocido")
user_id = fields.Many2one('res.users', default=lambda self: self.env.user)
start_date = fields.Date(default=fields.Date.today())
active = fields.Boolean(default=True)
def compute_default_value(self):
    return self.get_value()
a_field = fields.Char(default=compute_default_value)
```

Si queremos, por ejemplo, poner la fecha del momento de crear, no podemos hacer esto:

```python
start_date = fields.Date(default=fields.Date.today())  # INCORRECTO
```

Porque calcula la fecha en el momento de actualizar el módulo, no al crear el elemento en el modelo. Hay que hacer:

```python
start_date = fields.Date(default=lambda self: fields.Date.today())  # CORRECTO
```

o

```python
start_date = fields.Datetime(default=lambda self: fields.Datetime.now()) # CORRECTO
```

El valor por defecto no puede depender de un campo que se está creando en ese momento. En ese caso se puede utilizar un **on_change**.

En caso de tener muchos valores por defecto o que dependan del contexto, se puede utilizar la función **default_get** que ya tienen los modelos.

```python
@api.model
def default_get(self, default_fields):
    result = super(SelectSalePrice, self).default_get(default_fields)
    if self._context.get('default_picking_id') is not None:
        result['picking_id'] = self._context.get('default_picking_id')
    return result
```

Lo que hace esta función es un poco avanzado de momento, ya que hace uso del `context` y la herencia para añadir un valor por defecto al diccionario que retorna esta función en la clase *Model*.

### Restricciones (constrains)

Los objetos pueden incorporar, de forma opcional, restricciones de integridad, adicionales a las de la base de datos. ***Odoo*** valida estas restricciones en las modificaciones de datos y, en caso de violación, muestra una pantalla de error.

```python
from odoo.exceptions import ValidationError

@api.constrains('age')
def _check_something(self):
    for record in self:
        if record.age > 20:
            raise ValidationError("El registro es demasiado antiguo: %s" % record.age)
    # todos los registros pasaron la prueba, no retornar nada
```

En ocasiones, cuando tenemos claro cómo haríamos esta restricción en SQL, tal vez nos resulte más interesante hacer una restricción de base de datos con una **sql constraint**. Estas se definen con 3 strings **(name, sql_definition, message)**. Por ejemplo:

```python
_sql_constraints = [
    ('name_uniq', 'unique(name)', 'Mensaje de advertencia personalizado'),
    ('contact_uniq', 'unique(contact)', 'Mensaje de advertencia personalizado')
]
```

En este caso, se trata de una restricción de unicidad, que es más sencilla de implementar que realizar una búsqueda en Python.

