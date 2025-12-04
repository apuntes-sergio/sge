---
title: Herencia y Extensibilidad
description: Herencia y Extensibilidad
---

# Herencia y Extensibilidad

La herencia es uno de los mecanismos más potentes de Odoo. Permite extender y modificar módulos existentes sin alterar su código original, garantizando que las actualizaciones no rompan las personalizaciones. Este diseño modular es fundamental para la filosofía de Odoo y permite que múltiples módulos coexistan y se complementen.

En este capítulo aprenderás los tres tipos de herencia que Odoo proporciona, cómo extender modelos y vistas existentes, y cómo sobrescribir métodos manteniendo la funcionalidad original. Dominar la herencia es esencial para personalizar Odoo de forma profesional y mantenible.

## Tipos de Herencia en Odoo

Odoo proporciona tres mecanismos de herencia diferentes, cada uno con propósitos específicos:

### Tabla Comparativa

| Mecanismo | Crea Tabla | Modifica Original | Uso Principal |
|-----------|------------|-------------------|---------------|
| **Herencia de Clase** | No | Sí | Añadir campos/métodos a modelo existente |
| **Herencia por Prototipo** | Sí | No | Crear modelo nuevo basado en otro |
| **Herencia por Delegación** | Sí | No | Combinar múltiples modelos |

### Herencia de Clase (_inherit sin _name)

Es el tipo más común. Extiende un modelo existente añadiendo campos o métodos. El modelo original se modifica pero las vistas siguen funcionando.

**Características**:
- No crea nueva tabla en PostgreSQL
- Los nuevos campos se añaden a la tabla existente
- Las vistas del modelo original siguen funcionando
- El modelo mantiene su nombre original

**Cuándo usarla**: Cuando quieres añadir funcionalidad a un modelo existente sin crear uno nuevo.

!!!example "Herencia de clase"
    ```python
    from odoo import models, fields, api
    
    class ResPartner(models.Model):
        _inherit = 'res.partner'  # Modelo a extender
        # NO se define _name (usa el nombre del modelo heredado)
        
        # Añadir campos nuevos
        codigo_cliente = fields.Char(string='Código de Cliente')
        limite_credito = fields.Float(string='Límite de Crédito')
        es_proveedor_preferente = fields.Boolean(string='Proveedor Preferente')
        
        # Añadir método nuevo
        def verificar_credito(self):
            for partner in self:
                if partner.total_deuda > partner.limite_credito:
                    return False
            return True
        
        # Sobrescribir método existente
        @api.model
        def create(self, vals):
            # Añadir lógica antes de crear
            if 'codigo_cliente' not in vals:
                vals['codigo_cliente'] = self._generar_codigo_cliente()
            
            # Llamar al método original
            partner = super(ResPartner, self).create(vals)
            
            # Lógica después de crear
            partner.enviar_email_bienvenida()
            
            return partner
    ```

### Herencia por Prototipo (_inherit con _name)

Crea un nuevo modelo usando otro como plantilla. El modelo original NO se modifica y ambos coexisten.

**Características**:

- Crea nueva tabla en PostgreSQL
- El modelo original no se modifica
- Copia campos y métodos del modelo heredado
- Necesita definir sus propias vistas

**Cuándo usarla**: Cuando quieres crear un modelo nuevo que reutilice estructura de otro.

!!!example "Herencia por prototipo"
    ```python
    from odoo import models, fields
    
    class ResAlarm(models.Model):
        _name = 'res.alarm'  # Modelo base
        _description = 'Alarma Base'
        
        name = fields.Char(string='Nombre', required=True)
        duration = fields.Integer(string='Duración (minutos)')
        interval = fields.Selection([
            ('minutes', 'Minutos'),
            ('hours', 'Horas'),
            ('days', 'Días')
        ], string='Intervalo')
    
    class CalendarAlarm(models.Model):
        _name = 'calendar.alarm'  # Modelo NUEVO
        _inherit = 'res.alarm'    # Hereda de res.alarm
        _description = 'Alarma de Calendario'
        
        # Tiene todos los campos de res.alarm
        # Más campos propios
        tipo_notificacion = fields.Selection([
            ('email', 'Email'),
            ('popup', 'Popup'),
            ('sms', 'SMS')
        ], string='Tipo de Notificación')
        
        evento_id = fields.Many2one('calendar.event', string='Evento')
    ```

