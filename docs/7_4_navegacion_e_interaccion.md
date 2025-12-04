---
title: Navegación e Interacción
description: Acciones, menús, contexto y archivos de datos en Odoo
---

# Navegación e Interacción

La navegación en Odoo se construye mediante la interacción de varios elementos: archivos de datos XML que cargan información inicial, acciones que definen qué mostrar y cómo, menús que organizan la aplicación, y el contexto que transporta información entre vistas. Dominar estos conceptos es esencial para crear aplicaciones intuitivas y bien estructuradas.

En este capítulo aprenderás a cargar datos iniciales, crear acciones para abrir vistas, estructurar menús de navegación y usar el contexto y los dominios para filtrar y personalizar el comportamiento de tu aplicación.

## Archivos de Datos XML

Los archivos de datos XML permiten cargar información en la base de datos al instalar o actualizar un módulo. Pueden contener datos de configuración, registros de demostración, o incluso formar parte de la estructura de vistas y menús.

### Estructura Básica
```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="nombre.modelo" id="identificador_registro">
        <field name="nombre_campo">valor</field>
        <field name="otro_campo">otro valor</field>
    </record>
</odoo>
```

**Elementos clave**:

- `<odoo>`: Contenedor principal de todos los registros
- `<record>`: Define un registro a insertar/actualizar
- `model`: Modelo donde se creará el registro
- `id`: Identificador externo único (External ID)
- `<field>`: Define el valor de un campo

