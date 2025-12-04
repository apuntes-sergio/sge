---
title: Desarrollo de Vistas
description: Creación y personalización de vistas en Odoo
---

# Desarrollo de Vistas

Las vistas definen cómo se presenta la información al usuario en Odoo. Son la cara visible de tus modelos, permitiendo visualizar, crear y modificar datos de forma intuitiva. Dominar las vistas es esencial para crear aplicaciones empresariales con interfaces de usuario efectivas y agradables.

En este capítulo aprenderás a crear y personalizar los diferentes tipos de vistas que Odoo ofrece: formularios, listas, kanban, calendarios, gráficos y más. Cada tipo de vista tiene sus propias características y casos de uso específicos.

## Conceptos Fundamentales

Las vistas son archivos XML que describen cómo mostrar los datos de un modelo. Se almacenan en el modelo `ir.ui.view` de la base de datos y se cargan al instalar o actualizar el módulo.

### Estructura Básica de una Vista

!!!example "ejemplo"
    ```xml
    <record model="ir.ui.view" id="vista_form_mi_modelo">
        <field name="name">mi.modelo.form</field>
        <field name="model">mi.modelo</field>
        <field name="arch" type="xml">
            <!-- Contenido de la vista -->
            <form>
                <field name="nombre"/>
            </form>
        </field>
    </record>
    ```

**Elementos clave**:

- `<record>`: Define un registro en `ir.ui.view`
- `id`: Identificador único XML para referenciar la vista
- `name`: Nombre descriptivo de la vista
- `model`: Modelo al que pertenece la vista
- `arch`: Arquitectura XML que define la estructura

### Tipos de Vistas

Odoo proporciona varios tipos de vistas para diferentes propósitos:

| Tipo | Uso Principal | Etiqueta XML |
|------|---------------|--------------|
| **Form** | Crear/editar un registro | `<form>` |
| **List** | Mostrar múltiples registros en tabla | `<list>` |
| **Kanban** | Vista de tarjetas estilo tablero | `<kanban>` |
| **Search** | Filtros y búsquedas | `<search>` |
| **Calendar** | Vista de calendario | `<calendar>` |
| **Graph** | Gráficos y estadísticas | `<graph>` |
| **Pivot** | Tabla dinámica de análisis | `<pivot>` |
| **Activity** | Gestión de actividades | `<activity>` |

### Prioridad de Vistas

Si existen múltiples vistas del mismo tipo para un modelo, Odoo usa la de mayor prioridad (menor número = mayor prioridad):

!!!example "ejemplo"
    ```xml
    <field name="priority" eval="1"/>  <!-- Alta prioridad -->
    <field name="priority" eval="16"/> <!-- Baja prioridad (por defecto) -->
    ```

### Vistas por Defecto

Si no defines vistas para un modelo, Odoo genera automáticamente vistas básicas:

- Vista list con el campo `name` (o `_rec_name`)
- Vista form con todos los campos en un grupo simple

Aunque funcionales, estas vistas genéricas suelen ser inadecuadas para uso real.

## Vistas Form (Formulario)

Las vistas form permiten visualizar y editar un único registro. Son las más complejas y personalizables.

### Estructura Básica

!!!example "ejemplo"
    ```xml
    <record model="ir.ui.view" id="producto_form">
        <field name="name">producto.form</field>
        <field name="model">mi.producto</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="nombre"/>
                        <field name="precio"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    ```

### Elementos Estructurales

**sheet**

Contenedor principal que limita el ancho del formulario y proporciona el estilo visual estándar:

!!!example "ejemplo"
    ```xml
    <form>
        <sheet>
            <!-- Contenido del formulario -->
        </sheet>
    </form>
    ```

**header**

Zona superior del formulario para botones de acción y barra de estado:

!!!example "ejemplo"
    ```xml
    <form>
        <header>
            <button name="action_confirmar" string="Confirmar" type="object" class="oe_highlight"/>
            <button name="action_cancelar" string="Cancelar" type="object"/>
            <field name="estado" widget="statusbar" statusbar_visible="borrador,confirmado,enviado"/>
        </header>
        <sheet>
            <!-- Contenido -->
        </sheet>
    </form>
    ```

**group**

Organiza campos en columnas. Por defecto crea dos columnas:

!!!example "ejemplo"
    ```xml
    <group>
        <field name="nombre"/>      <!-- Columna 1 -->
        <field name="precio"/>      <!-- Columna 1 -->
        <field name="descripcion"/> <!-- Columna 1 -->
    </group>
    ```

