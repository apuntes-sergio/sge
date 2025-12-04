---
title: Introducción a Odoo
description: Arquitectura, ORM y estructura de módulos en Odoo
---

# Introducción a Odoo

Odoo es un framework de desarrollo de aplicaciones empresariales de código abierto que utiliza Python y PostgreSQL. Su diseño modular permite crear aplicaciones de negocio completas mediante la extensión y personalización de módulos existentes, evitando tener que desarrollar todo desde cero.

En este capítulo conocerás la arquitectura de Odoo, entenderás cómo funciona el ORM (Object-Relational Mapping) y aprenderás a crear y estructurar módulos correctamente. Estos conceptos son fundamentales para el desarrollo en Odoo, ya que todo lo que construyas se basará en esta base.

## Arquitectura de Odoo

Odoo sigue el patrón de diseño **Modelo-Vista-Controlador (MVC)**, separando la lógica de negocio, la presentación y el control de flujo en componentes independientes.

### Componentes Principales

**Modelo (Model)**

Representa la estructura de datos y la lógica de negocio. Los modelos se definen como clases Python que el ORM mapea automáticamente a tablas PostgreSQL. Cada modelo define:

- Campos de datos (nombre, tipo, restricciones)
- Relaciones con otros modelos
- Métodos de negocio
- Validaciones y restricciones

**Vista (View)**

Define cómo se presenta la información al usuario. Las vistas se declaran en archivos XML y pueden ser de varios tipos: formularios, listas, kanban, calendarios, gráficos, etc. Las vistas son independientes de los modelos, permitiendo múltiples formas de visualizar los mismos datos.

**Controlador (Controller)**

Gestiona la interacción entre el modelo y la vista. En Odoo, el controlador incluye:

- Métodos del ORM (create, read, update, delete)
- Decoradores que modifican el comportamiento (@api.depends, @api.onchange, etc.)
- Controladores web para rutas HTTP personalizadas
- Lógica de negocio en métodos Python

<figure markdown="span" align="center">
  ![Arquitectura de Odoo](./imgs/desarrollo/Odooarch.png){ width="75%" }
  <figcaption>Arquitectura MVC y ORM en Odoo</figcaption>
</figure>

### Flujo de Trabajo

Cuando un usuario interactúa con Odoo:

1. El **cliente web** (JavaScript) envía una petición al servidor
2. El **servidor Odoo** procesa la petición a través del controlador
3. El **ORM** traduce las operaciones Python a consultas SQL
4. **PostgreSQL** ejecuta las consultas y devuelve resultados
5. El **ORM** convierte los resultados en objetos Python
6. El **servidor** procesa los datos y renderiza la vista correspondiente
7. El **cliente** recibe y muestra la información al usuario

Este flujo permite desarrollar aplicaciones sin escribir SQL directamente, simplificando enormemente el desarrollo y mantenimiento.

## ORM: Object-Relational Mapping

El ORM es una de las características más potentes de Odoo. Actúa como una capa de abstracción entre Python y PostgreSQL, permitiendo trabajar con datos como si fueran objetos Python en lugar de tablas y filas.

### ¿Por Qué es Importante el ORM?

**Sin ORM** (SQL directo):

```sql
SELECT p.name, p.email, c.name as country 
FROM res_partner p 
LEFT JOIN res_country c ON p.country_id = c.id 
WHERE p.is_company = true AND p.customer = true;
```

**Con ORM** (Python):

```python
partners = self.env['res.partner'].search([
    ('is_company', '=', True),
    ('customer', '=', True)
])
for partner in partners:
    print(partner.name, partner.email, partner.country_id.name)
```

El código con ORM es:

- Más legible y mantenible
- Menos propenso a errores
- Independiente del motor de base de datos
- Automáticamente seguro contra SQL injection
- Gestiona automáticamente las relaciones entre tablas

### Funcionalidades del ORM

El ORM de Odoo proporciona:

**Mapeo automático de tablas**