### Herencia por Delegación (_inherits)

Combina múltiples modelos en uno nuevo mediante delegación. Permite "heredar" de varios modelos a la vez (herencia múltiple).

**Características**:

- Crea nueva tabla en PostgreSQL
- Cada registro del modelo nuevo tiene registros relacionados en los modelos base
- Los campos de los modelos base son accesibles directamente
- Requiere campos Many2one a cada modelo base

**Cuándo usarla**: Cuando necesitas combinar funcionalidad de varios modelos en uno nuevo.

!!!example "Herencia por delegación"
    ```python
    from odoo import models, fields
    
    # Modelo base 1: Usuario
    class ResUsers(models.Model):
        _inherit = 'res.users'
        # Ya existe en Odoo con campos: name, email, login, etc.
    
    # Modelo base 2: Partner
    class ResPartner(models.Model):
        _inherit = 'res.partner'
        # Ya existe con campos: name, street, city, phone, etc.
    
    # Modelo nuevo que combina ambos
    class Empleado(models.Model):
        _name = 'hr.empleado'
        _inherits = {
            'res.users': 'user_id',      # Delega a res.users
            'res.partner': 'partner_id',  # Delega a res.partner
        }
        _description = 'Empleado'
        
        # Campos Many2one obligatorios (con ondelete='cascade')
        user_id = fields.Many2one(
            'res.users',
            required=True,
            ondelete='cascade',
            string='Usuario Relacionado'
        )
        partner_id = fields.Many2one(
            'res.partner',
            required=True,
            ondelete='cascade',
            string='Partner Relacionado'
        )
        
        # Campos propios del empleado
        numero_empleado = fields.Char(string='Número de Empleado')
        departamento_id = fields.Many2one('hr.departamento', string='Departamento')
        fecha_ingreso = fields.Date(string='Fecha de Ingreso')
        salario = fields.Float(string='Salario')
        
        # Ahora puedes acceder directamente a:
        # empleado.name (de res.users y res.partner)
        # empleado.email (de res.users)
        # empleado.phone (de res.partner)
        # empleado.street (de res.partner)
    ```

## Extender Modelos Existentes

La forma más común de personalizar Odoo es extendiendo modelos estándar para añadir campos y funcionalidad específica de tu negocio.

### Añadir Campos a Modelos Estándar

!!!example "Añadir campos a res.partner"
    ```python
    from odoo import models, fields, api
    from odoo.exceptions import ValidationError
    
    class ResPartnerExtend(models.Model):
        _inherit = 'res.partner'
        
        # Campos simples
        referencia_interna = fields.Char(
            string='Referencia Interna',
            help='Código interno para identificar al contacto'
        )
        
        tipo_cliente = fields.Selection([
            ('retail', 'Minorista'),
            ('wholesale', 'Mayorista'),
            ('distributor', 'Distribuidor')
        ], string='Tipo de Cliente', default='retail')
        
        descuento_general = fields.Float(
            string='Descuento General (%)',
            digits=(5, 2)
        )
        
        # Campo relacional
        vendedor_asignado_id = fields.Many2one(
            'res.users',
            string='Vendedor Asignado',
            domain=[('share', '=', False)]
        )
        
        # Campo computado
        total_pedidos = fields.Integer(
            string='Total Pedidos',
            compute='_compute_total_pedidos',
            store=True
        )
        
        @api.depends('sale_order_ids')
        def _compute_total_pedidos(self):
            for partner in self:
                partner.total_pedidos = len(partner.sale_order_ids)
        
        # Restricción
        @api.constrains('descuento_general')
        def _check_descuento(self):
            for partner in self:
                if partner.descuento_general < 0 or partner.descuento_general > 100:
                    raise ValidationError(
                        'El descuento debe estar entre 0 y 100%'
                    )
    ```

### Añadir Métodos a Modelos Existentes

