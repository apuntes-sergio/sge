---
title: Seguridad
description: Programación en Odoo. Seguridad
---

# Seguridad

La seguridad en Odoo es fundamental para proteger los datos y controlar qué usuarios pueden hacer qué acciones. El sistema de seguridad de Odoo es robusto y flexible, permitiendo desde permisos simples hasta reglas complejas basadas en el contenido de los registros.

En este capítulo aprenderás a configurar permisos de acceso a modelos, crear grupos de usuarios, definir reglas de registro (record rules) y gestionar la seguridad de campos específicos. Un sistema de permisos bien diseñado protege los datos sensibles mientras mantiene la usabilidad para los usuarios legítimos.

## Conceptos Fundamentales

El sistema de seguridad de Odoo trabaja en varios niveles:

### Niveles de Seguridad

| Nivel | Archivo | Granularidad | Descripción |
|-------|---------|--------------|-------------|
| **Modelo** | `ir.model.access.csv` | CRUD por modelo | Controla crear, leer, actualizar, eliminar |
| **Registro** | `ir.rule` (XML) | Por registro | Filtra qué registros ve cada usuario |
| **Campo** | `groups` en campo | Por campo | Oculta campos según grupos |
| **Menú** | `groups` en menuitem | Por menú | Controla visibilidad de menús |

### Grupos de Usuarios

Los grupos organizan usuarios con permisos similares. Odoo incluye grupos predefinidos y puedes crear los tuyos propios.

**Grupos estándar importantes**:

- `base.group_user`: Usuario interno (empleado)
- `base.group_system`: Administrador/Configuración
- `base.group_portal`: Usuario portal (externo)
- `base.group_public`: Usuario público (sin login)

## Permisos de Acceso a Modelos (ACL)

Los permisos de acceso controlan las operaciones CRUD (Create, Read, Update, Delete) que cada grupo puede realizar sobre un modelo.

### Archivo ir.model.access.csv

Este archivo CSV define los permisos básicos. Debe estar en la carpeta `security/` y declararse en el manifest.

**Estructura del archivo**:

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
```

!!!example "ir.model.access.csv básico"
    ```csv
    id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
    access_mi_producto_user,mi.producto.user,model_mi_producto,base.group_user,1,1,1,0
    access_mi_producto_manager,mi.producto.manager,model_mi_producto,base.group_system,1,1,1,1
    access_mi_categoria_user,mi.categoria.user,model_mi_categoria,base.group_user,1,0,0,0
    access_mi_categoria_manager,mi.categoria.manager,model_mi_categoria,base.group_system,1,1,1,1
    ```

**Columnas del CSV**:

| Columna | Descripción | Ejemplo |
|---------|-------------|---------|
| `id` | Identificador único | `access_mi_producto_user` |
| `name` | Nombre descriptivo | `mi.producto.user` |
| `model_id:id` | External ID del modelo | `model_mi_producto` |
| `group_id:id` | External ID del grupo | `base.group_user` |
| `perm_read` | Permiso de lectura (1/0) | `1` |
| `perm_write` | Permiso de escritura (1/0) | `1` |
| `perm_create` | Permiso de creación (1/0) | `1` |
| `perm_unlink` | Permiso de eliminación (1/0) | `0` |

### Permisos Sin Grupo (Acceso Total)

Si no especificas `group_id`, el permiso aplica a todos los usuarios:

!!!example "Acceso público"
    ```csv
    id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
    access_mi_producto_public,mi.producto.public,model_mi_producto,,1,0,0,0
    ```

### Modelo de Acceso Completo

!!!example "Archivo completo de permisos"
    ```csv
    id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
    
    # Modelo: mi.producto
    # Usuario normal: puede leer, crear, modificar (no eliminar)
    access_producto_user,producto.user,model_mi_producto,base.group_user,1,1,1,0
    # Manager: acceso total
    access_producto_manager,producto.manager,model_mi_producto,mi_modulo.group_producto_manager,1,1,1,1
    # Portal: solo lectura
    access_producto_portal,producto.portal,model_mi_producto,base.group_portal,1,0,0,0
    
    # Modelo: mi.categoria
    # Usuario: solo lectura
    access_categoria_user,categoria.user,model_mi_categoria,base.group_user,1,0,0,0
    # Manager: acceso total
    access_categoria_manager,categoria.manager,model_mi_categoria,base.group_system,1,1,1,1
    
    # Modelo: mi.configuracion
    # Solo administradores pueden acceder
    access_config_admin,config.admin,model_mi_configuracion,base.group_system,1,1,1,1
    ```

## Crear Grupos Personalizados

Puedes crear grupos específicos para tu módulo para organizar mejor los permisos.

### Definir Grupos en XML

!!!example "Crear grupos personalizados"
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <data>
            
            <!-- Categoría de aplicación -->
            <record id="module_category_inventario" model="ir.module.category">
                <field name="name">Inventario</field>
                <field name="description">Gestión de inventario y productos</field>
                <field name="sequence">10</field>
            </record>
            
            <!-- Grupo: Usuario -->
            <record id="group_inventario_user" model="res.groups">
                <field name="name">Usuario</field>
                <field name="category_id" ref="module_category_inventario"/>
                <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
            </record>
            
            <!-- Grupo: Manager -->
            <record id="group_inventario_manager" model="res.groups">
                <field name="name">Manager</field>
                <field name="category_id" ref="module_category_inventario"/>
                <field name="implied_ids" eval="[(4, ref('group_inventario_user'))]"/>
            </record>
            
            <!-- Grupo: Administrador -->
            <record id="group_inventario_admin" model="res.groups">
                <field name="name">Administrador</field>
                <field name="category_id" ref="module_category_inventario"/>
                <field name="implied_ids" eval="[(4, ref('group_inventario_manager'))]"/>
            </record>
            
        </data>
    </odoo>
    ```