Cada modelo Python se convierte en una tabla PostgreSQL sin necesidad de crear las tablas manualmente. Los cambios en el modelo se reflejan automáticamente en la base de datos al actualizar el módulo.

**Operaciones CRUD simplificadas**

- `create()`: Crear registros
- `search()`: Buscar y leer registros
- `write()`: Actualizar registros
- `unlink()`: Eliminar registros

**Gestión automática de relaciones**

Los campos Many2one, One2many y Many2many se traducen automáticamente en claves foráneas y tablas intermedias, sin necesidad de gestionarlas manualmente.

**Validaciones y restricciones**

Define validaciones en Python que el ORM ejecuta automáticamente antes de guardar datos, asegurando la integridad sin triggers SQL.

!!! example "Ejemplo básico de un modelo:"

    ```python
    from odoo import models, fields, api
    from odoo.exceptions import ValidationError

    class Producto(models.Model):
        _name = 'mi.producto'
        _description = 'Modelo de ejemplo de producto'

        nombre = fields.Char(string='Nombre', required=True)
        precio = fields.Float(string='Precio', required=True)
        descripcion = fields.Text(string='Descripción')
        fecha_alta = fields.Date(string='Fecha de Alta', default=fields.Date.today)
        activo = fields.Boolean(string='Activo', default=True)

        @api.constrains('precio')
        def _check_precio(self):
            for producto in self:
                if producto.precio < 0:
                    raise ValidationError('El precio no puede ser negativo.')
    ```

En este ejemplo:

- El modelo `mi.producto` se convierte en la tabla `mi_producto`
- Los campos se crean automáticamente con los tipos correctos
- La validación se ejecuta automáticamente antes de guardar
- El ORM gestiona todo el ciclo de vida del dato

## Estructura de un Módulo

Todo el desarrollo en Odoo se realiza mediante módulos. Un módulo es un paquete Python con una estructura específica que Odoo puede cargar e integrar en el sistema.

### Componentes de un Módulo

Un módulo típico tiene esta estructura:

```
mi_modulo/
├── __init__.py              # Importa los paquetes del módulo
├── __manifest__.py          # Define metadatos y dependencias
├── models/                  # Modelos (lógica de negocio)
│   ├── __init__.py
│   └── mi_modelo.py
├── views/                   # Vistas XML
│   ├── mi_modelo_views.xml
│   └── menu.xml
├── security/                # Permisos y grupos
│   └── ir.model.access.csv
├── data/                    # Datos iniciales
│   └── datos_iniciales.xml
├── demo/                    # Datos de demostración
│   └── demo_data.xml
├── static/                  # Archivos estáticos
│   └── description/
│       └── icon.png
└── controllers/             # Controladores web (opcional)
    ├── __init__.py
    └── main.py
```

### Archivos Fundamentales

#### `__init__.py`

Archivo Python que inicializa el módulo importando los paquetes necesarios:

!!!example "__init_.py"
    ```python
    # -*- coding: utf-8 -*-

    from . import models
    from . import controllers
    ```

#### `__manifest__.py`

Archivo Python que define los metadatos del módulo:

!!!example "__manifest__.py"
    ```python
    # -*- coding: utf-8 -*-
    {
        'name': 'Mi Módulo',
        'version': '18.0.1.0.0',
        'category': 'Custom',
        'summary': 'Breve descripción del módulo',
        'description': """
            Descripción detallada del módulo.
            Puede ocupar varias líneas.
        """,
        'author': 'Tu Nombre',
        'website': 'https://www.ejemplo.com',
        'license': 'LGPL-3',
        'depends': [
            'base',
            'sale',
        ],
        'data': [
            'security/ir.model.access.csv',
            'views/mi_modelo_views.xml',
            'views/menu.xml',
            'data/datos_iniciales.xml',
        ],
        'demo': [
            'demo/demo_data.xml',
        ],
        'installable': True,
        'application': True,
        'auto_install': False,
    }
    ```

**Campos importantes del manifest**:

