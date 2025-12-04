---
title: Lógica de Negocio
description: API del ORM, recordsets, decoradores y métodos en Odoo
---

# Lógica de Negocio

La lógica de negocio es el corazón de cualquier aplicación Odoo. Es donde defines cómo se comportan tus modelos, cómo responden a las acciones del usuario y cómo interactúan entre sí. El ORM de Odoo proporciona una API potente y elegante que permite escribir código Python que se traduce automáticamente en operaciones de base de datos.

En este capítulo aprenderás a dominar la API del ORM, entender cómo funcionan los recordsets, usar decoradores para controlar el comportamiento de tus métodos, y trabajar con fechas y excepciones de forma eficiente.

## API del ORM

El ORM (Object-Relational Mapping) de Odoo proporciona métodos para interactuar con la base de datos sin escribir SQL. Todo el código se escribe en Python y el ORM se encarga de traducirlo a consultas SQL eficientes.

### El Entorno (Environment)

El entorno (`env`) es el punto de acceso a todo el sistema Odoo desde Python. Contiene:

- Acceso a todos los modelos
- Información del usuario actual
- El cursor de base de datos
- El contexto actual
- 
```python
# Acceder a un modelo
Partner = self.env['res.partner']

# Información del usuario actual
usuario_actual = self.env.user
print(usuario_actual.name)  # "Administrator"

# Acceder al contexto
idioma = self.env.context.get('lang')  # 'es_ES'

# Acceder a la empresa actual
empresa = self.env.company
```

### Recordsets

Un recordset es una colección ordenada de registros de un mismo modelo. Es similar a una lista de Python pero con capacidades especiales del ORM.

**Características de los recordsets**:

- Son iterables
- No tienen elementos duplicados
- Soportan operaciones de conjuntos
- Permiten acceder a campos cuando son singletons

!!! example "Trabajar con recordsets"
    ```python
    # Crear un recordset vacío
    productos = self.env['mi.producto']
    
    # Buscar productos (devuelve un recordset)
    productos = self.env['mi.producto'].search([('activo', '=', True)])
    
    # Iterar sobre recordsets
    for producto in productos:
        print(producto.nombre)  # producto es un singleton
    
    # Acceder al primer elemento
    primer_producto = productos[0]  # singleton
    
    # Longitud
    cantidad = len(productos)
    
    # Verificar si está vacío
    if productos:
        print("Hay productos")
    
    # IDs de los registros
    ids = productos.ids  # [1, 2, 3, 4]
    ```

### Singletons

Un singleton es un recordset con exactamente un registro. La mayoría de operaciones sobre campos solo funcionan en singletons.

!!! example "Singleton vs Recordset"
    ```python
    # Recordset con múltiples registros
    productos = self.env['mi.producto'].search([])
    print(productos)  # mi.producto(1, 2, 3, 4, 5)
    
    # Error: no puedes acceder a campos en recordsets múltiples
    # print(productos.nombre)  # ¡Error!
    
    # Iterar para obtener singletons
    for producto in productos:
        print(producto)  # mi.producto(1), luego mi.producto(2), etc.
        print(producto.nombre)  # Ahora sí funciona
    
    # Asegurar que es singleton
    producto = productos[0]  # Obtener el primero
    producto.ensure_one()  # Lanza error si no es singleton
    print(producto.nombre)  # Funciona
    ```

### Operaciones con Conjuntos

Los recordsets soportan operaciones matemáticas de conjuntos:

!!! example "Operaciones de conjuntos"
    ```python
    productos1 = self.env['mi.producto'].search([('categoria_id', '=', 1)])
    productos2 = self.env['mi.producto'].search([('precio', '>', 100)])
    
    # Unión (|)
    todos = productos1 | productos2
    
    # Intersección (&)
    comunes = productos1 & productos2
    
    # Diferencia (-)
    solo_cat1 = productos1 - productos2
    
    # Verificar pertenencia
    if producto in productos1:
        print("El producto está en la categoría 1")
    
    # Concatenar (mantiene orden, permite duplicados temporalmente)
    concatenados = productos1 + productos2
    ```

## Métodos CRUD