!!!example "Añadir métodos a sale.order"
    ```python
    from odoo import models, api
    from odoo.exceptions import UserError
    
    class SaleOrderExtend(models.Model):
        _inherit = 'sale.order'
        
        def action_aplicar_descuento_cliente(self):
            """Aplica el descuento general del cliente a todas las líneas"""
            self.ensure_one()
            
            if not self.partner_id.descuento_general:
                raise UserError('El cliente no tiene descuento general configurado')
            
            descuento = self.partner_id.descuento_general
            
            for linea in self.order_line:
                precio_original = linea.product_id.list_price
                precio_con_descuento = precio_original * (1 - descuento / 100)
                linea.price_unit = precio_con_descuento
            
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'message': f'Descuento del {descuento}% aplicado a todas las líneas',
                    'type': 'success',
                    'sticky': False,
                }
            }
        
        def _validar_disponibilidad_productos(self):
            """Valida que hay stock suficiente para todos los productos"""
            productos_sin_stock = []
            
            for linea in self.order_line:
                if linea.product_id.type == 'product':
                    disponible = linea.product_id.qty_available
                    if disponible < linea.product_uom_qty:
                        productos_sin_stock.append(
                            f"{linea.product_id.name} (disponible: {disponible})"
                        )
            
            return productos_sin_stock
    ```

### Modificar Valores por Defecto

!!!example "Modificar defaults de modelos"
    ```python
    from odoo import models, fields, api
    
    class SaleOrderExtend(models.Model):
        _inherit = 'sale.order'
        
        # Sobrescribir campo existente con nuevo default
        validity_date = fields.Date(
            default=lambda self: fields.Date.today() + timedelta(days=30)
        )
        
        @api.model
        def default_get(self, fields_list):
            """Personalizar múltiples valores por defecto"""
            defaults = super(SaleOrderExtend, self).default_get(fields_list)
            
            # Establecer vendedor por defecto según el cliente
            if 'partner_id' in self.env.context:
                partner = self.env['res.partner'].browse(
                    self.env.context['partner_id']
                )
                if partner.vendedor_asignado_id:
                    defaults['user_id'] = partner.vendedor_asignado_id.id
            
            # Establecer almacén por defecto según usuario
            if self.env.user.almacen_predeterminado_id:
                defaults['warehouse_id'] = self.env.user.almacen_predeterminado_id.id
            
            return defaults
    ```

## Sobrescribir Métodos Existentes

Cuando heredas un modelo, puedes sobrescribir sus métodos para modificar su comportamiento. Es crucial usar `super()` para mantener la funcionalidad original.

### Patrón Básico de Sobrescritura

!!!example "Patrón de sobrescritura"
    ```python
    from odoo import models
    
    class MiModelo(models.Model):
        _inherit = 'modelo.existente'
        
        def metodo_a_sobrescribir(self):
            # 1. Lógica ANTES de la funcionalidad original
            print("Ejecutando lógica personalizada antes...")
            
            # 2. Llamar al método original con super()
            resultado = super(MiModelo, self).metodo_a_sobrescribir()
            
            # 3. Lógica DESPUÉS de la funcionalidad original
            print("Ejecutando lógica personalizada después...")
            
            # 4. Devolver resultado (modificado o no)
            return resultado
    ```

### Sobrescribir create()

!!!example "Sobrescribir create en product.product"
    ```python
    from odoo import models, api
    
    class ProductProductExtend(models.Model):
        _inherit = 'product.product'
        
        @api.model_create_multi
        def create(self, vals_list):
            # Lógica ANTES de crear
            for vals in vals_list:
                # Generar código de barras automático si no existe
                if 'barcode' not in vals or not vals['barcode']:
                    vals['barcode'] = self._generar_codigo_barras()
                
                # Establecer categoría por defecto según tipo
                if 'categ_id' not in vals:
                    if vals.get('type') == 'service':
                        categoria = self.env.ref('product.product_category_services')
                        vals['categ_id'] = categoria.id
            
            # Crear productos (llamar al original)
            productos = super(ProductProductExtend, self).create(vals_list)
            
            # Lógica DESPUÉS de crear
            for producto in productos:
                # Crear ubicación de almacén automática
                if producto.type == 'product':
                    self._crear_ubicacion_almacen(producto)
                
                # Notificar al departamento de compras
                if producto.seller_ids:
                    producto._notificar_nuevo_producto()
            
            return productos
        
        @api.model
        def _generar_codigo_barras(self):
            """Genera un código de barras único"""
            import random
            codigo = ''.join([str(random.randint(0, 9)) for _ in range(13)])
            # Verificar que no existe
            while self.search([('barcode', '=', codigo)]):
                codigo = ''.join([str(random.randint(0, 9)) for _ in range(13)])
            return codigo
    ```