!!!example "Ejemplo Práctico"
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <record model="producto.categoria" id="categoria_electronica">
            <field name="name">Electrónica</field>
            <field name="descripcion">Productos electrónicos y tecnología</field>
            <field name="activo" eval="True"/>
        </record>
        
        <record model="mi.producto" id="producto_laptop">
            <field name="nombre">Laptop Dell XPS</field>
            <field name="codigo">LAPTOP-001</field>
            <field name="precio" eval="1299.99"/>
            <field name="categoria_id" ref="categoria_electronica"/>
            <field name="activo" eval="True"/>
        </record>
    </odoo>
    ```

### External IDs (Identificadores Externos)

Los External IDs permiten referenciar registros de forma estable entre diferentes módulos e instalaciones. Odoo los almacena en el modelo `ir.model.data`.

**Formato completo**: `modulo.identificador`

!!!example "Ejemplo"
    ```xml
    <!-- Definir un External ID -->
    <record model="res.partner" id="partner_ejemplo">
        <field name="name">Mi Empresa</field>
    </record>

    <!-- Referenciar desde otro módulo -->
    <record model="otra.tabla" id="otro_registro">
        <field name="partner_id" ref="mi_modulo.partner_ejemplo"/>
    </record>
    ```

!!! tip "Convención de nomenclatura"
    Usa prefijos descriptivos para los External IDs:

    - `view_`: para vistas
    - `action_`: para acciones
    - `menu_`: para menús
    - `data_`: para datos
    - `demo_`: para datos de demostración

### Referenciar Registros con ref

Para referenciar un registro existente por su External ID:
```xml
<field name="usuario_id" ref="base.user_admin"/>
<field name="pais_id" ref="base.es"/>
<field name="empresa_id" ref="base.main_company"/>
```

Si no especificas el módulo, Odoo busca en el módulo actual:
```xml
<!-- Busca en mi_modulo.categoria_electronica -->
<field name="categoria_id" ref="categoria_electronica"/>
```

### Usar `eval` para Expresiones

El atributo `eval` permite ejecutar expresiones Python:

!!!example "Ejemplo"

    ```xml
    <!-- Valores booleanos -->
    <field name="activo" eval="True"/>
    <field name="archivado" eval="False"/>

    <!-- Números -->
    <field name="cantidad" eval="10"/>
    <field name="precio" eval="99.99"/>

    <!-- Fechas -->
    <field name="fecha_actual" eval="datetime.now()"/>
    <field name="ayer" eval="(datetime.now() - timedelta(days=1))"/>
    <field name="fecha_fija" eval="datetime(2024, 1, 15)"/>

    <!-- Referenciar con eval (equivalente a ref) -->
    <field name="categoria_id" eval="ref('categoria_electronica')"/>

    <!-- Acceder a propiedades del registro referenciado -->
    <field name="precio_categoria" eval="ref('categoria_electronica').precio_base"/>
    ```

!!! warning "Importaciones en eval"
    Las expresiones `eval` tienen acceso a:

    - `datetime`, `timedelta` de Python
    - `ref()` para buscar External IDs
    - `obj()` para acceder al entorno actual
    
    Pero no a otros módulos o funciones personalizadas.

### Campos Relacionales en XML

!!!example "Many2one"
    ```xml
    <field name="categoria_id" ref="categoria_electronica"/>
    <!-- o con eval -->
    <field name="categoria_id" eval="ref('categoria_electronica')"/>
    ```

!!!example "Many2many - Usando tuplas de comandos:"
    ```xml
    <!-- Vincular registros existentes -->
    <field name="etiqueta_ids" eval="[(6, 0, [ref('etiqueta_nuevo'), ref('etiqueta_oferta')])]"/>

    <!-- Crear y vincular nuevos registros -->
    <field name="etiqueta_ids" eval="[(0, 0, {'name': 'Destacado', 'color': 5})]"/>
    ```

!!!example "One2many"
    ```xml
    <field name="linea_ids" eval="[
        (0, 0, {'producto_id': ref('producto_laptop'), 'cantidad': 2, 'precio': 1299.99}),
        (0, 0, {'producto_id': ref('producto_mouse'), 'cantidad': 1, 'precio': 29.99})
    ]"/>
    ```

### Comandos para Campos X2many

| Comando | Sintaxis | Descripción |
|---------|----------|-------------|
| Crear y vincular | `(0, 0, {valores})` | Crea un nuevo registro y lo vincula |
| Actualizar | `(1, id, {valores})` | Actualiza un registro vinculado |
| Desvincular y eliminar | `(2, id, 0)` | Desvincula y elimina |
| Desvincular | `(3, id, 0)` | Desvincula sin eliminar |
| Vincular | `(4, id, 0)` | Vincula un registro existente |
| Desvincular todos | `(5, 0, 0)` | Desvincula todos |
| Reemplazar | `(6, 0, [ids])` | Reemplaza con lista de IDs |

### Campos Binary e Image

!!!example "Con archivo local"
    ```xml
    <field name="imagen" type="base64" file="mi_modulo/static/img/producto.jpg"/>
    <field name="icono" type="base64" file="mi_modulo/static/description/icon.png"/>
    ```

La ruta es relativa al directorio raíz del módulo.

!!!example "Con base64 directo"
    ```xml
    <field name="imagen">iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk...</field>
    ```

### Eliminar Registros

Para eliminar registros al actualizar el módulo:
!!!example "ejemplo"

    ```xml
    <!-- Por External ID -->
    <delete model="mi.modelo" id="registro_a_eliminar"/>

    <!-- Por búsqueda -->
    <delete model="mi.modelo" search="[('nombre', '=', 'Obsoleto')]"/>
    ```

!!! danger "Cuidado con delete"
    Eliminar registros en datos puede romper dependencias. Úsalo principalmente en datos de demostración, no en datos de producción.

### Organizar Archivos de Datos

**Estructura recomendada**:
```
mi_modulo/
├── data/
│   ├── categorias_data.xml      # Datos base necesarios
│   ├── configuracion_data.xml   # Configuración por defecto
│   └── secuencias_data.xml      # Secuencias
├── demo/
│   ├── productos_demo.xml       # Datos de demostración
│   └── clientes_demo.xml
└── security/
    └── ir.model.access.csv      # Permisos