Crear múltiples columnas explícitamente:

!!!example "ejemplo"
    ```xml
    <group>
        <group string="Información Básica">
            <field name="nombre"/>
            <field name="codigo"/>
        </group>
        <group string="Precios">
            <field name="precio"/>
            <field name="coste"/>
        </group>
    </group>
    ```

**notebook y page**

Pestañas para organizar mucha información:

!!!example "ejemplo"
    ```xml
    <notebook>
        <page string="Información General">
            <group>
                <field name="nombre"/>
            </group>
        </page>
        <page string="Descripción">
            <field name="descripcion"/>
        </page>
        <page string="Notas Internas">
            <field name="notas"/>
        </page>
    </notebook>
    ```

**separator**

Línea separadora con texto opcional:

!!!example "ejemplo"
    ```xml
    <separator string="Información Adicional"/>
    ```

### Botones en Formularios

Los botones ejecutan acciones cuando el usuario hace clic:

!!!example "ejemplo"
    ```xml
    <button name="metodo_python" 
            string="Ejecutar" 
            type="object" 
            class="oe_highlight"
            confirm="¿Estás seguro?"/>
    ```

**Atributos de botones**:

- `name`: Nombre del método a ejecutar o ID de la acción
- `string`: Texto del botón
- `type`: Tipo de acción
  - `object`: Llama a un método del modelo
  - `action`: Abre una acción (ventana, reporte, etc.)
- `class`: Clases CSS para estilo
  - `oe_highlight`: Botón principal (azul)
  - `btn-primary`: Botón importante
  - `btn-secondary`: Botón secundario
- `confirm`: Mensaje de confirmación antes de ejecutar
- `icon`: Icono Font Awesome (ej: `fa-check`)

**Botones con iconos**:

!!!example "ejemplo"
    ```xml
    <button name="action_archivar" type="object" icon="fa-archive"/>
    <button name="action_imprimir" type="object" icon="fa-print" string="Imprimir"/>
    ```

### Smart Buttons

Botones especiales que muestran información resumida y permiten navegar a registros relacionados:

!!!example "ejemplo"
    ```xml
    <div class="oe_button_box" name="button_box">
        <button type="object" 
                name="action_ver_tareas" 
                class="oe_stat_button" 
                icon="fa-tasks">
            <field name="tarea_count" widget="statinfo" string="Tareas"/>
        </button>
        <button type="object" 
                name="action_ver_facturas" 
                class="oe_stat_button" 
                icon="fa-file-text-o">
            <div class="o_field_widget o_stat_info">
                <span class="o_stat_value">
                    <field name="factura_count"/>
                </span>
                <span class="o_stat_text">Facturas</span>
            </div>
        </button>
    </div>
    ```

### Campos en Formularios

**Campos simples**:

!!!example "ejemplo"
    ```xml
    <field name="nombre"/>
    <field name="precio" widget="monetary"/>
    <field name="descripcion" placeholder="Describe el producto..."/>
    ```

**Campos con opciones**:

!!!example "ejemplo"
    ```xml
    <field name="imagen" widget="image" class="oe_avatar"/>
    <field name="cliente_id" options="{'no_create': True, 'no_open': True}"/>
    <field name="etiquetas" widget="many2many_tags" options="{'color_field': 'color'}"/>
    ```

**Campos invisibles**:

!!!example "ejemplo"
    ```xml
    <field name="campo_interno" invisible="1"/>
    ```

Útil para campos necesarios en lógica pero que no deben mostrarse.

### Visibilidad Condicional

Mostrar u ocultar elementos según valores de otros campos:

**Con invisible**:

!!!example "ejemplo"
    ```xml
    <field name="es_empresa" invisible="1"/>
    <field name="nombre_empresa" invisible="es_empresa == False"/>
    ```

**Con readonly**:

!!!example "ejemplo"
    ```xml
    <field name="precio" readonly="estado != 'borrador'"/>
    ```

**Con required**:

!!!example "ejemplo"
    ```xml
    <field name="nif" required="es_empresa == True"/>
    ```

**Múltiples condiciones**:

!!!example "ejemplo"
    ```xml
    <field name="campo" 
           invisible="estado in ['cancelado', 'finalizado']"
           readonly="estado == 'confirmado'"
           required="tipo == 'especial'"/>
    ```