CRUD son las operaciones básicas: Create, Read, Update, Delete. El ORM proporciona métodos específicos para cada una.

### search() - Buscar Registros

Busca registros que cumplan un dominio y devuelve un recordset.

!!! example "Método search()"
    ```python
    # Búsqueda simple
    productos = self.env['mi.producto'].search([('activo', '=', True)])
    
    # Con límite
    ultimos_5 = self.env['mi.producto'].search([], limit=5, order='create_date desc')
    
    # Con offset (paginación)
    pagina_2 = self.env['mi.producto'].search([], limit=10, offset=10)
    
    # Solo contar
    cantidad = self.env['mi.producto'].search_count([('activo', '=', True)])
    
    # Ordenar
    ordenados = self.env['mi.producto'].search(
        [('categoria_id', '=', 1)],
        order='nombre asc, precio desc'
    )
    ```

**Parámetros de search()**:

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `args` | list | Dominio de búsqueda |
| `offset` | int | Número de resultados a saltar |
| `limit` | int | Número máximo de resultados |
| `order` | str | Campos de ordenación |
| `count` | bool | Solo devuelve el número de registros |

### create() - Crear Registros

Crea un nuevo registro con los valores especificados.

!!! example "Método create()"
    ```python
    # Crear un registro
    producto = self.env['mi.producto'].create({
        'nombre': 'Laptop Dell',
        'codigo': 'LAP-001',
        'precio': 1299.99,
        'activo': True,
    })
    print(producto.id)  # 42
    
    # Crear múltiples registros
    productos = self.env['mi.producto'].create([
        {'nombre': 'Producto 1', 'precio': 10.0},
        {'nombre': 'Producto 2', 'precio': 20.0},
        {'nombre': 'Producto 3', 'precio': 30.0},
    ])
    print(productos.ids)  # [43, 44, 45]
    
    # Con relaciones Many2one
    producto = self.env['mi.producto'].create({
        'nombre': 'Mouse',
        'categoria_id': categoria.id,  # ID de un registro existente
        'proveedor_id': self.env.ref('mi_modulo.proveedor_principal').id,
    })
    
    # Con relaciones Many2many
    producto = self.env['mi.producto'].create({
        'nombre': 'Teclado',
        'etiqueta_ids': [(6, 0, [etiqueta1.id, etiqueta2.id])],
    })
    ```

### write() - Actualizar Registros

Actualiza los campos de todos los registros en el recordset.

!!! example "Método write()"
    ```python
    # Actualizar un registro
    producto = self.env['mi.producto'].browse(42)
    producto.write({
        'precio': 1199.99,
        'activo': True,
    })
    
    # Actualizar múltiples registros
    productos = self.env['mi.producto'].search([('categoria_id', '=', 1)])
    productos.write({
        'descuento': 0.10,
    })
    
    # Asignación directa (más Pythónico)
    producto.precio = 1199.99
    producto.activo = True
    
    # Para Many2many
    producto.etiqueta_ids = [(6, 0, [id1, id2, id3])]
    # O más simple
    producto.etiqueta_ids = etiquetas  # recordset
    ```

### browse() - Obtener Registros por ID

Crea un recordset a partir de una lista de IDs.

!!! example "Método browse()"
    ```python
    # Un solo ID
    producto = self.env['mi.producto'].browse(42)
    
    # Múltiples IDs
    productos = self.env['mi.producto'].browse([1, 2, 3, 4, 5])
    
    # Desde el contexto
    producto_id = self.env.context.get('producto_activo_id')
    if producto_id:
        producto = self.env['mi.producto'].browse(producto_id)
    
    # Verificar existencia
    producto = self.env['mi.producto'].browse(42)
    if producto.exists():
        print(producto.nombre)
    ``` 

### unlink() - Eliminar Registros

Elimina los registros del recordset de la base de datos.

!!! example "Método unlink()"
    ```python
    # Eliminar un registro
    producto = self.env['mi.producto'].browse(42)
    producto.unlink()
    
    # Eliminar múltiples registros
    productos = self.env['mi.producto'].search([('activo', '=', False)])
    productos.unlink()
    
    # Con verificación
    if producto.exists():
        producto.unlink()
    ```