```

**En el manifest**:
```python
'data': [
    'security/ir.model.access.csv',
    'data/categorias_data.xml',
    'data/configuracion_data.xml',
    'views/menu.xml',
    'views/producto_views.xml',
],
'demo': [
    'demo/productos_demo.xml',
    'demo/clientes_demo.xml',
],
```

!!! tip "Orden de carga"
    Los archivos se cargan en el orden declarado en el manifest. Asegúrate de que:

    1. Primero se cargan permisos (security)
    2. Luego datos base (data)
    3. Después vistas y menús (views)
    4. Finalmente demos (demo)

## Acciones

Las acciones definen qué debe hacer Odoo cuando el usuario interactúa con menús, botones o enlaces. Son el puente entre la interfaz y la funcionalidad.

### Tipos de Acciones

Odoo proporciona varios tipos de acciones según el propósito:

| Tipo | Modelo | Uso Principal |
|------|--------|---------------|
| **Window** | `ir.actions.act_window` | Abrir vistas de modelos |
| **Server** | `ir.actions.server` | Ejecutar código Python |
| **Report** | `ir.actions.report` | Generar reportes PDF |
| **URL** | `ir.actions.act_url` | Abrir URLs |
| **Client** | `ir.actions.client` | Acciones del cliente web |

### Acciones Window (Ventana)

Son las más comunes. Abren una o varias vistas de un modelo.

!!!example "Ejemplo básico"
    ```xml
    <record model="ir.actions.act_window" id="action_producto_list">
        <field name="name">Productos</field>
        <field name="res_model">mi.producto</field>
        <field name="view_mode">list,form</field>
    </record>
    ```

**Atributos principales**:

| Atributo | Descripción |
|----------|-------------|
| `name` | Nombre visible de la acción |
| `res_model` | Modelo a mostrar |
| `view_mode` | Tipos de vista disponibles (separados por coma) |
| `view_id` | Vista específica a usar (opcional) |
| `domain` | Filtro de registros |
| `context` | Contexto a pasar a las vistas |
| `target` | Dónde abrir (current, new, fullscreen) |
| `limit` | Registros por página |

!!!example "Ejemplo completo"
    ```xml
    <record model="ir.actions.act_window" id="action_producto_activos">
        <field name="name">Productos Activos</field>
        <field name="res_model">mi.producto</field>
        <field name="view_mode">list,form,kanban</field>
        <field name="domain">[('activo', '=', True)]</field>
        <field name="context">{
            'default_activo': True,
            'search_default_categoria': 1
        }</field>
        <field name="limit">80</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                ¡Crea tu primer producto!
            </p>
            <p>
                Haz clic en "Nuevo" para añadir un producto al inventario.
            </p>
        </field>
    </record>
    ```

### Especificar Vista Concreta

!!!example "Por External ID"
    ```xml
    <record model="ir.actions.act_window" id="action_producto_form">
        <field name="name">Nuevo Producto</field>
        <field name="res_model">mi.producto</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="producto_form_simplificado"/>
        <field name="target">new</field>
    </record>
    ```

!!!example "Con view_ids (múltiples vistas específicas)""
    ```xml
    <record model="ir.actions.act_window" id="action_producto_custom">
        <field name="name">Productos</field>
        <field name="res_model">mi.producto</field>
        <field name="view_mode">list,form</field>
        <field name="view_ids" eval="[
            (5, 0, 0),
            (0, 0, {'view_mode': 'list', 'view_id': ref('producto_list_custom')}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('producto_form_custom')})
        ]"/>
    </record>
    ```

!!! tip "Atributo target"
    - `current`: Abre en la ventana actual (por defecto)
    - `new`: Abre en un diálogo modal
    - `fullscreen`: Abre en pantalla completa
    - `inline`: Incrusta la vista en el contexto actual

### Acciones desde Código Python

Las acciones también pueden devolverse desde métodos Python:

!!!example "ejemplo"

    ```python
    def action_abrir_tareas(self):
        return {
            'type': 'ir.actions.act_window',
            'name': 'Tareas del Proyecto',
            'res_model': 'proyecto.tarea',
            'view_mode': 'list,form',
            'domain': [('proyecto_id', '=', self.id)],
            'context': {
                'default_proyecto_id': self.id,
                'default_usuario_id': self.env.user.id,
            },
            'target': 'current',
        }
    ```

**Abrir un registro específico**:

!!!example "ejemplo"
    ```python
    def action_ver_factura(self):
        return {
            'type': 'ir.actions.act_window',
            'name': 'Factura',
            'res_model': 'account.move',
            'res_id': self.factura_id.id,  # ID del registro a abrir
            'view_mode': 'form',
            'target': 'current',
        }
    ```

### Acciones Server

Ejecutan código Python del lado del servidor. Útiles para automatizaciones y botones que ejecutan lógica compleja.

!!!example "ejemplo"

    ```xml
    <record model="ir.actions.server" id="action_recalcular_precios">
        <field name="name">Recalcular Precios</field>
        <field name="model_id" ref="model_mi_producto"/>
        <field name="state">code</field>
        <field name="code">
            for record in records:
                record.precio = record.coste * 1.30
        </field>
    </record>
    ```

!!!example "ejemplo·**Tipos de acciones server** (`state`):

- `code`: Ejecuta código Python
- `object_create`: Crea un registro
- `object_write`: Modifica registros
- `multi`: Ejecuta múltiples acciones

**Variables disponibles en el código**:

- `records`: Recordset con los registros seleccionados
- `env`: Entorno de Odoo
- `model`: Modelo actual
- `datetime`, `time`, `date`: Módulos de Python

### Acciones URL

Abren una URL externa o interna:

!!!example "ejemplo"
    ```xml
    <record model="ir.actions.act_url" id="action_documentacion">
        <field name="name">Ver Documentación</field>
        <field name="url">https://www.odoo.com/documentation</field>
        <field name="target">new</field>
    </record>
    ```

## Menús

Los menús estructuran la navegación de tu aplicación. Odoo organiza los menús en una jerarquía de tres niveles.

### Estructura de Menús

**Niveles de menú**:

1. **Nivel 1**: Aplicación principal (barra superior)
2. **Nivel 2**: Secciones dentro de la aplicación (lateral izquierdo)
3. **Nivel 3**: Opciones específicas que abren acciones

!!!example "ejemplo niveles menu"
    ```xml
    <!-- Nivel 1: Aplicación principal -->
    <menuitem id="menu_inventario_root" 
              name="Inventario"
              sequence="10"/>

    <!-- Nivel 2: Sección -->
    <menuitem id="menu_inventario_productos" 
              name="Productos"
              parent="menu_inventario_root"
              sequence="10"/>

    <!-- Nivel 3: Acción -->
    <menuitem id="menu_producto_list" 
              name="Todos los Productos"
              parent="menu_inventario_productos"
              action="action_producto_list"
              sequence="10"/>
    ```

!!!note "Sintaxis de menuitem"
    ```xml
    <menuitem id="identificador_unico"
              name="Nombre Visible"
              parent="id_menu_padre"
              action="id_accion"
              sequence="10"
              groups="grupo1,grupo2"
              web_icon="mi_modulo,static/description/icon.png"/>
    ```

**Atributos principales**:

| Atributo | Descripción |
|----------|-------------|
| `id` | Identificador único del menú |
| `name` | Texto visible del menú |
| `parent` | ID del menú padre (sin parent = nivel 1) |
| `action` | Acción a ejecutar al hacer clic |
| `sequence` | Orden de aparición (menor = primero) |
| `groups` | Grupos que pueden ver el menú |
| `web_icon` | Icono para aplicaciones de nivel 1 |

!!!example "Crear Menú Completo"
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <!-- Acción -->
        <record model="ir.actions.act_window" id="action_producto_list">
            <field name="name">Productos</field>
            <field name="res_model">mi.producto</field>
            <field name="view_mode">list,form,kanban</field>
        </record>

        <!-- Menú nivel 1 -->
        <menuitem id="menu_inventario_root"
                  name="Inventario"
                  sequence="10"
                  web_icon="mi_modulo,static/description/icon.png"/>

        <!-- Menú nivel 2 -->
        <menuitem id="menu_inventario_productos"
                  name="Productos"
                  parent="menu_inventario_root"
                  sequence="10"/>

        <!-- Menú nivel 3 con acción -->
        <menuitem id="menu_producto_list"
                  name="Todos los Productos"
                  parent="menu_inventario_productos"
                  action="action_producto_list"
                  sequence="10"/>
    </odoo>
    ```