**Operadores disponibles**:

- Comparación: `==`, `!=`, `<`, `>`, `<=`, `>=`
- Lógicos: `and`, `or`, `not`
- Pertenencia: `in`, `not in`

### Formularios Dinámicos

**Mostrar solo en modo edición o lectura**:

!!!example "ejemplo"
    ```xml
    <field name="campo_editable" class="oe_edit_only"/>
    <field name="campo_readonly" class="oe_read_only"/>
    ```

**Grupos condicionales**:

!!!example "ejemplo"
    ```xml
    <group invisible="tipo == 'servicio'">
        <field name="peso"/>
        <field name="volumen"/>
    </group>
    ```

### Ejemplo Completo de Formulario

!!!example "ejemplo"
    ```xml
    <record model="ir.ui.view" id="producto_form_completo">
        <field name="name">producto.form.completo</field>
        <field name="model">inventario.producto</field>
        <field name="arch" type="xml">
            <form>
                <header>
                    <button name="action_publicar" 
                            string="Publicar" 
                            type="object" 
                            class="oe_highlight"
                            invisible="estado != 'borrador'"/>
                    <button name="action_archivar" 
                            string="Archivar" 
                            type="object"
                            invisible="estado == 'archivado'"/>
                    <field name="estado" widget="statusbar" 
                           statusbar_visible="borrador,publicado"/>
                </header>
                
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button type="object" name="action_ver_ventas" 
                                class="oe_stat_button" icon="fa-shopping-cart">
                            <field name="venta_count" widget="statinfo" string="Ventas"/>
                        </button>
                    </div>
                    
                    <field name="imagen" widget="image" class="oe_avatar"/>
                    
                    <div class="oe_title">
                        <label for="nombre" class="oe_edit_only"/>
                        <h1><field name="nombre" placeholder="Nombre del producto"/></h1>
                    </div>
                    
                    <group>
                        <group string="Información General">
                            <field name="codigo"/>
                            <field name="categoria_id"/>
                            <field name="activo"/>
                        </group>
                        <group string="Precios e Inventario">
                            <field name="precio" widget="monetary"/>
                            <field name="coste" widget="monetary"/>
                            <field name="stock"/>
                        </group>
                    </group>
                    
                    <notebook>
                        <page string="Descripción">
                            <field name="descripcion" placeholder="Describe el producto..."/>
                        </page>
                        <page string="Ventas" invisible="venta_count == 0">
                            <field name="venta_ids">
                                <list>
                                    <field name="fecha"/>
                                    <field name="cliente_id"/>
                                    <field name="cantidad"/>
                                    <field name="total"/>
                                </list>
                            </field>
                        </page>
                        <page string="Notas Internas">
                            <field name="notas"/>
                        </page>
                    </notebook>
                </sheet>
                
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>
    ```

## Vistas List (Lista)

Las vistas list muestran múltiples registros en formato tabla. Son ideales para visualizar y comparar información.

### Estructura Básica

!!!example "ejemplo"
    ```xml
    <record model="ir.ui.view" id="producto_list">
        <field name="name">producto.list</field>
        <field name="model">inventario.producto</field>
        <field name="arch" type="xml">
            <list>
                <field name="codigo"/>
                <field name="nombre"/>
                <field name="precio"/>
                <field name="stock"/>
            </list>
        </field>
    </record>
    ```

### Decoraciones Condicionales

Aplicar colores a filas según condiciones:

!!!example "ejemplo"
    ```xml
    <list decoration-danger="stock == 0"
          decoration-warning="stock &lt; 10"
          decoration-success="stock &gt; 100">
        <field name="nombre"/>
        <field name="stock"/>
    </list>
    ```

**Decoraciones disponibles**:

- `decoration-bf`: Negrita
- `decoration-it`: Cursiva
- `decoration-danger`: Rojo (alertas)
- `decoration-warning`: Naranja (advertencias)
- `decoration-success`: Verde (éxito)
- `decoration-info`: Azul (información)
- `decoration-muted`: Gris (inactivo)
- `decoration-primary`: Morado (principal)

### Campos en Listas

**Campo invisible**:

Útil para usarlo en decoraciones sin mostrarlo:

!!!example "ejemplo"
    ```xml
    <list decoration-danger="activo == False">
        <field name="activo" invisible="1"/>
        <field name="nombre"/>
    </list>
    ```

**Campos opcionales**:

Permiten al usuario mostrar/ocultar columnas:

!!!example "ejemplo"
    ```xml
    <field name="descripcion" optional="hide"/>
    <field name="notas" optional="show"/>
    ```

**Campos con widgets**:

!!!example "ejemplo"
    ```xml
    <field name="progreso" widget="progressbar"/>
    <field name="prioridad" widget="priority"/>
    <field name="estado" widget="badge"/>
    <field name="etiquetas" widget="many2many_tags"/>
    ```

### Botones en Listas

!!!example "ejemplo"
    ```xml
    <list>
        <field name="nombre"/>
        <button name="action_publicar" 
                string="Publicar" 
                type="object" 
                icon="fa-check"
                invisible="estado != 'borrador'"/>
    </list>
    ```

### Sumatorios y Agregaciones

Mostrar totales al pie de las columnas:

!!!example "ejemplo"
    ```xml
    <list>
        <field name="nombre"/>
        <field name="cantidad" sum="Total"/>
        <field name="precio" avg="Promedio"/>
    </list>
    ```

**Funciones disponibles**: `sum`, `avg`, `min`, `max`

### Listas Editables

Permitir edición directa sin abrir formulario:

!!!example "ejemplo"
    ```xml
    <list editable="top">  <!-- o "bottom" -->
        <field name="nombre"/>
        <field name="cantidad"/>
    </list>
    ```

**Consideraciones**:

- Solo campos visibles son editables
- Útil para listas simples de configuración
- No recomendable para modelos complejos

### Orden por Defecto

!!!example "ejemplo"
    ```xml
    <list default_order="nombre, fecha desc">
        <field name="nombre"/>
        <field name="fecha"/>
    </list>
    ```

### Agrupación por Defecto

!!!example "ejemplo"
    ```xml
    <list default_group_by="categoria_id">
        <field name="categoria_id" invisible="1"/>
        <field name="nombre"/>
    </list>
    ```

### Ejemplo Completo de Lista

!!!example "ejemplo"
    ```xml
    <record model="ir.ui.view" id="producto_list_completo">
        <field name="name">producto.list.completo</field>
        <field name="model">inventario.producto</field>
        <field name="arch" type="xml">
            <list decoration-muted="activo == False"
                  decoration-warning="stock &lt; stock_minimo"
                  decoration-danger="stock == 0"
                  default_order="nombre">
                <field name="activo" invisible="1"/>
                <field name="stock_minimo" invisible="1"/>
                
                <field name="codigo"/>
                <field name="nombre"/>
                <field name="categoria_id"/>
                <field name="precio" widget="monetary"/>
                <field name="stock" sum="Total Stock"/>
                <field name="estado" widget="badge" 
                       decoration-success="estado == 'publicado'"
                       decoration-info="estado == 'borrador'"/>
                
                <button name="action_publicar" 
                        type="object" 
                        icon="fa-check" 
                        string="Publicar"
                        invisible="estado != 'borrador'"/>
            </list>
        </field>
    </record>
    ```

## Vistas Kanban

Las vistas kanban muestran registros como tarjetas, ideal para flujos de trabajo visuales estilo tablero.

### Estructura Básica

!!!example "ejemplo"
    ```xml
    <record model="ir.ui.view" id="tarea_kanban">
        <field name="name">tarea.kanban</field>
        <field name="model">proyecto.tarea</field>
        <field name="arch" type="xml">
            <kanban default_group_by="etapa_id">
                <field name="nombre"/>
                <field name="prioridad"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_card">
                            <div class="oe_kanban_content">
                                <h4><field name="nombre"/></h4>
                                <p><field name="descripcion"/></p>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>
    ```

### Elementos Kanban

**Campos fuera de templates**:

Deben declararse antes de `<templates>` para estar disponibles en el template:

!!!example "ejemplo"
    ```xml
    <kanban>
        <field name="nombre"/>
        <field name="prioridad"/>
        <field name="color"/>
        <templates>
            <!-- Usar campos aquí -->
        </templates>
    </kanban>
    ```

**Template kanban-box**:

Define cómo se ve cada tarjeta:

!!!example "ejemplo"
    ```xml
    <templates>
        <t t-name="kanban-box">
            <div class="oe_kanban_card" t-att-style="kanban_color(record.color.raw_value)">
                <div class="oe_kanban_content">
                    <!-- Contenido de la tarjeta -->
                </div>
            </div>
        </t>
    </templates>
    ```

### Acciones en Tarjetas