!!! warning "Precaución con unlink()"
    - No se puede deshacer
    - Puede fallar si hay restricciones de integridad
    - Dispara los métodos de eliminación en cascada si están definidos

### ref() - Buscar por External ID

Obtiene un registro usando su identificador externo.

!!! example "Método ref()"
    ```python
    # Buscar por External ID
    usuario_admin = self.env.ref('base.user_admin')
    print(usuario_admin.name)  # "Administrator"
    
    # Con módulo explícito
    categoria = self.env.ref('mi_modulo.categoria_electronica')
    
    # Con raise_if_not_found=False (no lanza error si no existe)
    registro = self.env.ref('mi_modulo.registro_opcional', raise_if_not_found=False)
    if registro:
        print(registro.name)
    ```

## Programación Funcional en el ORM

Odoo proporciona métodos inspirados en programación funcional para trabajar con recordsets de forma elegante y eficiente.

### filtered() - Filtrar Recordsets

Filtra un recordset devolviendo solo los registros que cumplen una condición.

!!! example "Método filtered()"
    ```python
    productos = self.env['mi.producto'].search([])
    
    # Con función lambda
    activos = productos.filtered(lambda p: p.activo)
    caros = productos.filtered(lambda p: p.precio > 1000)
    
    # Con expresión de cadena (acceso a campos relacionados)
    con_stock = productos.filtered('stock')
    categoria_especifica = productos.filtered(lambda p: p.categoria_id.nombre == 'Electrónica')
    
    # Condiciones múltiples
    productos_validos = productos.filtered(
        lambda p: p.activo and p.stock > 0 and p.precio > 0
    )
    
    # Con campo relacional
    productos_empresa = productos.filtered('categoria_id.empresa_id')
    ```

### mapped() - Transformar Recordsets

Aplica una función a cada registro y devuelve una lista o recordset con los resultados.

!!! example "Método mapped()"
    ```python
    productos = self.env['mi.producto'].search([])
    
    # Obtener lista de nombres
    nombres = productos.mapped('nombre')
    # ['Laptop', 'Mouse', 'Teclado']
    
    # Obtener lista de precios
    precios = productos.mapped('precio')
    # [1299.99, 29.99, 89.99]
    
    # Acceder a campos relacionados (devuelve recordset)
    categorias = productos.mapped('categoria_id')
    # producto.categoria(1, 2, 3) - sin duplicados
    
    # Cadena de relaciones
    empresas = productos.mapped('categoria_id.empresa_id')
    
    # Con función lambda (devuelve lista)
    precios_con_iva = productos.mapped(lambda p: p.precio * 1.21)
    
    # Cálculos complejos
    margenes = productos.mapped(lambda p: (p.precio - p.coste) / p.precio * 100)
    ```

### sorted() - Ordenar Recordsets

Ordena un recordset según una función de ordenación.

!!! example "Método sorted()"
    ```python
    productos = self.env['mi.producto'].search([])
    
    # Ordenar por campo simple
    por_nombre = productos.sorted(key=lambda p: p.nombre)
    por_precio = productos.sorted(key=lambda p: p.precio)
    
    # Orden descendente
    mas_caros = productos.sorted(key=lambda p: p.precio, reverse=True)
    
    # Ordenar por múltiples criterios
    ordenados = productos.sorted(key=lambda p: (p.categoria_id.nombre, -p.precio))
    
    # Ordenar por campo relacional
    por_categoria = productos.sorted(key=lambda p: p.categoria_id.sequence)
    
    # Sin clave (usa el _order del modelo)
    ordenados_default = productos.sorted()
    ```

## Decoradores del ORM

Los decoradores modifican cómo se ejecutan los métodos en Odoo. Son fundamentales para definir el comportamiento correcto de campos computados, validaciones y respuestas a cambios.

### @api.depends() - Dependencias de Campos Computados

Especifica de qué campos depende un campo computado. El método se ejecuta automáticamente cuando cambian las dependencias.