### Organizar Menús

**Por módulo**:
```
mi_modulo/
└── views/
    ├── menu.xml              # Todos los menús
    ├── producto_views.xml    # Vistas y acciones de productos
    └── categoria_views.xml   # Vistas y acciones de categorías
```

**En el manifest**:
```python
'data': [
    'views/producto_views.xml',
    'views/categoria_views.xml',
    'views/menu.xml',  # Cargar menús al final
],
```

### Iconos para Aplicaciones

Para menús de nivel 1, puedes especificar un icono:
```xml
<menuitem id="menu_mi_app"
          name="Mi Aplicación"
          web_icon="mi_modulo,static/description/icon.png"/>
```

El archivo debe estar en `mi_modulo/static/description/icon.png` con tamaño recomendado de 256x256px.

## Contexto

El contexto es un diccionario Python que transporta información entre vistas, métodos y acciones. Es fundamental para personalizar el comportamiento de Odoo.

### Acceder al Contexto

**En Python**:
```python
# Leer todo el contexto
contexto = self.env.context
print(contexto)

# Leer un valor específico
idioma = self.env.context.get('lang')
modo = self.env.context.get('modo_especial', False)
```

**En XML (vistas)**:
```xml
<field name="campo" invisible="context.get('ocultar_campo', False)"/>
```