### Sobrescribir write()

!!!example "Sobrescribir write en stock.picking"
    ```python
    from odoo import models
    from odoo.exceptions import UserError
    
    class StockPickingExtend(models.Model):
        _inherit = 'stock.picking'
        
        def write(self, vals):
            # Guardar estados anteriores si es necesario
            estados_anteriores = {picking.id: picking.state for picking in self}
            
            # Validaciones ANTES de escribir
            if 'state' in vals and vals['state'] == 'done':
                for picking in self:
                    # Validar que todas las líneas tienen lote/serie
                    if picking.picking_type_id.use_existing_lots:
                        sin_lote = picking.move_line_ids.filtered(
                            lambda ml: ml.product_id.tracking != 'none' 
                            and not ml.lot_id
                        )
                        if sin_lote:
                            raise UserError(
                                'Debe asignar número de lote/serie a todos los productos'
                            )
            
            # Escribir (llamar al original)
            resultado = super(StockPickingExtend, self).write(vals)
            
            # Acciones DESPUÉS de escribir
            if 'state' in vals:
                for picking in self:
                    estado_anterior = estados_anteriores[picking.id]
                    
                    # Si cambió a 'done', realizar acciones
                    if estado_anterior != 'done' and picking.state == 'done':
                        # Actualizar fecha de última entrega del proveedor
                        if picking.picking_type_id.code == 'incoming':
                            picking.partner_id.write({
                                'ultima_entrega': fields.Date.today()
                            })
                        
                        # Enviar notificación
                        picking._enviar_notificacion_completado()
            
            return resultado
    ```

### Sobrescribir unlink()

!!!example "Sobrescribir unlink en account.move"
    ```python
    from odoo import models
    from odoo.exceptions import UserError
    
    class AccountMoveExtend(models.Model):
        _inherit = 'account.move'
        
        def unlink(self):
            # Validaciones ANTES de eliminar
            for move in self:
                # No permitir eliminar facturas pagadas
                if move.payment_state in ['paid', 'in_payment']:
                    raise UserError(
                        f'No se puede eliminar la factura {move.name} '
                        f'porque tiene pagos asociados'
                    )
                
                # No permitir eliminar si tiene documentos relacionados
                if move.tipo_documento_relacionado == 'pedido' and move.pedido_id:
                    raise UserError(
                        f'No se puede eliminar la factura {move.name} '
                        f'porque está vinculada al pedido {move.pedido_id.name}'
                    )
            
            # Guardar información antes de eliminar (si es necesaria)
            info_facturas = []
            for move in self:
                info_facturas.append({
                    'name': move.name,
                    'partner_id': move.partner_id.id,
                    'amount_total': move.amount_total,
                })
            
            # Eliminar registros relacionados manualmente si es necesario
            # (solo si no se eliminan automáticamente por ondelete)
            self.mapped('mensaje_ids').unlink()
            
            # Llamar al unlink original
            resultado = super(AccountMoveExtend, self).unlink()
            
            # Registrar eliminación en log
            for info in info_facturas:
                self.env['mi.log.eliminaciones'].create({
                    'modelo': 'account.move',
                    'nombre_registro': info['name'],
                    'fecha_eliminacion': fields.Datetime.now(),
                    'usuario_id': self.env.user.id,
                })
            
            return resultado
    ```

### Sobrescribir Métodos de Negocio

!!!example "Sobrescribir action_confirm en sale.order"
    ```python
    from odoo import models
    from odoo.exceptions import UserError
    
    class SaleOrderExtend(models.Model):
        _inherit = 'sale.order'
        
        def action_confirm(self):
            """Añadir validaciones antes de confirmar pedido"""
            
            # Validaciones personalizadas
            for order in self:
                # Verificar límite de crédito del cliente
                if order.partner_id.limite_credito > 0:
                    deuda_actual = order.partner_id.total_deuda
                    if deuda_actual + order.amount_total > order.partner_id.limite_credito:
                        raise UserError(
                            f'El cliente {order.partner_id.name} excedería su límite '
                            f'de crédito ({order.partner_id.limite_credito}). '
                            f'Deuda actual: {deuda_actual}'
                        )
                
                # Verificar disponibilidad de productos
                productos_sin_stock = order._validar_disponibilidad_productos()
                if productos_sin_stock:
                    raise UserError(
                        'No hay stock suficiente para:\n' + 
                        '\n'.join(productos_sin_stock)
                    )
                
                # Aplicar descuento automático si supera monto mínimo
                if order.amount_total > 1000 and not order.descuento_aplicado:
                    order._aplicar_descuento_volumen()
            
            # Llamar al método original para confirmar
            resultado = super(SaleOrderExtend, self).action_confirm()
            
            # Acciones post-confirmación
            for order in self:
                # Notificar al departamento de almacén
                order._notificar_nuevo_pedido_almacen()
                
                # Crear tarea de seguimiento si es cliente nuevo
                if order.partner_id.sale_order_count == 1:
                    order._crear_tarea_seguimiento_cliente_nuevo()
            
            return resultado
    ```