!!! example "Decorador @api.depends"
    ```python
    from odoo import models, fields, api
    
    class Producto(models.Model):
        _name = 'mi.producto'
        
        precio = fields.Float('Precio')
        cantidad = fields.Integer('Cantidad')
        descuento = fields.Float('Descuento %')
        
        total = fields.Float('Total', compute='_compute_total', store=True)
        
        @api.depends('precio', 'cantidad', 'descuento')
        def _compute_total(self):
            for record in self:
                subtotal = record.precio * record.cantidad
                record.total = subtotal * (1 - record.descuento / 100)
    ```

**Dependencias en campos relacionados**:

!!! example "Depends con campos relacionados"
    ```python
    @api.depends('linea_ids.subtotal')
    def _compute_total_pedido(self):
        for pedido in self:
            pedido.total = sum(pedido.linea_ids.mapped('subtotal'))
    
    @api.depends('partner_id.country_id')
    def _compute_zona_geografica(self):
        for record in self:
            if record.partner_id.country_id.code in ['ES', 'PT']:
                record.zona = 'iberia'
    ```

### `@api.onchange()` - Reaccionar a Cambios en el Formulario

Ejecuta código cuando el usuario cambia un campo en el formulario, antes de guardar.

!!! example "Decorador @api.onchange"
    ```python
    class Pedido(models.Model):
        _name = 'mi.pedido'
        
        partner_id = fields.Many2one('res.partner')
        direccion_envio = fields.Char()
        producto_id = fields.Many2one('mi.producto')
        cantidad = fields.Integer(default=1)
        precio_unitario = fields.Float()
        
        @api.onchange('partner_id')
        def _onchange_partner(self):
            if self.partner_id:
                # Rellenar automáticamente la dirección
                self.direccion_envio = self.partner_id.street
        
        @api.onchange('producto_id')
        def _onchange_producto(self):
            if self.producto_id:
                # Rellenar precio del producto
                self.precio_unitario = self.producto_id.precio
                
                # Mostrar advertencia si no hay stock
                if self.producto_id.stock < self.cantidad:
                    return {
                        'warning': {
                            'title': 'Stock Insuficiente',
                            'message': f'Solo hay {self.producto_id.stock} unidades disponibles'
                        }
                    }
        
        @api.onchange('cantidad', 'precio_unitario')
        def _onchange_calcular_total(self):
            # Recalcular cuando cambia cantidad o precio
            # (Esto se podría hacer mejor con un computed)
            self.total = self.cantidad * self.precio_unitario
    ```

**Retornar advertencias**:

!!! example "Warnings en onchange"
    ```python
    @api.onchange('fecha_inicio', 'fecha_fin')
    def _onchange_fechas(self):
        if self.fecha_fin and self.fecha_inicio:
            if self.fecha_fin < self.fecha_inicio:
                self.fecha_fin = self.fecha_inicio
                return {
                    'warning': {
                        'title': 'Fechas Incorrectas',
                        'message': 'La fecha de fin no puede ser anterior a la de inicio. Se ha ajustado automáticamente.',
                        'type': 'notification'
                    }
                }
    ```

### `@api.constrains()` - Validaciones

Valida que los datos cumplan ciertas reglas antes de guardarse.

!!! example "Decorador @api.constrains"
    ```python
    from odoo.exceptions import ValidationError
    
    class Producto(models.Model):
        _name = 'mi.producto'
        
        nombre = fields.Char()
        precio = fields.Float()
        coste = fields.Float()
        stock = fields.Integer()
        stock_minimo = fields.Integer()
        
        @api.constrains('precio')
        def _check_precio(self):
            for record in self:
                if record.precio < 0:
                    raise ValidationError('El precio no puede ser negativo')
        
        @api.constrains('precio', 'coste')
        def _check_margen(self):
            for record in self:
                if record.coste and record.precio:
                    if record.precio < record.coste:
                        raise ValidationError(
                            f'El precio ({record.precio}) no puede ser menor '
                            f'que el coste ({record.coste})'
                        )
        
        @api.constrains('stock', 'stock_minimo')
        def _check_stock(self):
            for record in self:
                if record.stock_minimo < 0:
                    raise ValidationError('El stock mínimo no puede ser negativo')
                if record.stock < 0:
                    raise ValidationError('El stock no puede ser negativo')
    ```