### Valores Comunes del Contexto

Odoo incluye automáticamente ciertos valores:

| Clave | Descripción |
|-------|-------------|
| `lang` | Idioma del usuario (ej: 'es_ES') |
| `tz` | Zona horaria del usuario |
| `uid` | ID del usuario actual |
| `active_id` | ID del registro activo |
| `active_ids` | IDs de registros seleccionados |
| `active_model` | Modelo del registro activo |

### Pasar Contexto en Acciones

!!!example "ejemplo"
    ```xml
    <record model="ir.actions.act_window" id="action_tareas_usuario">
        <field name="name">Mis Tareas</field>
        <field name="res_model">proyecto.tarea</field>
        <field name="view_mode">list,form</field>
        <field name="context">{
            'default_usuario_id': uid,
            'search_default_abiertas': 1,
            'ocultar_campo_interno': True
        }</field>
    </record>
    ```

### Valores por Defecto con Contexto

Prefijo `default_` establece valores por defecto en formularios:

!!!example "ejemplo"
    ```xml
    <field name="context">{
        'default_activo': True,
        'default_categoria_id': active_id,
        'default_fecha': context_today()
    }</field>
    ```

!!!example "En botones:"
```xml
<button name="%(action_crear_tarea)d" 
        type="action" 
        string="Crear Tarea"
        context="{'default_proyecto_id': active_id, 'default_usuario_id': uid}"/>
```

### Filtros por Defecto con Contexto

Prefijo `search_default_` activa filtros de búsqueda automáticamente:

!!!example "ejemplo"
    ```xml
    <field name="context">{
        'search_default_activos': 1,
        'search_default_categoria': 1
    }</field>
    ```

Esto activa los filtros con `name="activos"` y `name="categoria"` en la vista search.

### Modificar Contexto en Python

