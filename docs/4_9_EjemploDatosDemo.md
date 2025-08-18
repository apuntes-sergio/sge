---
---
title: 4.9. Datos de demostración  
description: Ejemplo guiado de desarrollo de módulo de gestión de proyectos en Odoo
---

**POR REVISAR**

En este apartado vamos a ver cómo añadir **datos de demo** a nuestra aplicación en Odoo. Nuestra aplicación, que no tenía datos, acaba de introducir un ejemplo para un desarrollador, un proyecto y un sprint. Lo que queremos es que, cuando se instale nuestro módulo, se generen una serie de datos de demo automáticamente.

## Casilla "Datos de demo" en la BDD

Recordad que, para que esto funcione, cuando creamos la base de datos debemos marcar la casilla de **datos de demo**.  
Por ejemplo, si estamos en `localhost` en la URL web database selector, al crear una base de datos, si marcamos la casilla de datos de demo, cualquier módulo que instalemos se instalará con datos de demo. Si no la marcamos, la aplicación, independientemente de que tenga desarrollados esos datos demo, no los instalará.

## Cómo generar nuestros datos de demo

### 1. Archivo `__manifest__.py`

Hay un fichero, el manifiesto (`__manifest__.py`), que en la última sección tiene la parte de datos de demo, que solo serán cargados si estamos en el modo demo.  
Por defecto, se cargan ficheros XML. Normalmente generamos uno por cada uno de los modelos.  
En este caso, el fichero `demo.xml` está en la carpeta `demo` y dentro tenemos el registro básico en XML.

#### Ejemplo de registro básico en XML

```xml
<record id="dev_1" model="res.partner">
  <field name="name">Nombre Desarrollador</field>
  <field name="access_code">12345678A</field>
  <field name="is_dev">True</field>
</record>
```

El ID debe ser único. No podemos tener un contacto u objeto de `res.partner` con ese mismo ID.  
Además, el campo `access_code` tiene restricciones: debe cumplir un patrón (ocho dígitos y una letra) y ser único (restricción SQL de unicidad).

---

## Generación automática de datos de demo

Si solo tenemos que generar un par de registros, podemos hacerlo manualmente. Pero para generar muchos, normalmente se hace automáticamente.

### 2. Datos aleatorios con [Mockaroo](https://mockaroo.com/)

Mockaroo permite generar datos aleatorios configurando los campos que queremos.  
Por ejemplo, para 10 desarrolladores:

- **ID**: generado aleatoriamente
- **name**: tipo `Full Name`
- **is_dev**: siempre `True`
- **access_code**: patrón de 8 dígitos y una letra mayúscula (`########^`)

Configuramos Mockaroo para generar 10 filas en formato CSV (sin cabecera) y descargamos el archivo.

---

### 3. Script en Python para generar datos de demo

Queremos convertir el CSV en un fichero `demo.xml` válido.  
Creamos un script en Python, por ejemplo `data_gen.py`, que:

- Lee el fichero CSV
- Escribe en el directorio `demo` un XML con la estructura correcta para los datos de demo de `res.partner`

#### Ejemplo de función en Python

```python
import os

def write_text(text, file):
  with open(file, "a") as f:
    f.write(text)

def write_dev(line, file):
  xml = f'''
  <record id="dev_{line[0]}" model="res.partner">
    <field name="name">{line[1]}</field>
    <field name="access_code">{line[2]}</field>
    <field name="is_dev">True</field>
  </record>
  '''
  write_text(xml, file)

def devs(source, demo_file):
  if os.path.exists(demo_file):
    os.remove(demo_file)
  write_text('<odoo><data>\n', demo_file)
  with open(source) as file:
    for line in file:
      fields = line.strip().split(',')
      write_dev(fields, demo_file)
  write_text('</data></odoo>\n', demo_file)

# Uso:
# devs('devels.csv', 'demo/devs.xml')
```

Este script:

- Borra el fichero destino si ya existe
- Añade las etiquetas `<odoo><data>` al principio y `</data></odoo>` al final
- Genera un registro por cada línea del CSV

---

## Prueba y añadir al manifest

Probamos el script y comprobamos que genera el fichero `devs.xml` con los registros correctamente estructurados.

Para que Odoo cargue estos datos de demo, debemos añadir el nuevo fichero generado en la sección `demo` del `__manifest__.py`:

```python
'demo': [
  'demo/demo.xml',
  'demo/devs.xml',
],
```

**Nota:**  
Para que Odoo cargue los datos de demo correctamente, hay que reiniciar Docker (o el servidor Odoo) y actualizar el módulo.