## Extender Vistas Existentes

Además de extender modelos, puedes modificar vistas existentes para mostrar tus campos personalizados o cambiar la presentación.

### Herencia de Vistas

Para heredar una vista, usa el campo `inherit_id` referenciando el External ID de la vista padre.

!!!example "Estructura básica de herencia de vista"
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <record id="view_partner_form_inherit" model="ir.ui.view">
            <field name="name">res.partner.form.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <!-- Modificaciones aquí -->
            </field>
        </record>
    </odoo>
    ```

### Posiciones de Modificación

Odoo proporciona varios atributos `position` para controlar dónde se insertan los cambios:

| Position | Descripción |
|----------|-------------|
| `after` | Añade después del elemento |
| `before` | Añade antes del elemento |
| `inside` | Añade dentro del elemento (al final) |
| `replace` | Reemplaza el elemento completamente |
| `attributes` | Modifica solo los atributos del elemento |

### Añadir Campos a Formularios

!!!example "Añadir campos al formulario de contactos"
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <record id="view_partner_form_inherit_custom" model="ir.ui.view">
            <field name="name">res.partner.form.custom</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                
                <!-- Añadir después del campo 'phone' -->
                <field name="phone" position="after">
                    <field name="referencia_interna"/>
                </field>
                
                <!-- Añadir antes del campo 'email' -->
                <field name="email" position="before">
                    <field name="tipo_cliente"/>
                </field>
                
                <!-- Añadir dentro de un notebook existente -->
                <xpath expr="//notebook" position="inside">
                    <page string="Información Comercial">
                        <group>
                            <group>
                                <field name="descuento_general"/>
                                <field name="vendedor_asignado_id"/>
                            </group>
                            <group>
                                <field name="limite_credito"/>
                                <field name="total_pedidos"/>
                            </group>
                        </group>
                    </page>
                </xpath>
                
            </field>
        </record>
    </odoo>
    ```

### Usar XPath para Selecciones Complejas

XPath permite seleccionar elementos de forma más precisa cuando hay múltiples campos con el mismo nombre.

!!!example "Usar XPath en herencia de vistas"
    ```xml
    <odoo>
        <record id="view_sale_order_form_inherit" model="ir.ui.view">
            <field name="name">sale.order.form.inherit</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="arch" type="xml">
                
                <!-- Buscar campo específico en ubicación específica -->
                <xpath expr="//field[@name='partner_id']" position="after">
                    <field name="referencia_cliente"/>
                </xpath>
                
                <!-- Buscar dentro de un group específico -->
                <xpath expr="//group[@name='sale_info']" position="inside">
                    <field name="canal_venta"/>
                </xpath>
                
                <!-- Buscar por clase CSS -->
                <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                    <button type="object" name="action_ver_historial"
                            class="oe_stat_button" icon="fa-history">
                        <field name="historial_count" widget="statinfo" 
                               string="Historial"/>
                    </button>
                </xpath>
                
                <!-- Buscar el primer page de un notebook -->
                <xpath expr="//notebook/page[1]" position="after">
                    <page string="Información Adicional">
                        <group>
                            <field name="notas_internas"/>
                        </group>
                    </page>
                </xpath>
                
            </field>
        </record>
    </odoo>
    ```

### Reemplazar Elementos