!!!example "ejemplo"
    ```python
    # Añadir valores al contexto
    nuevo_contexto = dict(self.env.context, clave_nueva='valor')
    self = self.with_context(nuevo_contexto)

    # O directamente
    self = self.with_context(clave_nueva='valor', otra_clave='otro_valor')

    # Crear contexto limpio
    self = self.with_context({})
    ```

!!!example "Ejemplo práctico:"
    ```python
    def action_duplicar_sin_copiar_ventas(self):
        # Duplicar producto pero sin copiar las ventas asociadas
        return {
            'type': 'ir.actions.act_window',
            'name': 'Duplicar Producto',
            'res_model': 'mi.producto',
            'view_mode': 'form',
            'context': dict(self.env.context, no_copiar_ventas=True),
            'target': 'new',
        }

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        # Si el contexto tiene no_copiar_ventas, no copiar las ventas
        if self.env.context.get('no_copiar_ventas'):
            default = dict(default or {})
            default['venta_ids'] = []
        return super(Producto, self).copy(default)
    ```

## Dominios (Domains)

Los dominios son filtros que limitan los registros visibles o seleccionables. Se escriben como listas de tuplas con operadores de comparación.

### Sintaxis Básica
```python
[('campo', 'operador', 'valor')]
```

!!!example "Ejemplo"
    ```python
    [('activo', '=', True)]  # Registros activos
    [('precio', '>', 100)]    # Precio mayor a 100
    [('nombre', 'ilike', 'laptop')]  # Nombre contiene "laptop"
    ```

### Operadores de Comparación

| Operador | Descripción | Ejemplo |
|----------|-------------|---------|
| `=` | Igual | `('estado', '=', 'abierto')` |
| `!=` | Diferente | `('estado', '!=', 'cancelado')` |
| `>` | Mayor que | `('precio', '>', 100)` |
| `>=` | Mayor o igual | `('stock', '>=', 10)` |
| `<` | Menor que | `('edad', '<', 18)` |
| `<=` | Menor o igual | `('descuento', '<=', 0.5)` |
| `in` | En lista | `('estado', 'in', ['borrador', 'confirmado'])` |
| `not in` | No en lista | `('tipo', 'not in', ['temporal', 'prueba'])` |
| `like` | Coincide (case-sensitive) | `('codigo', 'like', 'PROD%')` |
| `ilike` | Coincide (case-insensitive) | `('nombre', 'ilike', '%laptop%')` |
| `=like` | Patrón SQL | `('email', '=like', '%@gmail.com')` |
| `=ilike` | Patrón SQL (insensitive) | `('email', '=ilike', '%@gmail.com')` |

### Operadores Lógicos

!!!example "AND (por defecto):"
    ```python
    [('activo', '=', True), ('precio', '>', 100)]
    # Registros activos Y con precio > 100
    ```

!!!example "OR:"
    ```python
    ['|', ('estado', '=', 'borrador'), ('estado', '=', 'confirmado')]
    # Registros en estado borrador O confirmado
    ```

!!!example "NOT:"
    ```python
    ['!', ('archivado', '=', True)]
    # Registros NO archivados
    ```

!!!example "Combinar operadores:"
    ```python
    [
        '|',  # OR principal
            ('estado', '=', 'urgente'),
            '&',  # AND anidado
                ('prioridad', '>', 5),
                ('fecha_limite', '<=', '2024-12-31')
    ]
    # (urgente) O (prioridad > 5 Y fecha_limite <= 2024-12-31)
    ```

!!! tip "Prefijo de operadores lógicos"
    Los operadores `|`, `&`, `!` se colocan ANTES de los elementos que afectan:
    - `|` afecta a los 2 siguientes elementos
    - `&` afecta a los 2 siguientes elementos
    - `!` afecta al siguiente elemento

### Dominios en Acciones

