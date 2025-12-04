---
title: Desarrollo de Modelos
description: Campos, relaciones, validaciones y restricciones en Odoo
---

# Desarrollo de Modelos

Los modelos son el corazón de cualquier aplicación Odoo. Definen la estructura de datos, las relaciones entre entidades y las reglas de negocio que garantizan la integridad de la información. En este capítulo aprenderás a crear modelos robustos, desde los campos más simples hasta las validaciones más complejas.

Un modelo bien diseñado facilita el desarrollo de vistas, simplifica el mantenimiento y asegura que los datos se mantengan consistentes a lo largo del tiempo. Dominar los modelos es esencial para desarrollar aplicaciones empresariales de calidad en Odoo.

## Definición Básica de un Modelo

Todo modelo en Odoo es una clase Python que hereda de `models.Model` y se declara con ciertos atributos obligatorios.

!!!example "Estructura Mínima"

    ```python
    from odoo import models, fields, api

    class MiModelo(models.Model):
        _name = 'mi.modulo.mimodelo'
        _description = 'Descripción del modelo'
        
        nombre = fields.Char(string='Nombre', required=True)
    ```

**Atributos fundamentales**:

- `_name`: Identificador único del modelo (obligatorio). Se convierte en el nombre de la tabla
- `_description`: Descripción legible del modelo (recomendado para accesibilidad)

### Atributos Adicionales del Modelo

Odoo proporciona varios atributos para configurar el comportamiento del modelo:

**_order**

Define el orden por defecto de los registros en búsquedas y listas:

!!!example "ejemplo"

    ```python
    _order = 'nombre, fecha_creacion desc'
    ```

Puedes especificar varios campos separados por comas. Usa `desc` para orden descendente.

**_rec_name**

Especifica qué campo usar como identificador visible si no tienes un campo `name`:

!!!example "ejemplo"

    ```python
    _rec_name = 'codigo'
    codigo = fields.Char(string='Código')
    ```

**_sql_constraints**

Define restricciones a nivel de base de datos:

!!!example "ejemplo"

    ```python
    _sql_constraints = [
        ('nombre_unico', 'unique(nombre)', 'El nombre debe ser único'),
        ('codigo_unico', 'unique(codigo)', 'Ya existe un registro con ese código'),
    ]
    ```

**_inherit**

Para extender modelos existentes (se verá en detalle en el capítulo de Herencia):

!!!example "ejemplo"

    ```python
    _inherit = 'res.partner'  # Extiende el modelo de contactos
    ```

!!!example "Ejemplo Completo"

    ```python
    from odoo import models, fields, api
    from odoo.exceptions import ValidationError

    class Producto(models.Model):
        _name = 'inventario.producto'
        _description = 'Productos del inventario'
        _order = 'codigo, nombre'
        _rec_name = 'codigo'
        
        _sql_constraints = [
            ('codigo_unico', 'unique(codigo)', 'El código de producto debe ser único'),
        ]
        
        codigo = fields.Char(string='Código', required=True, index=True)
        nombre = fields.Char(string='Nombre', required=True)
        descripcion = fields.Text(string='Descripción')
        precio = fields.Float(string='Precio', digits=(10, 2))
        activo = fields.Boolean(string='Activo', default=True)
        
        @api.constrains('precio')
        def _check_precio(self):
            for producto in self:
                if producto.precio < 0:
                    raise ValidationError('El precio no puede ser negativo')
    ```

## Tipos de Campos Básicos

Odoo proporciona diversos tipos de campos para representar diferentes tipos de datos. Todos los campos se definen en el namespace `fields`.

### Campos de Texto

**Char**

Texto corto de una sola línea. Usa el atributo `size` para limitar la longitud:

!!!example "ejemplo"

    ```python
    nombre = fields.Char(string='Nombre', size=100, required=True)
    email = fields.Char(string='Email', size=255)
    codigo_postal = fields.Char(string='Código Postal', size=10)
    ```

**Text**

Texto largo multilínea sin límite de caracteres:

!!!example "ejemplo"

    ```python
    descripcion = fields.Text(string='Descripción')
    notas = fields.Text(string='Notas internas')
    ```

**Html**

Texto enriquecido con formato HTML. El cliente web proporciona un editor WYSIWYG:

!!!example "ejemplo"

    ```python
    contenido = fields.Html(string='Contenido de la página')
    ```

### Campos Numéricos

**Integer**

Números enteros:

!!!example "ejemplo"

    ```python
    cantidad = fields.Integer(string='Cantidad', default=0)
    stock = fields.Integer(string='Stock disponible')
    ```

**Float**

Números decimales. Usa `digits=(total, decimales)` para precisión:

!!!example "ejemplo"

    ```python
    precio = fields.Float(string='Precio', digits=(10, 2))  # 10 dígitos totales, 2 decimales
    peso = fields.Float(string='Peso (kg)', digits=(8, 3))
    ```

**Monetary**

Número decimal optimizado para importes monetarios. Requiere un campo de moneda asociado:

!!!example "ejemplo"

    ```python
    moneda_id = fields.Many2one('res.currency', string='Moneda')
    precio = fields.Monetary(string='Precio', currency_field='moneda_id')
    ```

### Campos de Fecha y Hora

**Date**

Solo fecha, sin hora:

!!!example "ejemplo"

    ```python
    fecha_nacimiento = fields.Date(string='Fecha de Nacimiento')
    fecha_alta = fields.Date(string='Fecha de Alta', default=fields.Date.today)
    ```

**Datetime**

Fecha y hora completas:

!!!example "ejemplo"

    ```python
    fecha_creacion = fields.Datetime(string='Fecha de Creación', default=fields.Datetime.now)
    fecha_inicio = fields.Datetime(string='Fecha de Inicio')
    ```

!!! tip "Valores por defecto con fechas"
    No uses `default=fields.Date.today()` (con paréntesis) porque ejecutaría la función al cargar el módulo. Usa `default=fields.Date.today` (sin paréntesis) o una lambda: `default=lambda self: fields.Date.today()`

### Campos Booleanos

**Boolean**

Valores verdadero/falso:

!!!example "ejemplo"

    ```python
    activo = fields.Boolean(string='Activo', default=True)
    es_cliente = fields.Boolean(string='Es Cliente')
    ```

### Campos de Selección

**Selection**

Lista cerrada de opciones. Puede definirse como lista de tuplas o función:

!!!example "ejemplo"

    ```python
    # Con lista de tuplas
    estado = fields.Selection([
        ('borrador', 'Borrador'),
        ('confirmado', 'Confirmado'),
        ('enviado', 'Enviado'),
        ('entregado', 'Entregado'),
    ], string='Estado', default='borrador')

    # Con función
    tipo = fields.Selection(selection='_get_tipos', string='Tipo')

    @api.model
    def _get_tipos(self):
        return [
            ('tipo1', 'Tipo 1'),
            ('tipo2', 'Tipo 2'),
        ]
    ```

Usar una función es útil cuando las opciones dependen de datos dinámicos o configuraciones.

### Campos Binarios

**Binary**

Almacena archivos binarios codificados en base64:

!!!example "ejemplo"

    ```python
    archivo_adjunto = fields.Binary(string='Archivo')
    ```

**Image**

Especialización de Binary para imágenes. Permite redimensionamiento automático:

!!!example "ejemplo"

    ```python
    imagen = fields.Image(string='Imagen', max_width=1024, max_height=1024)
    imagen_128 = fields.Image(string='Imagen pequeña', max_width=128, max_height=128)
    ```

Odoo genera automáticamente miniaturas si especificas diferentes tamaños.

### Atributos Comunes de Campos

Todos los campos pueden tener estos atributos:

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `string` | str | Etiqueta visible en la interfaz |
| `required` | bool | Si el campo es obligatorio |
| `readonly` | bool | Si el campo es de solo lectura |
| `help` | str | Texto de ayuda al pasar el cursor |
| `index` | bool | Crea índice en la base de datos |
| `default` | valor/función | Valor por defecto |
| `copy` | bool | Si se copia al duplicar el registro |
| `groups` | str | Grupos que pueden ver el campo |

**Ejemplo con varios atributos**:

!!!example "ejemplo"

    ```python
    nombre = fields.Char(
        string='Nombre Completo',
        required=True,
        index=True,
        help='Nombre y apellidos del contacto',
        copy=False,
        groups='base.group_user',
    )
    ```

## Campos Relacionales

Las relaciones entre modelos son fundamentales en Odoo. Permiten conectar entidades y navegar entre ellas de forma natural.

### Many2one (Muchos a Uno)

Representa una relación donde muchos registros del modelo actual apuntan a un registro de otro modelo.

**Ejemplo**: Muchas tareas pertenecen a un proyecto.