### `@api.model` - Métodos de Clase

Define métodos que operan sobre el modelo en general, no sobre registros específicos.

!!! example "Decorador @api.model"
    ```python
    class Producto(models.Model):
        _name = 'mi.producto'
        
        @api.model
        def obtener_estadisticas(self):
            """Devuelve estadísticas generales de productos"""
            return {
                'total': self.search_count([]),
                'activos': self.search_count([('activo', '=', True)]),
                'sin_stock': self.search_count([('stock', '=', 0)]),
                'precio_medio': sum(self.search([]).mapped('precio')) / self.search_count([]),
            }
        
        @api.model
        def create(self, vals):
            # Sobrescribir create para añadir lógica
            if 'codigo' not in vals:
                # Generar código automático
                vals['codigo'] = self._generar_codigo()
            return super(Producto, self).create(vals)
        
        @api.model
        def _generar_codigo(self):
            """Genera un código secuencial"""
            ultimo = self.search([], order='id desc', limit=1)
            if ultimo and ultimo.codigo:
                numero = int(ultimo.codigo.split('-')[1]) + 1
            else:
                numero = 1
            return f"PROD-{numero:05d}"
    ```

### @api.model_create_multi - Crear Múltiples Registros

Optimiza la creación de múltiples registros a la vez.

!!! example "Decorador @api.model_create_multi"
    ```python
    class Producto(models.Model):
        _name = 'mi.producto'
        
        @api.model_create_multi
        def create(self, vals_list):
            # vals_list es una lista de diccionarios
            for vals in vals_list:
                if 'codigo' not in vals:
                    vals['codigo'] = self._generar_codigo()
            
            return super(Producto, self).create(vals_list)
    ```

## Sobrescribir Métodos del ORM

Puedes personalizar el comportamiento de los métodos CRUD heredando y sobrescribiendo.

### Sobrescribir create()

!!! example "Sobrescribir create()"
    ```python
    class Pedido(models.Model):
        _name = 'mi.pedido'
        
        @api.model_create_multi
        def create(self, vals_list):
            for vals in vals_list:
                # Generar número de pedido
                if 'numero' not in vals:
                    vals['numero'] = self.env['ir.sequence'].next_by_code('mi.pedido')
                
                # Establecer estado inicial
                if 'estado' not in vals:
                    vals['estado'] = 'borrador'
            
            # Llamar al create() original
            pedidos = super(Pedido, self).create(vals_list)
            
            # Acciones post-creación
            for pedido in pedidos:
                pedido._enviar_notificacion_creacion()
            
            return pedidos
    ```

### Sobrescribir write()

!!! example "Sobrescribir write()"
    ```python
    class Pedido(models.Model):
        _name = 'mi.pedido'
        
        def write(self, vals):
            # Lógica antes de guardar
            if 'estado' in vals and vals['estado'] == 'confirmado':
                # Verificar que tiene líneas
                if not self.linea_ids:
                    raise ValidationError('No se puede confirmar un pedido sin líneas')
            
            # Guardar valores anteriores si necesario
            estados_anteriores = {pedido.id: pedido.estado for pedido in self}
            
            # Llamar al write() original
            resultado = super(Pedido, self).write(vals)
            
            # Lógica después de guardar
            if 'estado' in vals:
                for pedido in self:
                    if estados_anteriores[pedido.id] != pedido.estado:
                        pedido._notificar_cambio_estado(estados_anteriores[pedido.id])
            
            return resultado
    ```

### Sobrescribir unlink()

!!! example "Sobrescribir unlink()"
    ```python
    class Pedido(models.Model):
        _name = 'mi.pedido'
        
        def unlink(self):
            # Verificaciones antes de eliminar
            for pedido in self:
                if pedido.estado in ['confirmado', 'enviado']:
                    raise ValidationError(
                        f'No se puede eliminar el pedido {pedido.numero} '
                        f'porque está en estado {pedido.estado}'
                    )
            
            # Eliminar registros relacionados manualmente si es necesario
            self.mapped('linea_ids').unlink()
            
            # Llamar al unlink() original
            return super(Pedido, self).unlink()
    ```