!!!example "ejemplo"
    ```xml
    <record model="ir.actions.act_window" id="action_productos_activos">
        <field name="name">Productos Activos</field>
        <field name="res_model">mi.producto</field>
        <field name="view_mode">list,form</field>
        <field name="domain">[('activo', '=', True), ('stock', '>', 0)]</field>
    </record>
    ```

### Dominios en Campos Relacionales

Limitar las opciones disponibles en campos Many2one o Many2many:

!!!example "ejemplo"
    ```xml
    <!-- En la vista -->
    <field name="categoria_id" domain="[('tipo', '=', 'producto')]"/>
    <field name="cliente_id" domain="[('is_company', '=', False), ('customer', '=', True)]"/>
    ```

!!!example "En el modelo:"
    ```python
    categoria_id = fields.Many2one(
        'producto.categoria',
        domain=[('tipo', '=', 'producto')]
    )
    ```

### Dominios Dinámicos

Usar valores de otros campos en el dominio:

!!!example "ejemplo"
    ```xml
    <!-- Categorías del tipo seleccionado -->
    <field name="tipo" invisible="1"/>
    <field name="categoria_id" domain="[('tipo', '=', tipo)]"/>

    <!-- Productos de la categoría seleccionada -->
    <field name="categoria_id" invisible="1"/>
    <field name="producto_id" domain="[('categoria_id', '=', categoria_id)]"/>

!!!example "Con múltiples condiciones:"
    ```xml
    <field name="es_empresa" invisible="1"/>
    <field name="pais_id" invisible="1"/>
    <field name="contacto_id" domain="[
        ('is_company', '=', es_empresa),
        ('country_id', '=', pais_id),
        ('active', '=', True)
    ]"/>
    ```

### Dominios con Contexto

Usar valores del contexto en dominios:
```xml
<field name="producto_id" domain="[('id', 'in', context.get('producto_ids', []))]"/>
```

!!!example "En Python:"
    ```python
    def _compute_productos_disponibles(self):
        productos_ids = self.categoria_id.producto_ids.ids
        return [('id', 'in', productos_ids)]

    producto_id = fields.Many2one(
        'mi.producto',
        domain=_compute_productos_disponibles
    )
    ```

### Dominios en Búsquedas

!!!example "ejemplo de dominios de búsquedas"

    ```python
    # Buscar productos activos con stock
    productos = self.env['mi.producto'].search([
        ('activo', '=', True),
        ('stock', '>', 0)
    ])

    # Con OR
    productos = self.env['mi.producto'].search([
        '|',
        ('categoria_id.name', '=', 'Electrónica'),
        ('categoria_id.name', '=', 'Informática')
    ])

    # Con límite
    ultimos_10 = self.env['mi.producto'].search([], limit=10, order='create_date desc')
    ```

!!!tip "Buenas Prácticas"

    **Archivos de datos**

    - Separa datos de configuración de datos de demo
    - Usa External IDs descriptivos y únicos
    - Carga datos en orden lógico (dependencias primero)
    - Documenta datos complejos con comentarios XML

    **Acciones**

    - Nombra acciones descriptivamente: `action_producto_activos`
    - Establece límites razonables (80-100 registros)
    - Añade mensajes de ayuda cuando la vista esté vacía
    - Usa target="new" para formularios temporales/wizards

    **Menús**

    - Estructura jerárquica lógica (no más de 3 niveles)
    - Secuencias coherentes (10, 20, 30... para facilitar reordenar)
    - Nombres de menú claros y concisos
    - Agrupa funcionalidades relacionadas

    **Contexto**

    - No abuses del contexto (solo información necesaria)
    - Usa prefijos estándar: `default_`, `search_default_`
    - Documenta claves personalizadas del contexto
    - Limpia el contexto cuando sea necesario

    **Dominios**

    - Mantén dominios simples y legibles
    - Documenta lógica compleja con comentarios
    - Usa dominios dinámicos en lugar de código Python cuando sea posible
    - Valida que los campos usados existan en el modelo