!!!example "ejemplo"

    ```python
    proyecto_id = fields.Many2one(
        'proyecto.proyecto',           # Modelo relacionado
        string='Proyecto',
        ondelete='restrict',           # Comportamiento al eliminar
        index=True,
    )
    ```

**Opciones de `ondelete`**:

- `restrict`: Impide eliminar si hay registros relacionados (por defecto)
- `cascade`: Elimina también los registros relacionados
- `set null`: Pone el campo a NULL al eliminar el relacionado

**Uso en código**:

!!!example "ejemplo"

    ```python
    # Acceder al registro relacionado
    print(tarea.proyecto_id.nombre)

    # Asignar una relación
    tarea.proyecto_id = proyecto_obj
    ```

### One2many (Uno a Muchos)

Representa la relación inversa de Many2one. Un registro tiene muchos registros relacionados.

**Ejemplo**: Un proyecto tiene muchas tareas.

!!!example "ejemplo"

    ```python
    # En el modelo Proyecto
    tarea_ids = fields.One2many(
        'proyecto.tarea',              # Modelo relacionado
        'proyecto_id',                 # Campo Many2one inverso
        string='Tareas',
    )
    ```

El segundo parámetro (`proyecto_id`) debe ser el nombre exacto del campo Many2one en el modelo relacionado.

**Uso en código**:

!!!example "ejemplo"

    ```python
    # Acceder a los registros relacionados
    for tarea in proyecto.tarea_ids:
        print(tarea.nombre)

    # Añadir un registro
    proyecto.tarea_ids = [(4, tarea.id)]  # Vincula una tarea existente
    proyecto.tarea_ids = [(0, 0, {'nombre': 'Nueva tarea'})]  # Crea y vincula
    ```

### Many2many (Muchos a Muchos)

Representa una relación donde múltiples registros de un modelo se relacionan con múltiples registros de otro modelo.

**Ejemplo**: Un producto puede tener múltiples etiquetas y una etiqueta puede estar en múltiples productos.

!!!example "ejemplo"

    ```python
    etiqueta_ids = fields.Many2many(
        'producto.etiqueta',           # Modelo relacionado
        'producto_etiqueta_rel',       # Tabla intermedia (opcional)
        'producto_id',                 # Columna de este modelo (opcional)
        'etiqueta_id',                 # Columna del otro modelo (opcional)
        string='Etiquetas',
    )
    ```

Si no especificas los parámetros opcionales, Odoo genera nombres automáticamente.

**Cuándo especificar tabla intermedia**:

- Cuando hay múltiples relaciones Many2many entre los mismos modelos
- Cuando quieres un nombre específico para la tabla
- Cuando el modelo se relaciona consigo mismo

**Uso en código**:

!!!example "ejemplo"

    ```python
    # Acceder a los registros relacionados
    for etiqueta in producto.etiqueta_ids:
        print(etiqueta.nombre)

    # Asignar múltiples registros
    producto.etiqueta_ids = [(6, 0, [etiqueta1.id, etiqueta2.id])]
    ```

### Comandos para Relaciones X2many

Odoo usa una sintaxis especial para modificar relaciones One2many y Many2many:

| Comando | Sintaxis | Acción |
|---------|----------|--------|
| Crear y vincular | `(0, 0, {valores})` | Crea un nuevo registro y lo vincula |
| Actualizar | `(1, id, {valores})` | Actualiza un registro ya vinculado |
| Desvincular y eliminar | `(2, id, 0)` | Desvincula y elimina el registro |
| Desvincular | `(3, id, 0)` | Desvincula sin eliminar |
| Vincular | `(4, id, 0)` | Vincula un registro existente |
| Desvincular todos | `(5, 0, 0)` | Desvincula todos los registros |
| Reemplazar | `(6, 0, [ids])` | Reemplaza todos con la lista de IDs |

**Ejemplos prácticos**:

!!!example "ejemplo"

    ```python
    # Crear y vincular una nueva tarea
    proyecto.tarea_ids = [(0, 0, {'nombre': 'Nueva tarea', 'descripcion': 'Descripción'})]

    # Vincular tareas existentes
    proyecto.tarea_ids = [(4, tarea1.id), (4, tarea2.id)]

    # Reemplazar todas las tareas
    proyecto.tarea_ids = [(6, 0, [tarea1.id, tarea2.id, tarea3.id])]

    # Desvincular y eliminar una tarea
    proyecto.tarea_ids = [(2, tarea_a_eliminar.id, 0)]

    # Desvincular todas las tareas
    proyecto.tarea_ids = [(5, 0, 0)]
    ```