| Campo | Descripción |
|-------|-------------|
| `name` | Nombre del módulo (visible en interfaz) |
| `version` | Versión en formato `{odoo}.{mayor}.{menor}.{patch}` |
| `depends` | Lista de módulos requeridos |
| `data` | Archivos XML/CSV a cargar al instalar |
| `demo` | Datos de demostración (opcionales) |
| `installable` | Si está disponible para instalación |
| `application` | Si aparece como aplicación principal |
| `auto_install` | Si se instala automáticamente |

### Ubicación de Módulos

Los módulos deben estar en directorios especificados en la configuración de Odoo mediante el parámetro `--addons-path` o en el archivo de configuración:

```ini
[options]
addons_path = /ruta/addons,/ruta/mi_modulo,/ruta/extra-addons
```

Odoo busca módulos en todos los directorios especificados. Puedes tener múltiples directorios para organizar módulos estándar, personalizados y de terceros.

## Crear Módulos con Scaffold

Odoo proporciona el comando `scaffold` para generar la estructura básica de un módulo automáticamente, evitando tener que crear todos los archivos manualmente.

### Sintaxis Básica

```bash
odoo scaffold nombre_modulo /ruta/donde/crear
```

Por ejemplo, para crear un módulo en el directorio de addons:

```bash
odoo scaffold mi_gestion /mnt/extra-addons
```

Esto genera una estructura completa con:

- Archivos `__init__.py` y `__manifest__.py` configurados
- Carpetas `models`, `views`, `security`, `controllers`, `demo`
- Ejemplos de modelo, vista y controlador
- Archivo de permisos básico

### Plantillas de Scaffold

Odoo incluye plantillas predefinidas en el directorio de instalación, normalmente en `cli/templates/`. Las plantillas usan el motor **Jinja2** para generar código dinámico.

<figure markdown="span" align="center">
  ![Plantillas de scaffold](./imgs/desarrollo/Odoo_Templates.png){ width="75%" }
  <figcaption>Plantillas incluidas en Odoo para scaffold</figcaption>
</figure>

**Plantillas disponibles**:

- `default`: Módulo estándar con modelo, vista y controlador de ejemplo
- `theme`: Para crear temas de sitio web

Puedes especificar la plantilla con el parámetro `-t`:

```bash
odoo scaffold mi_modulo /ruta/addons -t default
```

### Personalizar Plantillas

Si siempre creas módulos con la misma estructura (tu logo, copyright, estructura específica), puedes crear tu propia plantilla:

1. Copia el directorio `default` a uno nuevo (ej: `mi_plantilla`)
2. Modifica los archivos según tus necesidades
3. Úsala al crear módulos:

```bash
odoo scaffold mi_modulo /ruta/addons -t mi_plantilla
```

### Después del Scaffold

Una vez generado el módulo:

1. **Ajusta permisos** si trabajas con Docker o entornos multi-usuario:
   ```bash
   chmod -R 777 /ruta/mi_modulo
   ```

2. **Personaliza el manifest** con información real del módulo

3. **Reinicia Odoo** para que detecte el nuevo módulo:
   ```bash
   docker compose restart odoo
   ```

4. **Actualiza la lista de aplicaciones** en Odoo (Modo desarrollador → Apps → Update Apps List)

5. **Instala el módulo** desde la interfaz

## Tipos de Modelos en Odoo

Odoo proporciona tres tipos base de modelos según el propósito y persistencia de los datos:

### models.Model

**Modelos persistentes** que representan tablas reales en PostgreSQL. Los datos se guardan permanentemente en la base de datos.

!!! example "ejemplo"
    ```python
    class Cliente(models.Model):
        _name = 'mi.cliente'
        _description = 'Clientes de la empresa'
        
        nombre = fields.Char(required=True)
        email = fields.Char()
    ```

**Uso típico**: Entidades de negocio principales (clientes, productos, pedidos, facturas, etc.)

### models.TransientModel

**Modelos temporales** para datos que solo necesitan existir durante una sesión. Los registros se eliminan automáticamente después de cierto tiempo (por defecto 7 días).

