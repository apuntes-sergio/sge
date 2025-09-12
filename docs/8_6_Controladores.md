---
title: 8.6. Controladores
description: Desarrollando máodulos en Odoo. Controladores
---

## El controlador

Ya hemos mencionado parte del controlador al hablar de los campos **computed**. Sin embargo, es importante destacar las facilidades que proporciona ***Odoo*** para evitar tener que acceder directamente a la base de datos.

La capa **ORM** de Odoo ofrece métodos que se encargan del mapeo entre los objetos Python y las tablas de PostgreSQL. Así, disponemos de métodos para crear, modificar, eliminar y buscar registros en la base de datos.

En ocasiones, puede ser necesario alterar la acción automática de búsqueda, creación, modificación o eliminación facilitada por ***Odoo***, y deberemos sobreescribir los métodos correspondientes en nuestras clases.

Como desarrolladores en el framework de Odoo, debemos conocer los métodos proporcionados por la capa ORM y dominar el diseño de métodos para:

- Definir campos funcionales en el diseño del modelo.
- Definir la acción a ejecutar al modificar el contenido de un campo en una vista formulario (`@api.onchange`).
- Alterar las acciones automáticas de búsqueda, creación, modificación y eliminación de recursos.

Una última consideración a tener en cuenta al escribir métodos y funciones en ***Odoo*** es que los textos de los mensajes incluidos deben ser traducibles. Para ello, deben introducirse con la sintaxis `_(‘texto’)` y el archivo `.py` debe contener `from odoo.tools.translate import _` en la cabecera.

### API del ORM

!!! tip "Interactuar en la terminal"

    ```bash
    $ odoo shell -d castillo -u containers
    ```

    Observa cómo hemos añadido el parámetro `shell`. Las operaciones realizadas en la terminal **no son persistentes** en la base de datos hasta que se ejecuta `self.env.cr.commit()`. Dentro de la terminal, puedes obtener ayuda de los métodos de Odoo con `help()`, por ejemplo: `help(tools.image)`.

    También puedes arrancar Odoo en modo shell sin interferir con la instancia en ejecución redefiniendo los puertos:

    ```bash
    $ odoo shell -c /ruta/a/odoo.conf --xmlrpc-port 8888 --longpolling-port 8899
    ```

Un método creado dentro de un modelo actúa sobre todos los elementos del modelo que estén activos en el momento de llamar al método. Si es una vista tipo árbol (*tree*), serán varios; si es un formulario (*form*), solo uno. En cualquier caso, se trata de una "lista" de elementos llamada **recordset**.

La interacción con los modelos en el controlador se realiza mediante los llamados **recordsets**, que son colecciones de objetos sobre un modelo. Si iteramos sobre un recordset, obtenemos los **singletons**, que son objetos individuales correspondientes a cada fila en la base de datos.

```python
def do_operation(self):
    print(self)  # => a.model(1, 2, 3, 4, 5)
    for record in self:
        print(record)  # => a.model(1), luego a.model(2), luego a.model(3), ...
```

Podemos acceder a todos los campos de un modelo siempre que estemos en un singleton, no en un recordset:

```python
>>> record.name
'Nombre de ejemplo'
>>> record.company_id.name
'Nombre de la empresa'
>>> record.name = "Bob"
```

Intentar leer o escribir un campo en un recordset dará un error. Acceder a un campo **many2one**, **one2many** o **many2many** devolverá un recordset.

### Operaciones con conjuntos (Set operations)

Los recordsets pueden combinarse mediante operaciones típicas de conjuntos:

- `record in set`: Devuelve si el registro está en el conjunto.
- `set1 | set2`: Unión de conjuntos.
- `set1 & set2`: Intersección de conjuntos.
- `set1 - set2`: Diferencia de conjuntos.

Además, un recordset no tiene elementos repetidos y permite acceder a recordsets dentro de él. Por ejemplo:

```python
>>> record.students.classrooms
```

Devuelve la lista de todas las clases de todos los estudiantes, sin repetir ninguna.

#### Programación funcional en el ORM

Python dispone de varias funciones para iterar listas y aplicar funciones a sus elementos, como `map()`, `filter()`, `reduce()`, `sort()`, `zip()`, etc. Sin embargo, ***Odoo*** trabaja con *recordsets* en lugar de listas y proporciona sus propios métodos para imitar estas operaciones:

- **filtered()**: Filtra el *recordset* para que solo contenga los registros que cumplen una condición.

    ```python
    records.filtered(lambda r: r.company_id == user.company_id)
    records.filtered("partner_id.is_company")
    ```

- **sorted()**: Ordena los registros según una función, normalmente una función lambda que indica el campo por el que ordenar.

    ```python
    # Ordenar registros por el campo 'name'
    records.sorted(key=lambda r: r.name)
    records.sorted(key=lambda r: r.name, reverse=True)
    ```

- **mapped()**: Aplica una función a cada registro del *recordset* y devuelve una lista con los resultados.

    ```python
    # Devuelve una lista sumando dos campos para cada registro
    records.mapped(lambda r: r.field1 + r.field2)
    # Devuelve una lista de nombres
    records.mapped('name')
    # Devuelve un recordset de partners
    record.mapped('partner_id')
    # Devuelve la unión de todos los bancos de los partners, sin duplicados
    record.mapped('partner_id.bank_ids')
    ```

Estas funciones son útiles para aplicar técnicas de [programación funcional](https://docs.python.org/3.7/howto/functional.html).

---

#### Entorno (*Environment*)

El llamado *environment* o **env** almacena información contextual relevante para trabajar con el ORM, como el cursor de la base de datos, el usuario actual o el contexto (que contiene metadatos).

Todos los *recordsets* tienen acceso al *environment* mediante `env`. Cuando queremos crear un *recordset* dentro de otro, podemos usar `env`:

```python
>>> self.env['res.partner']
res.partner
>>> self.env['res.partner'].search([['is_company', '=', True], ['customer', '=', True]])
res.partner(7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74)
```

El primer caso crea un *recordset* vacío pero referenciado al modelo `res.partner`, sobre el que se pueden ejecutar las funciones del ORM.

##### Contexto

El contexto es un diccionario de Python que contiene datos útiles para todas las vistas y métodos. Las funciones de Odoo reciben el contexto y lo consultan si es necesario. El contexto suele incluir, al menos, el ID de usuario, el idioma o la zona horaria. Cuando ***Odoo*** va a renderizar una vista XML, consulta el contexto para ver si debe aplicar algún parámetro.

```python
print(self.env.context)
```

Algunos parámetros habituales en el contexto:

- `active_id`: ID del elemento del modelo que está en pantalla.
- `active_ids`: Lista de los IDs seleccionados en una vista tipo árbol.
- `active_model`: El modelo actual.
- `default_<campo>`: Permite asignar un valor por defecto a un campo en una acción o en un One2many.
- `search_default_<filtro>`: Aplica un filtro por defecto en la vista de una acción.
- `group_by`: Para crear agrupaciones en vistas de tipo *search*.
- `graph_mode`: En vistas *graph*, cambia el tipo de gráfico.
- `context.get`: Permite extraer datos del contexto en vistas o para *domains*.

El contexto se transmite entre métodos y vistas, y a veces queremos modificarlo. Por ejemplo, para pasar parámetros a un *wizard* desde un botón:

```xml
<button name="%(reserves.act_w_clients_bookings)d" type="action" string="Seleccionar reservas" context="{'b_fs':bookings_fs}"/>
```

En el *wizard* (modelo transitorio), podemos definir un campo con el valor recibido por contexto:

```python
def _default_bookings(self):
    return self._context.get('b_fs')
bookings_fs = fields.Many2many('reserves.bookings', readonly=True, default=_default_bookings)
```

También se puede usar el contexto para definir un *domain* en un campo Many2one o Many2many:

```python
def _domain_bookings(self):
    return [('id', '=', self._context.get('b_fs').ids)]
bookings_fs = fields.Many2many('reserves.bookings', readonly=True, domain=_domain_bookings)
```

Para modificar el contexto actual y enviarlo a una acción o llamar a una función de un modelo con otro contexto, se puede usar [`with_context`](https://www.odoo.com/documentation/11.0/reference/orm.html#odoo.models.Model.with_context):

```python
# El contexto actual es {'key1': True}
r2 = records.with_context({}, key2=True)
# -> r2._context es {'key2': True}
r2 = records.with_context(key2=True)
# -> r2._context es {'key1': True, 'key2': True}
```

Si es necesario modificar el contexto:

```python
self.env.context = dict(self.env.context)
self.env.context.update({'key': 'val'})
```
o

```python
self = self.with_context(get_sizes=True)
print(self.env.context)
```

Esto solo afecta al *recordset* actual y no modifica el contexto global.

En acciones, el contexto puede pasarse y utilizarse en la vista, ya que está disponible en QWeb. Por ejemplo, para pasar un *domain* por contexto a un campo en una vista:

```python
return {
    'name': 'Travel wizard action',
    'type': 'ir.actions.act_window',
    'res_model': self._name,
    'res_id': self.id,
    'view_mode': 'form',
    'target': 'new',
    'context': dict(self._context, cities_available_context=(self.cities_available.city).ids),
}
```

```xml
<field name="destiny"
       domain="[('id','in',context.get('cities_available_context',[]))]"
/>
```

---

#### Métodos del ORM

##### search()

A partir de un **domain** de Odoo, devuelve un *recordset* con todos los elementos que cumplen la condición:

```python
# Busca en el modelo actual
self.search([('is_company', '=', True), ('customer', '=', True)])
self.search([('is_company', '=', True)], limit=1).name
```

!!! tip
    Puedes obtener la cantidad de elementos con el método `search_count()`.

Parámetros principales:

- `args`: Dominio de búsqueda. Usa una lista vacía para todos los registros.
- `offset`: Número de resultados a ignorar (por defecto: ninguno).
- `limit`: Máximo de registros a devolver (por defecto: todos).
- `order`: Cadena de ordenación.
- `count`: Si es `True`, solo cuenta y devuelve el número de registros encontrados.

##### create()

Crea un registro a partir de un diccionario de campos:

```python
self.create({'name': "Nuevo Nombre"})
```

El método **create** suele sobreescribirse en herencia para realizar acciones adicionales al crear un registro.

##### write()

Escribe valores en los campos de todos los registros del *recordset*:

```python
self.write({'name': "Nombre actualizado"})
```

Para campos Many2many, se pueden usar códigos especiales para modificar la relación (ver [Expresiones en Odoo](8_3_ArchivosDeDatos.md#expresiones)):

```python
self.sessions = [(4, s.id)]
self.write({'sessions': [(4, s.id)]})
self.write({'sessions': [(6, 0, [ref('vehicle_tag_leasing'), ref('fleet.vehicle_tag_compact'), ref('fleet.vehicle_tag_senior')])]})
```

##### browse()

A partir de una lista de IDs, devuelve un *recordset*:

```python
self.browse([7, 18, 12])
```

##### exists()

Devuelve si un registro sigue existiendo en la base de datos:

```python
if not record.exists():
    raise Exception("El registro ha sido eliminado")
# O para refrescar un recordset:
records = records.exists()
```

##### ref()

Devuelve un singleton a partir de un **External ID**:

```python
env.ref('base.group_public')
```

##### ensure_one()

Asegura que el *recordset* contiene un solo registro (singleton):

```python
records.ensure_one()
# Equivalente a:
assert len(records) == 1, "Expected singleton"
```

##### unlink()

Elimina de la base de datos los registros del *recordset* actual.

Ejemplo de sobreescritura para eliminar en cascada:

```python
def unlink(self):
    for x in self:
        x.catid.unlink()
    return super(product_uom_class, self).unlink()
```

##### Otros métodos y atributos útiles

- **read()**: Método de bajo nivel para leer campos concretos. Se recomienda usar `browse()`.
- **name_search()**: Busca registros cuyo nombre visible coincida con un patrón.
- **ids**: Lista de IDs del *recordset* actual.
- **sorted(key=None, reverse=False)**: Devuelve el *recordset* ordenado por un criterio.
- **display_name**: Por defecto, muestra el campo `name`. Se puede sobreescribir `_compute_display_name` o cambiar `_rec_name` para mostrar otro campo.
- **copy()**: Crea una copia del singleton, permitiendo nuevos valores para los campos. En campos One2many, hay que indicar `copy=True` para permitir la copia.

---

##### onchange

Si queremos que un valor se modifique en tiempo real al cambiar otro campo (sin guardar aún), podemos usar los métodos **onchange**.

!!! tip
    Los campos *computed* ya tienen su propio *onchange*, por lo que no es necesario definirlo aparte.

!!! tip
    Retornar un *domain* en *onchange* está deprecado en versiones recientes de Odoo.

En *onchange* se modifica el valor de uno o más campos directamente y, si es necesario, se puede devolver un aviso o un filtro:

```python
return {
    'warning': {'title': "Aviso", 'message': "Algo ha ocurrido", 'type': 'notification'},
}
```

Ejemplo de *onchange*:

```python
@api.onchange('amount', 'unit_price')
def _onchange_price(self):
    self.price = self.amount * self.unit_price
    return {
        'warning': {
            'title': "Algo ha ocurrido",
            'message': "Revisa los valores introducidos",
        }
    }

@api.onchange('seats', 'attendee_ids')
def _verify_valid_seats(self):
    if self.seats < 0:
        return {
            'warning': {
                'title': "Valor incorrecto",
                'message': "El número de plazas no puede ser negativo",
            },
        }
    if self.seats < len(self.attendee_ids):
        return {
            'warning': {
                'title': "Demasiados asistentes",
                'message': "Aumenta las plazas o elimina asistentes",
            },
        }
```

!!! tip
    Para evitar errores de usuario, Odoo ofrece varias opciones:
    - Constraints
    - *onchange* con mensaje de error y restablecimiento de valores
    - Sobrescritura de los métodos `write` o `create` para validar antes de guardar

---

##### Tareas programadas (*Cron Jobs*)

Para crear tareas automáticas, hay que crear un registro en el modelo `ir.cron`:

```xml
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

Y un método decorado con `@api.model`:

```python
@api.model
def update_resources(self):
    ...
```

El modelo **ir.cron** tiene un Many2one con **ir.actions.server** y, al crearse, genera la acción de servidor correspondiente. Es importante declarar la dependencia del módulo `mail` en el manifest, ya que añade campos a `ir.actions.server`.

Más información:
- [Creación de métodos automatizados en Odoo](https://poncesoft.blogspot.com/2018/05/creacion-metodos-automatizados-en-odoo.html)
- [Creating cron server action Odoo 11](https://webkul.com/blog/creating-cron-server-action-odoo-11/)
- [Odoo ir.cron documentation](https://odoo-development.readthedocs.io/en/latest/odoo/models/ir.cron.html)

---

### Decoradores en Odoo

Los decoradores modifican la forma en que se llama a una función, alterando el contenido de `self`, el número de llamadas y el momento de ejecución.

- **@api.depends()**: Llama a la función siempre que el campo del que depende se modifica. Si el campo tiene `store=True`, también se recalcula. Por defecto, `self` es un *recordset*, por lo que hay que iterar.
- **@api.model**: Para funciones que afectan al modelo y no a los *recordsets*.
- **@api.constrains()**: Para comprobar restricciones. `self` es un *recordset*.
- **@api.onchange()**: Se ejecuta cada vez que se modifica el campo indicado en la vista. `self` suele ser un singleton.

---

### Cálculos con fechas

Odoo gestiona las fechas como cadenas de texto, mientras que Python tiene sus propios tipos de datos (`datetime`, `date`, `timedelta`). Para facilitar los cálculos, Odoo proporciona utilidades adicionales.

Importar los módulos necesarios:

```python
from odoo import models, fields, api
from datetime import datetime, timedelta
```

#### Conversión entre string y datetime

Supongamos que tenemos un campo:

```python
start_date = fields.Datetime()
```

Para convertir de string a `datetime`:

```python
fmt = '%Y-%m-%d %H:%M:%S'
data = datetime.strptime(self.start_date, fmt)
```

O usando la utilidad de Odoo:

```python
data = fields.Datetime.from_string(self.start_date)
```

Para convertir de `datetime` a string:

```python
self.start_date = fields.Datetime.to_string(data)
```

#### Sumar o restar tiempo a una fecha

Para sumar tiempo:

```python
data = fields.Datetime.from_string(self.start_date)
data = data + timedelta(hours=3)
self.end_date = fields.Datetime.to_string(data)
```

#### Calcular la diferencia entre dos fechas

Con `relativedelta`:

```python
from dateutil.relativedelta import relativedelta

start = fields.Datetime.from_string(self.start_date)
end = fields.Datetime.from_string(self.end_date)

relative = relativedelta(end, start)
print(relative.years)
print(relative.months)
print(relative.days)
```

O solo con `datetime`:

```python
start = fields.Datetime.from_string(self.start_date)
end = fields.Datetime.from_string(self.end_date)

print((end - start).days * 24 * 60)
print((end - start).total_seconds() / 60 / 60 / 24)
```

#### Comparar fechas

Las fechas en formato `Datetime` o `Date` pueden compararse directamente:

```python
d3 = fields.Datetime.from_string(self.d3)
d4 = datetime.now()
if d3 < d4:
    print("La fecha es anterior")
```

Para comparar solo la fecha (sin hora):

```python
if d3.date() == d4.date():
    ...
```

---