## Trabajo con Fechas

Odoo proporciona utilidades para trabajar con fechas y horas de forma sencilla.

### Tipos de Campos de Fecha

```python
fecha_alta = fields.Date()       # Solo fecha
fecha_hora = fields.Datetime()   # Fecha y hora
```

### Obtener Fecha/Hora Actual

!!! example "Fechas actuales"
    ```python
    from odoo import fields
    
    # Fecha actual
    hoy = fields.Date.today()
    print(hoy)  # '2024-12-03'
    
    # Fecha y hora actual
    ahora = fields.Datetime.now()
    print(ahora)  # '2024-12-03 15:30:45'
    ```

### Convertir entre String y Datetime

!!! example "Conversión de fechas"
    ```python
    from odoo import fields
    from datetime import datetime, timedelta
    
    # String a Date
    fecha_str = '2024-12-03'
    fecha = fields.Date.from_string(fecha_str)
    # o directamente
    fecha = fields.Date.to_date(fecha_str)
    
    # Date a String
    fecha_str = fields.Date.to_string(fecha)
    
    # String a Datetime
    datetime_str = '2024-12-03 15:30:45'
    dt = fields.Datetime.from_string(datetime_str)
    # o
    dt = fields.Datetime.to_datetime(datetime_str)
    
    # Datetime a String
    datetime_str = fields.Datetime.to_string(dt)
    ```

### Sumar/Restar Tiempo

!!! example "Operaciones con fechas"
    ```python
    from datetime import datetime, timedelta
    from odoo import fields
    
    # Obtener fecha actual
    hoy = fields.Date.today()
    
    # Sumar días
    manana = hoy + timedelta(days=1)
    proxima_semana = hoy + timedelta(weeks=1)
    proximo_mes = hoy + timedelta(days=30)
    
    # Restar días
    ayer = hoy - timedelta(days=1)
    hace_una_semana = hoy - timedelta(weeks=1)
    
    # Con horas
    ahora = fields.Datetime.now()
    en_2_horas = ahora + timedelta(hours=2)
    hace_30_minutos = ahora - timedelta(minutes=30)
    ```

### Diferencia entre Fechas

!!! example "Calcular diferencias"
    ```python
    from datetime import datetime
    from odoo import fields
    
    fecha_inicio = fields.Date.from_string('2024-01-01')
    fecha_fin = fields.Date.from_string('2024-12-31')
    
    # Diferencia (devuelve timedelta)
    diferencia = fecha_fin - fecha_inicio
    
    # Días de diferencia
    dias = diferencia.days  # 365
    
    # Horas de diferencia
    horas = diferencia.total_seconds() / 3600
    
    # Semanas
    semanas = diferencia.days / 7
    ```

### Usar relativedelta para Meses/Años

!!! example "relativedelta"
    ```python
    from dateutil.relativedelta import relativedelta
    from odoo import fields
    
    hoy = fields.Date.today()
    
    # Sumar meses
    proximo_mes = hoy + relativedelta(months=1)
    dentro_6_meses = hoy + relativedelta(months=6)
    
    # Sumar años
    proximo_ano = hoy + relativedelta(years=1)
    
    # Combinaciones
    fecha_futura = hoy + relativedelta(years=1, months=6, days=15)
    
    # Diferencia exacta
    fecha_inicio = fields.Date.from_string('2020-01-15')
    fecha_fin = fields.Date.from_string('2024-12-03')
    
    diff = relativedelta(fecha_fin, fecha_inicio)
    print(f"{diff.years} años, {diff.months} meses, {diff.days} días")
    ```

### Comparar Fechas

!!! example "Comparación de fechas"
    ```python
    from odoo import fields
    
    fecha1 = fields.Date.from_string('2024-01-01')
    fecha2 = fields.Date.from_string('2024-12-31')
    
    if fecha1 < fecha2:
        print("fecha1 es anterior")
    
    if fecha2 > fecha1:
        print("fecha2 es posterior")
    
    if fecha1 == fields.Date.today():
        print("Es hoy")
    
    # Comparar solo fecha (ignorando hora)
    dt1 = fields.Datetime.now()
    dt2 = fields.Datetime.now() + timedelta(hours=5)
    
    if dt1.date() == dt2.date():
        print("Es el mismo día")
    ```