!!! example "ejemplo"
    ```python
    class AsistenteExportacion(models.TransientModel):
        _name = 'mi.asistente.exportacion'
        _description = 'Asistente para exportar datos'
        
        formato = fields.Selection([('csv', 'CSV'), ('xlsx', 'Excel')])
        fecha_desde = fields.Date()
    ```

**Uso típico**: Wizards (asistentes), formularios temporales, diálogos de configuración

### models.AbstractModel

**Modelos abstractos** que no crean tabla en la base de datos. Se usan como plantillas para que otros modelos hereden funcionalidad común.

!!! example "ejemplo"
    ```python
    class ModeloConTimestamp(models.AbstractModel):
        _name = 'mi.modelo.timestamp'
        _description = 'Modelo base con campos de auditoría'
        
        fecha_creacion = fields.Datetime(default=fields.Datetime.now)
        usuario_creacion = fields.Many2one('res.users', default=lambda self: self.env.user)
    ```

**Uso típico**: Definir funcionalidades reutilizables sin duplicar código

### Comparativa de Tipos de Modelos

| Característica | Model | TransientModel | AbstractModel |
|----------------|-------|----------------|---------------|
| **Crea tabla en BD** | Sí | Sí | No |
| **Persistencia** | Permanente | Temporal | No aplica |
| **Eliminación automática** | No | Sí (configurable) | No aplica |
| **Uso principal** | Datos reales | Wizards/Diálogos | Funcionalidad compartida |
| **Heredable** | Sí | Sí | Sí |
| **Ejemplo** | Productos, Clientes | Asistente de importación | Modelo base con auditoría |

## Campos Reservados del ORM

Odoo crea automáticamente ciertos campos en todos los modelos que heredan de `models.Model`. Estos campos proporcionan funcionalidades estándar y trazabilidad.

### Campos Automáticos

**id**

Clave primaria autoincremental generada por PostgreSQL. Único para cada registro en su tabla.

```python
# Acceder al ID de un registro
print(registro.id)  # 42
```

**create_date**

Fecha y hora de creación del registro (tipo Datetime). Se establece automáticamente al crear.

**create_uid**

Referencia (Many2one) al usuario que creó el registro. Apunta al modelo `res.users`.

```python
print(registro.create_uid.name)  # "Administrator"
```

**write_date**

Fecha y hora de la última modificación del registro. Se actualiza automáticamente en cada `write()`.

**write_uid**

Referencia al usuario que realizó la última modificación.

### Campos Especiales Opcionales

Algunos campos tienen comportamiento especial si los defines en tu modelo:

**name**

Campo usado como identificador visible del registro. Si existe, Odoo lo usa:

- En referencias Many2one (muestra el name en lugar del ID)
- Como título en formularios
- En búsquedas por defecto

```python
nombre = fields.Char(string='Nombre', required=True)
```

Si no defines un campo `name`, puedes especificar cuál usar con `_rec_name`:

```python
_rec_name = 'codigo'
codigo = fields.Char(string='Código')
```

**active**

Campo booleano que permite archivar registros sin eliminarlos. Si existe:

- Los registros con `active=False` no aparecen en búsquedas por defecto
- Aparece un botón "Archivar/Desarchivar" en la interfaz

```python
active = fields.Boolean(string='Activo', default=True)
```

**sequence**

Campo entero para ordenar manualmente los registros. Útil en listas donde el usuario quiere controlar el orden.

```python
sequence = fields.Integer(string='Secuencia', default=10)

# Especificar orden por defecto en el modelo
_order = 'sequence, name'
```

## Inspeccionar Modelos en Odoo

Cuando desarrollas en Odoo, es útil poder inspeccionar los modelos existentes, sus campos y relaciones. Esto te ayuda a entender la estructura del sistema y aprovechar modelos estándar.

### Desde la Interfaz (Modo Desarrollador)

Activa el **modo desarrollador** desde Ajustes → Activar modo desarrollador. Luego:

