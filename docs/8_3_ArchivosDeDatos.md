---
title: 8.3. Archivos de Datos
description: Desarrollando máodulos en Odoo
---

## Archivos de datos

Cuando desarrollamos un módulo en Odoo, podemos definir datos que se almacenarán en la base de datos. Estos datos pueden ser necesarios para el funcionamiento del módulo, para demostración o incluso formar parte de la vista.

!!! Tip

    Algunos módulos solo existen para insertar datos en Odoo.


Todos los archivos de datos son en formato XML y tienen una estructura como la siguiente:

```xml
<odoo>
    <record model="{nombre_modelo}" id="{identificador_registro}">
        <field name="{nombre_campo}">{valor}</field>
    </record>
</odoo>
```

Dentro de las etiquetas **odoo** podemos encontrar una etiqueta **record** por cada fila de la tabla que queremos introducir. Es necesario especificar el modelo y el id. El **id** es un identificador externo, que no tiene por qué coincidir con la clave primaria que el ORM utilizará después. Cada **field** tendrá un nombre y un valor.

### Identificadores Externos (External Ids)

Todos los registros de la base de datos tienen un identificador único en su tabla, el **id**, que es un número autoincremental asignado por la base de datos. Sin embargo, si queremos referenciarlo en archivos de datos u otros lugares, no siempre conocemos ese id. La solución de Odoo son los **Identificadores Externos**. Se trata de una tabla que relaciona cada id de cada tabla con un nombre. Es el modelo **ir.model.data**. Para encontrarlos, accede a:

`Ajustes > Técnico > Secuencias e identificadores > Identificadores externos`

Ahí encontrarás la columna **Complete ID**.

Para encontrar los *id* al crear archivos de demostración o de datos, podemos ir al menú, pero esos ids cambian de una instalación a otra. Por tanto, es recomendable utilizar los **external id**. Para obtenerlo, puedes activar el modo desarrollador y abrir el menú **Ver metadatos**.

En los datos de demo, los **external ids** se utilizan para no depender de los ids, que pueden variar al ser autoincrementales. Para que funcione, hay que usar el atributo **ref**:

```xml
<field name="product_id" ref="product.product1"/>
```

!!! Tip

    Se recomienda usar el atributo `id` en el record, aunque no sobrescribe el id real, sirve para declarar el External Id y es más fácil referenciarlo después.

> Ver también la función **ref()** del ORM.

### Expresiones

A veces queremos que los campos se calculen cada vez que se actualiza el módulo. Esto se puede hacer con el atributo **eval**, que evalúa una expresión de Python.

```xml
<field name="date" eval="(datetime.now()+timedelta(-1)).strftime('%Y-%m-%d')"/>
<field name="product_id" eval="ref('product.product1')"/> <!-- Equivalente al ejemplo anterior -->
<field name="price" eval="ref('product.product1').price"/>
<field name="avatar" model="school.template" eval="obj().env.ref('school.template_student1').image" />
```

Para los campos *x2many*, se puede usar eval para asignar una lista de elementos:

```xml
<field name="tag_ids" eval="[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_compact'),ref('fleet.vehicle_tag_senior')])]" />
```

Observa que hemos pasado una tupla con un 6, un 0 y una lista de refs. Las tuplas pueden ser:

- (0, _, {'field': value}): Crea un nuevo registro y lo vincula.
- (1, id, {'field': value}): Actualiza los valores en un registro ya vinculado.
- (2, id, _): Desvincula y elimina el registro.
- (3, id, _): Desvincula pero no elimina el registro de la relación.
- (4, id, _): Vincula un registro ya existente.
- (5, _, _): Desvincula pero no elimina todos los registros vinculados.
- (6, _, [ids]): Reemplaza la lista de registros vinculados.

### Datos para campos Binary e Image

Algunos datos como imágenes o archivos pueden incluirse en los registros. Hay dos formas:

- Convertir el archivo a Base64 y pegar el resultado en el campo.
- Añadir el atributo `type="base64"` y el atributo `file="modulo/demo/archivo"`.

```xml
<field name="image_1920" type="base64" file="ejemplo/demo/caritas/1000.jpg"/>
```

Observa que la ruta parte desde el directorio del módulo.

### Eliminar registros

Con la etiqueta **delete** se pueden especificar los elementos a eliminar usando el external ID o mediante una búsqueda:

```xml
<delete model="cine.sessio" id="sessio_cine1_1"></delete>
```

!!! Danger "Cuidado"

    Si falla la actualización con datos de demo, es posible que Odoo desactive la posibilidad de volver a instalarlos. Esto es el campo demo de ir.module.module, que es de solo lectura, por lo que hay que modificarlo manualmente en la base de datos:

    `update ir_module_module set demo = 't' where name='school';`

Más información: [https://www.odoo.com/documentation/master/developer/reference/backend/data.html](https://www.odoo.com/documentation/master/developer/reference/backend/data.html)

Aquí tienes el fragmento corregido, traducido al castellano y formateado correctamente en Markdown:

---