**Campos importantes**:

- `name`: Nombre visible del grupo
- `category_id`: Categoría de la aplicación
- `implied_ids`: Grupos que se heredan automáticamente

### Grupos Implícitos (Herencia)

Los grupos pueden heredar permisos de otros grupos usando `implied_ids`:

!!!example "Jerarquía de grupos"
    ```xml
    <odoo>
        <!-- Usuario básico -->
        <record id="group_ventas_user" model="res.groups">
            <field name="name">Usuario Ventas</field>
            <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
        </record>
        
        <!-- Manager hereda de usuario -->
        <record id="group_ventas_manager" model="res.groups">
            <field name="name">Manager Ventas</field>
            <field name="implied_ids" eval="[(4, ref('group_ventas_user'))]"/>
        </record>
        
        <!-- Director hereda de manager -->
        <record id="group_ventas_director" model="res.groups">
            <field name="name">Director Ventas</field>
            <field name="implied_ids" eval="[
                (4, ref('group_ventas_manager')),
                (4, ref('account.group_account_user'))
            ]"/>
        </record>
    </odoo>
    ```

### Usar Grupos en CSV

Una vez definidos los grupos, úsalos en el archivo CSV:

!!!example "CSV con grupos personalizados"
    ```csv
    id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
    access_producto_user,producto.user,model_mi_producto,mi_modulo.group_inventario_user,1,1,1,0
    access_producto_manager,producto.manager,model_mi_producto,mi_modulo.group_inventario_manager,1,1,1,1
    access_producto_admin,producto.admin,model_mi_producto,mi_modulo.group_inventario_admin,1,1,1,1
    ```

## Reglas de Registro (Record Rules)

Las record rules filtran qué registros específicos puede ver o modificar cada usuario. Son mucho más granulares que los permisos de modelo.

### Definir Record Rules en XML