!!!example "ejemplo"
    ```xml
    <t t-name="kanban-box">
        <div class="oe_kanban_card">
            <!-- Abrir formulario al hacer clic en el nombre -->
            <h4><a type="open"><field name="nombre"/></a></h4>
            
            <!-- Editar inline -->
            <a type="edit">Editar</a>
            
            <!-- Eliminar -->
            <a type="delete">Eliminar</a>
        </div>
    </t>
    ```

### Kanban con Imágenes

!!!example "ejemplo"
    ```xml
    <t t-name="kanban-box">
        <div class="oe_kanban_card oe_kanban_global_click">
            <div class="o_kanban_image">
                <img t-att-src="kanban_image('producto.producto', 'imagen', record.id.raw_value)"/>
            </div>
            <div class="oe_kanban_details">
                <strong><field name="nombre"/></strong>
                <ul>
                    <li>Precio: <field name="precio"/></li>
                    <li>Stock: <field name="stock"/></li>
                </ul>
            </div>
        </div>
    </t>
    ```

### Colores en Kanban

!!!example "ejemplo"
    ```xml
    <kanban>
        <field name="color"/>
        <templates>
            <t t-name="kanban-box">
                <div class="oe_kanban_card" t-attf-class="oe_kanban_color_#{record.color.raw_value}">
                    <!-- Contenido -->
                </div>
            </t>
        </templates>
    </kanban>
    ```

### Menú Dropdown en Tarjetas

!!!example "ejemplo"
    ```xml
    <div class="oe_kanban_card">
        <div class="oe_dropdown_kanban dropdown">
            <a class="dropdown-toggle o-no-caret btn" data-bs-toggle="dropdown" href="#">
                <span class="fa fa-ellipsis-v"/>
            </a>
            <div class="dropdown-menu" role="menu">
                <a type="edit" role="menuitem" class="dropdown-item">Editar</a>
                <a type="delete" role="menuitem" class="dropdown-item">Eliminar</a>
            </div>
        </div>
        <!-- Resto del contenido -->
    </div>
    ```

## Vistas Search (Búsqueda)

Las vistas search definen filtros y opciones de búsqueda disponibles en otras vistas.

### Estructura Básica

!!!example "ejemplo"
    ```xml
    <record model="ir.ui.view" id="producto_search">
        <field name="name">producto.search</field>
        <field name="model">inventario.producto</field>
        <field name="arch" type="xml">
            <search>
                <field name="nombre"/>
                <field name="codigo"/>
                <field name="categoria_id"/>
            </search>
        </field>
    </record>
    ```

### Campos de Búsqueda

**Búsqueda simple**:

!!!example "ejemplo"
    ```xml
    <field name="nombre"/>
    ```

**Búsqueda con dominio personalizado**:

!!!example "ejemplo"
    ```xml
    <field name="nombre_o_codigo" 
           string="Nombre o Código"
           filter_domain="['|', ('nombre', 'ilike', self), ('codigo', 'ilike', self)]"/>
    ```

### Filtros Predefinidos

!!!example "ejemplo"
    ```xml
    <search>
        <field name="nombre"/>
        
        <!-- Filtros booleanos -->
        <filter name="activos" string="Activos" domain="[('activo', '=', True)]"/>
        <filter name="sin_stock" string="Sin Stock" domain="[('stock', '=', 0)]"/>
        
        <!-- Separador visual -->
        <separator/>
        
        <!-- Filtros con dominio complejo -->
        <filter name="precio_alto" string="Precio Alto" domain="[('precio', '&gt;', 100)]"/>
        
        <!-- Agrupación -->
        <group expand="0" string="Agrupar por">
            <filter name="group_categoria" string="Categoría" context="{'group_by': 'categoria_id'}"/>
            <filter name="group_estado" string="Estado" context="{'group_by': 'estado'}"/>
        </group>
    </search>
    ```

### Filtros por Fecha

!!!example "ejemplo"
    ```xml
    <filter name="mes_actual" string="Este Mes"
            domain="[('fecha', '&gt;=', (context_today() - relativedelta(months=1)).strftime('%Y-%m-01')),
                     ('fecha', '&lt;', (context_today() + relativedelta(months=1)).strftime('%Y-%m-01'))]"/>

    <filter name="este_año" string="Este Año"
            domain="[('fecha', '&gt;=', context_today().strftime('%Y-01-01'))]"/>
    ```