## Excepciones y Mensajes

Odoo proporciona excepciones específicas para diferentes situaciones.

### Tipos de Excepciones

!!! example "Excepciones de Odoo"
    ```python
    from odoo.exceptions import (
        ValidationError,
        UserError,
        AccessError,
        Warning,
        RedirectWarning
    )
    
    # ValidationError - Para validaciones de negocio
    @api.constrains('edad')
    def _check_edad(self):
        for record in self:
            if record.edad < 18:
                raise ValidationError('Debe ser mayor de 18 años')
    
    # UserError - Para errores generales
    def confirmar_pedido(self):
        if not self.linea_ids:
            raise UserError('No se puede confirmar un pedido vacío')
    
    # AccessError - Para problemas de permisos
    def eliminar_factura(self):
        if not self.env.user.has_group('account.group_account_manager'):
            raise AccessError('No tiene permisos para eliminar facturas')
    
    # Warning - Para advertencias
    def action_archivar(self):
        if self.tiene_ventas_activas:
            raise Warning('Este producto tiene ventas activas')
    ```

### RedirectWarning con Acción

!!! example "RedirectWarning"
    ```python
    from odoo.exceptions import RedirectWarning

    def confirmar_pedido(self):
    if not self.cliente_id.direccion_envio:
        # Redirigir al formulario del cliente
        action = self.env.ref('base.action_partner_form')
        message = (
            'El cliente no tiene dirección de envío configurada. '
            'Por favor, complete la información del cliente.'
        )
        raise RedirectWarning(
            message,
            action.id,
            'Ir a Cliente'
        )
    ```


!!! example "Mensajes no bloqueantes"
    ```python
    # Usando el sistema de mensajería de Odoo
    def action_procesar(self):
        # Lógica de procesamiento
        self.estado = 'procesado'
        
        # Enviar mensaje al chatter
        self.message_post(
            body='El registro ha sido procesado correctamente',
            subject='Procesamiento Completado'
        )
        
        # Notificación al usuario
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'title': 'Éxito',
                'message': 'El proceso se completó correctamente',
                'type': 'success',  # success, warning, danger, info
                'sticky': False,  # Si se cierra automáticamente
            }
        }
    ```

!!!tip "Buenas Prácticas"

    **Recordsets**

    - Siempre itera sobre `self` en métodos que operan sobre múltiples registros
    - Usa `ensure_one()` cuando el método solo debe trabajar con un registro
    - Aprovecha operaciones de conjuntos en lugar de loops complejos
    - Usa `exists()` antes de acceder a registros que podrían haber sido eliminados

    **Métodos CRUD**

    - Usa `search_count()` cuando solo necesites la cantidad
    - Especifica `limit` en búsquedas que puedan devolver muchos registros
    - Usa `browse()` cuando ya tienes los IDs
    - Prefiere asignación directa (`record.campo = valor`) sobre `write()` cuando sea más claro

    **Decoradores**

    - Declara todas las dependencias en `@api.depends()`
    - Usa `@api.onchange` para UX, no para lógica crítica
    - Prefiere `@api.constrains` para validaciones sobre sobrescribir `write()`
    - Documenta métodos decorados con docstrings

    **Fechas**

    - Usa siempre las utilidades de Odoo (`fields.Date`, `fields.Datetime`)
    - No mezcles fechas timezone-aware con naive
    - Usa `relativedelta` para operaciones con meses/años
    - Almacena fechas en UTC en la base de datos

    **Excepciones**

    - Usa el tipo correcto de excepción según el contexto
    - Mensajes de error claros y orientados al usuario
    - Traduce los mensajes con `_()` cuando sea necesario
    - Captura excepciones específicas, no genéricas

    **Rendimiento**

    - Minimiza operaciones dentro de loops
    - Usa `mapped()`, `filtered()`, `sorted()` en lugar de loops cuando sea posible
    - Batch de operaciones: crea/actualiza múltiples registros a la vez
    - Usa `store=True` en campos computados si se usan frecuentemente en búsquedas