**Ver todos los modelos**:

1. Ve a **Ajustes → Técnico → Estructura de la base de datos → Modelos**
2. Busca el modelo por nombre (ej: `res.partner`)
3. Haz clic para ver sus detalles

**Información disponible**:

- Lista completa de campos y sus tipos
- Relaciones con otros modelos
- Módulos que extienden el modelo
- Vistas asociadas
- Permisos configurados

**Ver External IDs**:

Los identificadores externos son cruciales para referenciar registros en XML:

1. Ve a **Ajustes → Técnico → Secuencias e identificadores → Identificadores externos**
2. Filtra por modelo o busca por Complete ID

**Ver metadatos de un registro**:

En cualquier formulario (modo desarrollador):

1. Haz clic en el icono de "bug" (depuración)
2. Selecciona "Ver metadatos"
3. Verás: ID, External ID, usuario de creación, fechas, etc.

### Desde PostgreSQL

Puedes conectarte directamente a la base de datos para inspeccionar tablas:

!!! example "ejemplo"
    ```bash
    # Entrar en el contenedor de PostgreSQL
    docker exec -it postgres_container bash

    # Conectar a la base de datos
    psql -U odoo -d nombre_base_datos

    # Ver todas las tablas
    \dt

    # Ver estructura de una tabla
    \d nombre_tabla

    # Buscar tablas por patrón
    \dt *partner*

    # Ver los campos de un modelo
    SELECT column_name, data_type 
    FROM information_schema.columns 
    WHERE table_name = 'res_partner';
    ```

### Desde Python Shell

Odoo incluye un shell interactivo para ejecutar código Python directamente:

```bash
# Iniciar shell de Odoo
odoo shell -d nombre_base_datos

# O si usas Docker
docker exec -it odoo_container odoo shell -d nombre_base_datos
```


!!! example "Ejemplos de uso en el shell"

    ```python
    # Acceder a un modelo
    Partner = env['res.partner']

    # Buscar registros
    partners = Partner.search([('is_company', '=', True)])

    # Ver campos de un modelo
    Partner._fields.keys()

    # Ver información de un campo específico
    Partner._fields['name']

    # Buscar un registro por External ID
    env.ref('base.main_partner')

    # Hacer cambios persistentes
    env.cr.commit()
    ```

!!! warning "Cambios en el Shell"
    Los cambios realizados en el shell **no son persistentes** hasta que ejecutes `env.cr.commit()`. Esto previene cambios accidentales en producción.

!!!tipo "Buenas Prácticas"

    Al desarrollar módulos en Odoo, sigue estas recomendaciones:

    **Nomenclatura de módulos**

    - Usa minúsculas y guiones bajos: `mi_modulo_gestion`
    - Nombre descriptivo del propósito
    - Evita nombres genéricos que puedan colisionar

    **Nomenclatura de modelos**

    - Usa el prefijo del módulo: `mi_modulo.cliente`
    - Nombre en singular: `product.product` no `product.products`
    - Usa puntos para separar: `mi_modulo.sub_categoria`

    **Estructura de archivos**

    - Organiza por tipo: models/, views/, controllers/
    - Un archivo por modelo en modelos complejos
    - Nombra archivos descriptivamente: `cliente.py`, `cliente_views.xml`

    **Dependencias**

    - Declara todas las dependencias en el manifest
    - No dependas de módulos opcionales sin validar
    - Mantén las dependencias al mínimo necesario

    **Documentación**

    - Añade docstrings a modelos y métodos complejos
    - Documenta campos no obvios con el atributo `help`
    - Mantén actualizado el manifest con descripción clara

    **Versionado**

    - Sigue el formato `{odoo_version}.{major}.{minor}.{patch}`
    - Incrementa la versión en cada actualización
    - Documenta cambios importantes en el README

    **Testing**

    - Usa datos de demo para pruebas
    - No mezcles datos de demo con datos necesarios
    - Valida que el módulo se desinstala limpiamente