!!!example "Record rules básicas"
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <data>
            
            <!-- Regla: Usuarios solo ven sus propios productos -->
            <record id="producto_rule_user_own" model="ir.rule">
                <field name="name">Usuario ve sus productos</field>
                <field name="model_id" ref="model_mi_producto"/>
                <field name="domain_force">[('create_uid', '=', user.id)]</field>
                <field name="groups" eval="[(4, ref('base.group_user'))]"/>
                <field name="perm_read" eval="True"/>
                <field name="perm_write" eval="True"/>
                <field name="perm_create" eval="True"/>
                <field name="perm_unlink" eval="False"/>
            </record>
            
            <!-- Regla: Managers ven todos los productos -->
            <record id="producto_rule_manager_all" model="ir.rule">
                <field name="name">Manager ve todos los productos</field>
                <field name="model_id" ref="model_mi_producto"/>
                <field name="domain_force">[(1, '=', 1)]</field>
                <field name="groups" eval="[(4, ref('group_inventario_manager'))]"/>
                <field name="perm_read" eval="True"/>
                <field name="perm_write" eval="True"/>
                <field name="perm_create" eval="True"/>
                <field name="perm_unlink" eval="True"/>
            </record>
            
        </data>
    </odoo>
    ```

**Campos de ir.rule**:

| Campo | Descripción |
|-------|-------------|
| `name` | Nombre descriptivo de la regla |
| `model_id` | Modelo al que aplica |
| `domain_force` | Dominio que filtra los registros |
| `groups` | Grupos a los que aplica (vacío = global) |
| `perm_read` | Aplica a operaciones de lectura |
| `perm_write` | Aplica a operaciones de escritura |
| `perm_create` | Aplica a operaciones de creación |
| `perm_unlink` | Aplica a operaciones de eliminación |

### Dominios en Record Rules

Los dominios en record rules tienen acceso a variables especiales:

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `user` | Usuario actual | `[('user_id', '=', user.id)]` |
| `company_id` | Empresa del usuario | `[('company_id', '=', user.company_id.id)]` |
| `company_ids` | Empresas del usuario | `[('company_id', 'in', user.company_ids.ids)]` |
| `time` | Módulo time de Python | `[('fecha', '>=', time.strftime('%Y-01-01'))]` |

!!!example "Record rules con dominios complejos"
    ```xml
    <odoo>
        <!-- Solo productos de la empresa del usuario -->
        <record id="producto_rule_multicompany" model="ir.rule">
            <field name="name">Productos por empresa</field>
            <field name="model_id" ref="model_mi_producto"/>
            <field name="domain_force">['|',
                ('company_id', '=', False),
                ('company_id', 'in', company_ids)
            ]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        </record>
        
        <!-- Solo pedidos del departamento del usuario -->
        <record id="pedido_rule_departamento" model="ir.rule">
            <field name="name">Pedidos por departamento</field>
            <field name="model_id" ref="model_mi_pedido"/>
            <field name="domain_force">[
                ('departamento_id', '=', user.departamento_id.id)
            ]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        </record>
        
        <!-- Solo registros activos y del año actual -->
        <record id="documento_rule_activo_actual" model="ir.rule">
            <field name="name">Documentos activos del año</field>
            <field name="model_id" ref="model_mi_documento"/>
            <field name="domain_force">[
                ('activo', '=', True),
                ('fecha', '>=', time.strftime('%Y-01-01')),
                ('fecha', '<=', time.strftime('%Y-12-31'))
            ]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        </record>
    </odoo>
    ```

### Reglas Globales vs por Grupo

**Reglas por grupo** (con `groups` definido):
- Solo aplican a usuarios de esos grupos
- Se combinan con OR entre grupos
- Son aditivas (más permisivas)

**Reglas globales** (sin `groups`):
- Aplican a TODOS los usuarios
- Son restrictivas por defecto
- Útiles para restricciones generales

!!!example "Combinación de reglas"
    ```xml
    <odoo>
        <!-- Global: Nadie ve registros archivados -->
        <record id="producto_rule_global_no_archivado" model="ir.rule">
            <field name="name">No ver productos archivados</field>
            <field name="model_id" ref="model_mi_producto"/>
            <field name="domain_force">[('activo', '=', True)]</field>
            <!-- Sin groups = aplica a todos -->
        </record>
        
        <!-- Por grupo: Usuarios ven solo los suyos -->
        <record id="producto_rule_user_propios" model="ir.rule">
            <field name="name">Usuarios ven sus productos</field>
            <field name="model_id" ref="model_mi_producto"/>
            <field name="domain_force">[('create_uid', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        </record>
        
        <!-- Por grupo: Managers ven todos -->
        <record id="producto_rule_manager_todos" model="ir.rule">
            <field name="name">Managers ven todos</field>
            <field name="model_id" ref="model_mi_producto"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_inventario_manager'))]"/>
        </record>
        
        <!-- Resultado:
             - Usuarios normales: solo ven sus productos activos
             - Managers: ven todos los productos activos
             - Nadie ve productos archivados
        -->
    </odoo>
    ```

## Seguridad de Campos

Puedes restringir el acceso a campos específicos según grupos de usuarios.

### Atributo groups en Campos

!!!example "Campos restringidos por grupo"
    ```python
    from odoo import models, fields
    
    class Producto(models.Model):
        _name = 'mi.producto'
        
        nombre = fields.Char(string='Nombre', required=True)
        
        # Solo visible para usuarios internos
        precio_coste = fields.Float(
            string='Precio de Coste',
            groups='base.group_user'
        )
        
        # Solo visible para managers
        margen = fields.Float(
            string='Margen de Beneficio',
            compute='_compute_margen',
            groups='mi_modulo.group_inventario_manager'
        )
        
        # Solo visible para administradores
        notas_internas = fields.Text(
            string='Notas Internas',
            groups='base.group_system'
        )
        
        # Visible para múltiples grupos (separados por coma)
        precio_especial = fields.Float(
            string='Precio Especial',
            groups='sales_team.group_sale_manager,mi_modulo.group_inventario_manager'
        )
    ```

### Campo Invisible para Grupos

En las vistas, puedes usar `groups` para controlar visibilidad:

!!!example "Campos condicionados en vistas"
    ```xml
    <odoo>
        <record id="view_producto_form" model="ir.ui.view">
            <field name="name">mi.producto.form</field>
            <field name="model">mi.producto</field>
            <field name="arch" type="xml">
                <form>
                    <sheet>
                        <group>
                            <field name="nombre"/>
                            <field name="precio"/>
                            
                            <!-- Solo visible para usuarios internos -->
                            <field name="precio_coste" 
                                   groups="base.group_user"/>
                            
                            <!-- Solo visible para managers -->
                            <field name="margen" 
                                   groups="mi_modulo.group_inventario_manager"/>
                            
                            <!-- Solo visible para admin -->
                            <field name="notas_internas" 
                                   groups="base.group_system"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>
    </odoo>
    ```

## Seguridad de Menús

Controla qué usuarios pueden ver cada menú.

!!!example "Menús restringidos por grupo"
    ```xml
    <odoo>
        <!-- Menú visible para todos los usuarios internos -->
        <menuitem id="menu_productos"
                  name="Productos"
                  parent="menu_inventario_root"
                  action="action_producto_list"
                  groups="base.group_user"
                  sequence="10"/>
        
        <!-- Menú solo para managers -->
        <menuitem id="menu_configuracion"
                  name="Configuración"
                  parent="menu_inventario_root"
                  action="action_configuracion"
                  groups="mi_modulo.group_inventario_manager"
                  sequence="90"/>
        
        <!-- Menú solo para administradores -->
        <menuitem id="menu_configuracion_avanzada"
                  name="Configuración Avanzada"
                  parent="menu_configuracion"
                  action="action_configuracion_avanzada"
                  groups="base.group_system"
                  sequence="10"/>
        
        <!-- Menú para múltiples grupos -->
        <menuitem id="menu_reportes"
                  name="Reportes"
                  parent="menu_inventario_root"
                  action="action_reportes"
                  groups="mi_modulo.group_inventario_manager,account.group_account_manager"
                  sequence="50"/>
    </odoo>
    ```

## Verificar Permisos en Código Python

Puedes verificar permisos programáticamente en tus métodos.

### Verificar Pertenencia a Grupo

!!!example "Verificar grupos en Python"
    ```python
    from odoo import models, api
    from odoo.exceptions import AccessError
    
    class Producto(models.Model):
        _name = 'mi.producto'
        
        def action_publicar(self):
            """Solo managers pueden publicar productos"""
            if not self.env.user.has_group('mi_modulo.group_inventario_manager'):
                raise AccessError('Solo los managers pueden publicar productos')
            
            self.write({'estado': 'publicado'})
        
        def verificar_acceso_precio_coste(self):
            """Verificar si el usuario puede ver el precio de coste"""
            puede_ver = self.env.user.has_group('base.group_user')
            return puede_ver
        
        @api.model
        def create(self, vals):
            """Añadir lógica según permisos"""
            producto = super(Producto, self).create(vals)
            
            # Si es admin, enviar notificación especial
            if self.env.user.has_group('base.group_system'):
                producto._enviar_notificacion_admin()
            
            return producto
    ```

### Verificar Permisos de Modelo

!!!example "Verificar permisos CRUD"
    ```python
    from odoo import models
    from odoo.exceptions import AccessError
    
    class Pedido(models.Model):
        _name = 'mi.pedido'
        
        def verificar_puede_eliminar(self):
            """Verificar si el usuario puede eliminar"""
            try:
                self.check_access_rights('unlink')
                self.check_access_rule('unlink')
                return True
            except AccessError:
                return False
        
        def action_eliminar_seguro(self):
            """Eliminar solo si tiene permisos"""
            if not self.verificar_puede_eliminar():
                raise AccessError(
                    'No tiene permisos para eliminar este pedido'
                )
            
            self.unlink()
        
        @api.model
        def verificar_permisos_escritura(self):
            """Verificar permiso de escritura en el modelo"""
            try:
                self.check_access_rights('write')
                return True
            except AccessError:
                return False
    ```

### Bypass de Seguridad (sudo)

En ocasiones necesitas realizar operaciones como superusuario:

!!!example "Usar sudo()"
    ```python
    from odoo import models, api
    
    class Producto(models.Model):
        _name = 'mi.producto'
        
        def actualizar_stock_sistema(self):
            """Actualizar stock como sistema (bypass seguridad)"""
            # El usuario actual podría no tener permisos de escritura
            # pero esta operación la hace el sistema
            self.sudo().write({
                'stock': self.stock - 1,
                'fecha_ultimo_movimiento': fields.Datetime.now()
            })
        
        @api.model
        def tarea_programada_limpieza(self):
            """Tarea cron que necesita acceso completo"""
            # Buscar como superusuario (ve todos los registros)
            productos_obsoletos = self.sudo().search([
                ('activo', '=', False),
                ('create_date', '<', fields.Date.today() - timedelta(days=365))
            ])
            
            # Eliminar como superusuario
            productos_obsoletos.unlink()
    ```

!!! warning "Precaución con sudo()"
    - Usa `sudo()` solo cuando sea absolutamente necesario
    - Documenta claramente por qué lo necesitas
    - No lo uses para bypass reglas de negocio
    - Puede ser un riesgo de seguridad si se usa incorrectamente

## Estructura de Archivos de Seguridad

Organiza los archivos de seguridad de forma clara:

```
mi_modulo/
├── security/
│   ├── ir.model.access.csv          # Permisos de modelo
│   ├── security_groups.xml          # Definición de grupos
│   └── security_rules.xml           # Record rules
└── __manifest__.py
```

**En el manifest**:

```python
'data': [
    'security/security_groups.xml',      # Primero los grupos
    'security/ir.model.access.csv',      # Luego permisos de modelo
    'security/security_rules.xml',       # Después record rules
    'views/producto_views.xml',
    'views/menu.xml',
],
```

!!!tip  "Buenas Prácticas"

    **Permisos de modelo**

    - Define permisos para todos tus modelos (evita acceso no controlado)
    - Usa el principio de mínimo privilegio (dar solo los permisos necesarios)
    - Documenta la lógica de permisos en comentarios del CSV
    - Prueba los permisos con diferentes usuarios

    **Grupos**

    - Crea jerarquías claras (usuario → manager → admin)
    - Usa nombres descriptivos y consistentes
    - Agrupa permisos relacionados en el mismo grupo
    - Documenta qué hace cada grupo

    **Record rules**

    - Mantén las reglas simples y comprensibles
    - Usa reglas globales para restricciones generales
    - Combina reglas por grupo para permisos específicos
    - Prueba exhaustivamente las combinaciones de reglas

    **Seguridad de campos**

    - Restringe campos sensibles (costes, márgenes, datos personales)
    - Usa groups en el modelo, no solo en vistas
    - Documenta por qué ciertos campos están restringidos
    - Verifica que campos computados respetan restricciones

    **Testing**

    - Prueba con usuarios de diferentes grupos
    - Verifica que las restricciones funcionan
    - Comprueba casos límite (usuarios sin grupos, etc.)
    - Usa tests automatizados para seguridad crítica