### Related (Campos Relacionados)

Un campo Related es un atajo para acceder a un campo de un modelo relacionado sin escribir código Python.

!!!example "ejemplo"

    ```python
    # En lugar de acceder con código...
    proyecto_nombre = proyecto_id.nombre

    # Puedes definir un campo related
    proyecto_nombre = fields.Char(
        string='Nombre del Proyecto',
        related='proyecto_id.nombre',
        readonly=True,
        store=False,
    )
    ```

**Ventajas de Related**:

- Simplifica búsquedas y filtros en vistas
- Evita código Python repetitivo
- Permite usar el campo en domains y groupby

**Parámetro `store`**:

- `store=False` (por defecto): No ocupa espacio en BD, se calcula siempre
- `store=True`: Se guarda en BD, más rápido pero ocupa espacio

### Reference (Referencia)

Un campo Reference permite apuntar a registros de diferentes modelos:

!!!example "ejemplo"

    ```python
    documento_ref = fields.Reference(
        selection='_get_modelos_documento',
        string='Documento Relacionado',
    )

    @api.model
    def _get_modelos_documento(self):
        return [
            ('proyecto.proyecto', 'Proyecto'),
            ('proyecto.tarea', 'Tarea'),
            ('res.partner', 'Contacto'),
        ]
    ```

Es menos común que los otros campos relacionales, pero útil para relaciones polimórficas.

## Campos Computados

Los campos computados son aquellos cuyo valor se calcula automáticamente mediante una función Python en lugar de ser introducido por el usuario.

### Sintaxis Básica

!!!example "ejemplo"

    ```python
    # Campo computado
    total = fields.Float(string='Total', compute='_compute_total')

    # Método que lo calcula
    @api.depends('precio', 'cantidad')
    def _compute_total(self):
        for record in self:
            record.total = record.precio * record.cantidad
    ```

**Elementos clave**:

- `compute`: Nombre del método que calcula el valor
- `@api.depends()`: Lista de campos de los que depende el cálculo
- El método debe iterar sobre `self` (es un recordset)

### Almacenamiento de Campos Computados

**Sin almacenamiento** (`store=False`, por defecto):

- Se calcula cada vez que se lee el campo
- No ocupa espacio en base de datos
- No se puede buscar ni filtrar por él
- Ideal para cálculos rápidos

**Con almacenamiento** (`store=True`):

!!!example "ejemplo"

    ```python
    total = fields.Float(string='Total', compute='_compute_total', store=True)
    ```

- Se guarda en la base de datos
- Solo se recalcula cuando cambian las dependencias
- Permite búsquedas y filtros
- Ideal para cálculos costosos o campos usados en reportes

### Campos Computados con Tipos Relacionales

Los campos computados pueden ser de cualquier tipo, incluyendo relacionales:

**Many2one computado**:

!!!example "ejemplo"

    ```python
    proyecto_activo_id = fields.Many2one(
        'proyecto.proyecto',
        compute='_compute_proyecto_activo',
        string='Proyecto Activo',
    )

    @api.depends('usuario_id')
    def _compute_proyecto_activo(self):
        for record in self:
            # Buscar el proyecto activo del usuario
            proyecto = self.env['proyecto.proyecto'].search([
                ('usuario_id', '=', record.usuario_id.id),
                ('activo', '=', True)
            ], limit=1)
            record.proyecto_activo_id = proyecto.id if proyecto else False
    ```

**Many2many computado**:

!!!example "ejemplo"

    ```python
    tecnologias_ids = fields.Many2many(
        'proyecto.tecnologia',
        compute='_compute_tecnologias',
        string='Tecnologías Usadas',
    )

    @api.depends('tarea_ids', 'tarea_ids.tecnologia_ids')
    def _compute_tecnologias(self):
        for proyecto in self:
            # Recopilar todas las tecnologías de todas las tareas
            tecnologias = self.env['proyecto.tecnologia']
            for tarea in proyecto.tarea_ids:
                tecnologias |= tarea.tecnologia_ids
            proyecto.tecnologias_ids = tecnologias
    ```

### Campos Computados Editables

Por defecto, los campos computados son de solo lectura. Para hacerlos editables, define una función inversa:

!!!example "ejemplo"

    ```python
    nombre_completo = fields.Char(
        compute='_compute_nombre_completo',
        inverse='_inverse_nombre_completo',
        store=True,
    )

    @api.depends('nombre', 'apellidos')
    def _compute_nombre_completo(self):
        for record in self:
            record.nombre_completo = f"{record.nombre} {record.apellidos}"

    def _inverse_nombre_completo(self):
        for record in self:
            if record.nombre_completo:
                partes = record.nombre_completo.split(' ', 1)
                record.nombre = partes[0]
                record.apellidos = partes[1] if len(partes) > 1 else ''
    ```

### Búsquedas en Campos Computados

Para permitir búsquedas en campos computados no almacenados, define un método de búsqueda:

!!!example "ejemplo"

    ```python
    edad = fields.Integer(compute='_compute_edad', search='_search_edad')

    @api.depends('fecha_nacimiento')
    def _compute_edad(self):
        hoy = fields.Date.today()
        for record in self:
            if record.fecha_nacimiento:
                record.edad = (hoy - record.fecha_nacimiento).days // 365
            else:
                record.edad = 0

    def _search_edad(self, operator, value):
        # Convertir búsqueda de edad en búsqueda de fecha_nacimiento
        hoy = fields.Date.today()
        fecha_limite = hoy - timedelta(days=value * 365)
        
        if operator == '=':
            # Edad exacta es complejo, aproximamos con rango
            return [('fecha_nacimiento', '>=', fecha_limite - timedelta(days=365)),
                    ('fecha_nacimiento', '<', fecha_limite)]
        elif operator == '>':
            return [('fecha_nacimiento', '<', fecha_limite)]
        elif operator == '<':
            return [('fecha_nacimiento', '>', fecha_limite)]
        # ... otros operadores
    ```

## Valores por Defecto

Los valores por defecto se asignan automáticamente al crear nuevos registros. Pueden ser estáticos o calculados dinámicamente.

### Valores Estáticos

!!!example "ejemplo"

    ```python
    activo = fields.Boolean(default=True)
    cantidad = fields.Integer(default=1)
    estado = fields.Selection([...], default='borrador')
    ```

### Valores Calculados con Función

!!!example "ejemplo"

    ```python
    fecha_alta = fields.Date(default=lambda self: fields.Date.today())
    usuario_id = fields.Many2one('res.users', default=lambda self: self.env.user)
    ```

!!! warning "Funciones vs Lambdas para valores por defecto"
    ```python
    # ❌ INCORRECTO - ejecuta al cargar el módulo
    fecha = fields.Date(default=fields.Date.today())
    
    # ✅ CORRECTO - ejecuta al crear el registro
    fecha = fields.Date(default=lambda self: fields.Date.today())
    ```

### Función de Valor por Defecto Compleja

!!!example "ejemplo"

    ```python
    def _default_codigo(self):
        # Generar código secuencial
        ultimo = self.search([], order='id desc', limit=1)
        if ultimo and ultimo.codigo:
            numero = int(ultimo.codigo.split('-')[1]) + 1
            return f"PROD-{numero:05d}"
        return "PROD-00001"

    codigo = fields.Char(default=_default_codigo)
    ```

### Método default_get

Para valores por defecto complejos o que dependen del contexto:

!!!example "ejemplo"

    ```python
    @api.model
    def default_get(self, fields_list):
        defaults = super(MiModelo, self).default_get(fields_list)
        
        # Valor por defecto según contexto
        if self._context.get('desde_proyecto'):
            proyecto_id = self._context.get('proyecto_id')
            defaults['proyecto_id'] = proyecto_id
        
        return defaults
    ```

## Restricciones y Validaciones

Las restricciones aseguran la integridad de los datos. Odoo proporciona dos tipos: restricciones Python y restricciones SQL.

### Restricciones Python (@api.constrains)

Se ejecutan en Python antes de guardar y permiten lógica compleja:

!!!example "ejemplo"

    ```python
    from odoo.exceptions import ValidationError

    @api.constrains('edad')
    def _check_edad(self):
        for record in self:
            if record.edad < 0:
                raise ValidationError('La edad no puede ser negativa')
            if record.edad > 150:
                raise ValidationError('La edad no puede superar 150 años')
    ```

**Características**:

- Se ejecutan solo cuando cambian los campos especificados
- Permiten lógica compleja y acceso a otros modelos
- Mensajes de error personalizados en cualquier idioma
- Se ejecutan en el servidor, no en la base de datos

**Validar múltiples campos**:

!!!example "ejemplo"

    ```python
    @api.constrains('fecha_inicio', 'fecha_fin')
    def _check_fechas(self):
        for record in self:
            if record.fecha_fin and record.fecha_inicio:
                if record.fecha_fin < record.fecha_inicio:
                    raise ValidationError(
                        'La fecha de fin no puede ser anterior a la fecha de inicio'
                    )
    ```

**Validaciones complejas**:

!!!example "ejemplo"

    ```python
    @api.constrains('precio', 'precio_coste')
    def _check_margenes(self):
        for record in self:
            if record.precio_coste and record.precio:
                margen = ((record.precio - record.precio_coste) / record.precio) * 100
                if margen < 10:
                    raise ValidationError(
                        f'El margen ({margen:.1f}%) es inferior al mínimo (10%)'
                    )
    ```

### Restricciones SQL (_sql_constraints)

Se implementan directamente en PostgreSQL y son más eficientes:

!!!example "ejemplo"

    ```python
    _sql_constraints = [
        ('nombre_unico', 
         'unique(nombre)', 
         'Ya existe un registro con ese nombre'),
        
        ('codigo_unico', 
         'unique(codigo)', 
         'El código debe ser único'),
        
        ('precio_positivo', 
         'CHECK(precio >= 0)', 
         'El precio no puede ser negativo'),
    ]
    ```

**Sintaxis**: `('nombre_restriccion', 'sql_constraint', 'mensaje_error')`

**Tipos comunes**:

- `unique(campo)`: Unicidad
- `unique(campo1, campo2)`: Unicidad compuesta
- `CHECK(condicion)`: Condición personalizada

### Comparativa: Python vs SQL

| Aspecto | Python (@api.constrains) | SQL (_sql_constraints) |
|---------|--------------------------|------------------------|
| **Rendimiento** | Más lento | Más rápido |
| **Complejidad** | Lógica compleja | Lógica simple |
| **Acceso a otros modelos** | Sí | No |
| **Mensaje personalizado** | Muy flexible | Limitado |
| **Garantía BD** | No (solo en Odoo) | Sí (nivel BD) |
| **Uso típico** | Validaciones de negocio | Unicidad, rangos |

**Recomendación**: Usa SQL para restricciones simples y críticas (unicidad, no nulos). Usa Python para validaciones de negocio complejas.

!!!tip "Buenas Prácticas"

    Al desarrollar modelos, sigue estas recomendaciones:

    **Nomenclatura**

    - Nombres de modelo en minúsculas con puntos: `mi_modulo.mi_modelo`
    - Nombres de campos en minúsculas con guiones bajos: `fecha_inicio`
    - Sufijos para campos relacionales: `_id` (Many2one), `_ids` (X2many)
    - Nombres descriptivos: `cliente_id` mejor que `c_id`

    **Organización de campos**

    !!!example "ejemplo"

        ```python
        class MiModelo(models.Model):
            _name = 'mi.modelo'
            
            # 1. Campos básicos primero
            nombre = fields.Char()
            descripcion = fields.Text()
            
            # 2. Campos numéricos
            cantidad = fields.Integer()
            precio = fields.Float()
            
            # 3. Campos de fecha
            fecha_inicio = fields.Date()
            
            # 4. Campos booleanos y selection
            activo = fields.Boolean()
            estado = fields.Selection()
            
            # 5. Campos relacionales
            cliente_id = fields.Many2one()
            linea_ids = fields.One2many()
            
            # 6. Campos computados al final
            total = fields.Float(compute='_compute_total')
        ```

    **Documentación**

    - Usa `string` descriptivo en todos los campos
    - Añade `help` en campos no obvios
    - Documenta `_description` del modelo
    - Comenta lógica compleja en métodos

    **Índices**

    - Añade `index=True` a campos usados frecuentemente en búsquedas
    - Campos Many2one ya tienen índice por defecto
    - No abuses de índices (ralentizan escrituras)

    **Valores por defecto**

    - Establece valores por defecto sensatos
    - Usa funciones lambda para valores dinámicos
    - No uses valores por defecto en campos relacionales obligatorios

    **Restricciones**

    - Usa SQL para restricciones simples y críticas
    - Usa Python para lógica de negocio compleja
    - Mensajes de error claros y útiles para el usuario
    - Valida solo lo necesario (evita validaciones redundantes)

    **Campos computados**

    - Usa `store=True` solo cuando sea necesario
    - Declara todas las dependencias en `@api.depends`
    - Itera siempre sobre `self` en métodos compute
    - Inicializa el campo al principio del bucle