!!!example "Reemplazar campos o elementos"
    ```xml
    <odoo>
        <record id="view_product_form_inherit" model="ir.ui.view">
            <field name="name">product.product.form.inherit</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="product.product_normal_form_view"/>
            <field name="arch" type="xml">
                
                <!-- Reemplazar un campo por otro -->
                <field name="list_price" position="replace">
                    <field name="list_price" widget="monetary" 
                           options="{'currency_field': 'currency_id'}"
                           attrs="{'readonly': [('puede_cambiar_precio', '=', False)]}"/>
                </field>
                
                <!-- Reemplazar un grupo completo -->
                <xpath expr="//group[@name='group_standard_price']" position="replace">
                    <group name="group_standard_price" string="Costes">
                        <field name="standard_price"/>
                        <field name="coste_adicional"/>
                        <field name="coste_total" readonly="1"/>
                    </group>
                </xpath>
                
            </field>
        </record>
    </odoo>
    ```

### Modificar Atributos

!!!example "Modificar atributos de elementos existentes"
    ```xml
    <odoo>
        <record id="view_partner_form_modify_attrs" model="ir.ui.view">
            <field name="name">res.partner.form.modify.attrs</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                
                <!-- Hacer campo obligatorio -->
                <field name="phone" position="attributes">
                    <attribute name="required">1</attribute>
                </field>
                
                <!-- Añadir clase CSS -->
                <field name="name" position="attributes">
                    <attribute name="class">oe_inline font-weight-bold</attribute>
                </field>
                
                <!-- Modificar string (etiqueta) -->
                <field name="street" position="attributes">
                    <attribute name="string">Dirección Completa</attribute>
                </field>
                
                <!-- Añadir placeholder -->
                <field name="email" position="attributes">
                    <attribute name="placeholder">correo@ejemplo.com</attribute>
                </field>
                
                <!-- Hacer invisible condicionalmente -->
                <field name="vat" position="attributes">
                    <attribute name="invisible">[('is_company', '=', False)]</attribute>
                </field>
                
                <!-- Añadir widget -->
                <field name="image_1920" position="attributes">
                    <attribute name="widget">image</attribute>
                    <attribute name="class">oe_avatar</attribute>
                </field>
                
            </field>
        </record>
    </odoo>
    ```

### Ocultar Elementos

!!!example "Ocultar campos o elementos"
    ```xml
    <odoo>
        <record id="view_sale_order_hide_fields" model="ir.ui.view">
            <field name="name">sale.order.form.hide.fields</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="arch" type="xml">
                
                <!-- Ocultar campo completamente -->
                <field name="fiscal_position_id" position="attributes">
                    <attribute name="invisible">1</attribute>
                </field>
                
                <!-- Ocultar grupo -->
                <xpath expr="//group[@name='sale_shipping']" position="attributes">
                    <attribute name="invisible">1</attribute>
                </xpath>
                
                <!-- Ocultar página de notebook -->
                <xpath expr="//page[@name='other_information']" position="attributes">
                    <attribute name="invisible">1</attribute>
                </xpath>
                
                <!-- Ocultar botón -->
                <xpath expr="//button[@name='action_cancel']" position="attributes">
                    <attribute name="invisible">1</attribute>
                </xpath>
                
            </field>
        </record>
    </odoo>
    ```

!!!tip "Buenas Prácticas"

    **Herencia de modelos**

    - Usa herencia de clase para extender, no crear modelos nuevos innecesarios
    - Documenta qué modelo estás heredando y por qué
    - Prefiere `super()` sobre llamadas directas a la clase padre
    - No sobrescribas métodos si solo necesitas añadir un campo
    - Mantén la coherencia con el modelo original

    **Sobrescritura de métodos**

    - Siempre llama a `super()` a menos que tengas una razón muy específica
    - Añade lógica antes o después del `super()`, no reemplaces completamente
    - Maneja los recordsets correctamente (itera si es necesario)
    - Devuelve el tipo correcto de dato
    - Documenta cambios significativos en el comportamiento

    **Herencia de vistas**

    - Usa `xpath` solo cuando sea necesario (campo simple es más claro)
    - No modifiques vistas de otros módulos custom innecesariamente
    - Mantén las modificaciones simples y específicas
    - Documenta modificaciones complejas con comentarios XML
    - Verifica que tus cambios no rompan otras vistas heredadas

    **Organización**

    - Un archivo de herencia por modelo heredado
    - Nombra descriptivamente: `res_partner_extend.py`, `res_partner_views.xml`
    - Agrupa herencias relacionadas en el mismo módulo
    - Declara dependencias correctamente en el manifest