### Agrupación por Fechas

!!!example "ejemplo"
    ```xml
    <group expand="0" string="Agrupar por">
        <filter name="group_fecha_mes" string="Mes" context="{'group_by': 'fecha:month'}"/>
        <filter name="group_fecha_año" string="Año" context="{'group_by': 'fecha:year'}"/>
    </group>
    ```

## Vistas Calendar

Muestran registros en un calendario basándose en campos de fecha.

!!!example "ejemplo"
    ```xml
    <record model="ir.ui.view" id="evento_calendar">
        <field name="name">evento.calendar</field>
        <field name="model">agenda.evento</field>
        <field name="arch" type="xml">
            <calendar date_start="fecha_inicio" 
                      date_stop="fecha_fin"
                      color="usuario_id"
                      mode="month">
                <field name="nombre"/>
                <field name="descripcion"/>
            </calendar>
        </field>
    </record>
    ```

**Atributos principales**:

- `date_start`: Campo de fecha/hora de inicio (obligatorio)
- `date_stop`: Campo de fecha/hora de fin (opcional)
- `date_delay`: Duración en horas (alternativa a date_stop)
- `color`: Campo para colorear eventos
- `mode`: Vista inicial (day, week, month)

## Vistas Graph

Muestran datos agregados en gráficos.

!!!example "ejemplo"
    ```xml
    <record model="ir.ui.view" id="venta_graph">
        <field name="name">venta.graph</field>
        <field name="model">venta.venta</field>
        <field name="arch" type="xml">
            <graph type="bar" stacked="True">
                <field name="fecha" type="row" interval="month"/>
                <field name="categoria_id" type="col"/>
                <field name="total" type="measure"/>
            </graph>
        </field>
    </record>
    ```

**Tipos de gráfico**:

- `bar`: Barras
- `line`: Líneas
- `pie`: Circular

**Tipos de campo**:

- `row`: Eje X o agrupación principal
- `col`: Series o sub-agrupación
- `measure`: Valores a medir (sum, avg, min, max, count)

## Widgets Especiales

Los widgets cambian cómo se muestra un campo sin cambiar su tipo.

### Widgets para Integer/Float

!!!example "ejemplo"
    ```xml
    <field name="porcentaje" widget="percentpie"/>
    <field name="progreso" widget="progressbar"/>
    <field name="horas" widget="float_time"/>
    <field name="cantidad" widget="integer"/>
    ```

### Widgets para Char/Text

!!!example "ejemplo"
    ```xml
    <field name="email" widget="email"/>
    <field name="url" widget="url"/>
    <field name="telefono" widget="phone"/>
    <field name="contenido" widget="html"/>
    ```

### Widgets para Many2one

!!!example "ejemplo"
    ```xml
    <field name="usuario_id" widget="many2one_avatar"/>
    <field name="pais_id" widget="many2one" options="{'no_create': True, 'no_open': True}"/>
    ```

### Widgets para Many2many

!!!example "ejemplo"
    ```xml
    <field name="etiquetas" widget="many2many_tags" options="{'color_field': 'color'}"/>
    <field name="habilidades" widget="many2many_checkboxes"/>
    <field name="adjuntos" widget="many2many_binary"/>
    ```

### Widgets para Boolean

!!!example "ejemplo"
    ```xml
    <field name="activo" widget="boolean_toggle"/>
    ```

### Widgets para Image

!!!example "ejemplo"
    ```xml
    <field name="foto" widget="image" options="{'size': [100, 100]}"/>
    ```

!!!tip "Buenas Prácticas"

    **Organización de vistas**

    - Un archivo XML por modelo
    - Nombrar archivos descriptivamente: `producto_views.xml`
    - Orden: search, list, form, kanban, otras
    - Comentar vistas complejas

    **Rendimiento**

    - No incluir campos no necesarios en list
    - Usar `invisible="1"` en lugar de ocultar con CSS
    - Limitar campos computados en vistas list
    - Usar paginación por defecto en listas largas

    **Usabilidad**

    - Campos importantes primero en formularios
    - Agrupar campos relacionados
    - Usar pestañas para evitar formularios muy largos
    - Proporcionar valores por defecto sensatos
    - Mensajes de ayuda (`help`) en campos complejos

    **Diseño**

    - Mantener consistencia con vistas estándar de Odoo
    - Usar decoraciones con moderación
    - Smart buttons para navegación entre registros
    - Botones de acción en header del formulario