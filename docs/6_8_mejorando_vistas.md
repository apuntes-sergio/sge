---
title: Mejorando vistas
description: Personalización avanzada de vistas lista en Odoo
---

# Mejorando Vistas

Hasta ahora hemos trabajado principalmente con vistas de formulario, dejando que Odoo genere automáticamente las vistas de lista (tree) por defecto. En este apartado aprenderemos a personalizar estas vistas para mejorar la experiencia de usuario, añadiendo características como colores condicionales, campos calculados visibles, sumatorios y otras funcionalidades que harán nuestras listas más informativas y útiles.

## Vistas Lista en Odoo

La vista lista (anteriormente llamada *tree*) es la vista más común en Odoo, mostrando múltiples registros en formato tabular. Por defecto, cuando no definimos una vista lista personalizada, Odoo genera una automáticamente mostrando el campo `name` y algunos campos básicos del modelo.

En nuestro módulo de gestión de tareas, hemos creado varios modelos pero apenas hemos personalizado sus vistas lista. Por ejemplo, si revisamos el código hasta ahora, encontraremos que la mayoría de vistas que hemos definido son de tipo formulario, y las listas son las que Odoo genera por defecto.

Estas vistas predeterminadas funcionan, pero son muy básicas y no aprovechan todas las posibilidades que Odoo ofrece para hacer las listas más informativas y funcionales.

## Situación Actual de Nuestras Vistas

Actualmente, nuestro módulo muestra varias vistas lista generadas automáticamente. Por ejemplo:

- **Sprints**: Muestra solo el nombre por defecto
- **Tareas**: Tiene una vista lista básica que definimos al principio
- **Tecnologías**: Vista automática con nombre y descripción

Si queremos mejorar la experiencia de usuario y hacer que estas listas sean más útiles, necesitamos personalizarlas. Vamos a empezar con el modelo Sprint, que es perfecto para demostrar las diferentes técnicas de personalización.

## Crear una Vista Lista Personalizada

Para personalizar una vista lista, debemos definir un registro en el modelo `ir.ui.view` igual que hacemos con las vistas de formulario. La estructura básica es muy similar, cambiando el tipo de vista de `form` a `list`.

Vamos a crear una vista lista completa para el modelo Sprint. En tu archivo `views.xml`:

!!! example "views.xml"

    ```xml
    <record model="ir.ui.view" id="sprints_list">
        <field name="name">gestion_tareas_sergio.sprints_sergio.list</field>
        <field name="model">gestion_tareas_sergio.sprints_sergio</field>
        <field name="arch" type="xml">
            <list>
                <field name="name"/>
                <field name="descripcion"/>
                <field name="duracion"/>
                <field name="fecha_ini"/>
                <field name="fecha_fin"/>
            </list>
        </field>
    </record>
    ```

**Elementos de la definición**:

- `id`: Identificador único de la vista, en este caso `sprints_list`
- `name`: Nombre descriptivo de la vista
- `model`: El modelo al que aplica esta vista
- `arch`: La arquitectura XML que define la estructura de la vista
- `<list>`: Etiqueta principal que indica que es una vista de tipo lista
- `<field>`: Cada campo que queremos mostrar como columna

Una vez definida y actualizado el módulo, veremos que la lista de sprints muestra todas las columnas especificadas. Si tienes sprints creados con fechas y duraciones, verás cómo los campos computados como `fecha_fin` se muestran correctamente calculados.


## Decoraciones Condicionales

Una de las características más útiles de las vistas lista es la posibilidad de aplicar colores y estilos condicionales a las filas según los valores de sus campos. Esto se hace mediante el atributo `decoration` en la etiqueta `<list>`.

### Tipos de Decoraciones Disponibles

Odoo proporciona varios tipos de decoraciones predefinidas:

| Decoración | Color/Estilo | Uso típico |
|-----------|--------------|------------|
| `decoration-bf` | Negrita | Resaltar registros importantes |
| `decoration-it` | Cursiva | Registros secundarios o notas |
| `decoration-danger` | Rojo | Errores, alertas críticas |
| `decoration-warning` | Naranja/Amarillo | Advertencias, atención requerida |
| `decoration-success` | Verde | Éxito, completado |
| `decoration-info` | Azul | Información, en proceso |
| `decoration-muted` | Gris | Inactivo, archivado |
| `decoration-primary` | Azul oscuro | Principal, destacado |

### Aplicar Decoración Simple

Por ejemplo, para resaltar en amarillo los sprints con exactamente 15 días de duración:

!!! example "views.xml - Decorador simple por campo duración"

    ```xml
    <list decoration-warning="duracion == 15">
        <field name="name"/>
        <field name="descripcion"/>
        <field name="duracion"/>
        <field name="fecha_ini"/>
        <field name="fecha_fin"/>
    </list>
    ```

La condición `duracion == 15` se evalúa para cada registro. Si es verdadera, se aplica el estilo de advertencia (color amarillo/naranja).

### Múltiples Decoraciones

Podemos aplicar varias decoraciones a la vez. Por ejemplo, resaltar en rojo sprints muy cortos y en verde los de duración estándar:

!!! example "views.xml - Decoración multiple dependiendo de valor de campos duración"

    ```xml
    <list decoration-danger="duracion &lt; 7" 
        decoration-success="duracion == 14">
        <field name="name"/>
        <field name="descripcion"/>
        <field name="duracion"/>
        <field name="fecha_ini"/>
        <field name="fecha_fin"/>
    </list>
    ```

!!! note "Nota importante sobre el orden" 

    Si un registro cumple múltiples condiciones, se aplicará la última decoración que coincida. En el ejemplo anterior, un sprint de 14 días se mostrará en verde aunque también cumpla otras condiciones.

    Pero además por encima de esta regla tenemos que hay un orden de prioridad interno que tiene Odoo para las decoraciones CSS.

    El motor de Odoo aplica las clases CSS en un orden específico. La prioridad (de mayor a mayor importancia) es la siguiente:

    - danger (Rojo) - Máxima prioridad.
    - warning (Naranja)
    - success (Verde)
    - info (Azul)
    - muted (Gris) - Menor prioridad.

Si queremos que una decoración se cumpla siempre entonces la condición será `="1"` (igual a 1 con comillas)

## Uso de Operadores en Condiciones

Al escribir condiciones dentro de atributos XML, debemos tener cuidado con ciertos caracteres especiales que tienen significado en XML.

### Operadores de Comparación

| Operador Python | En XML | Significado |
|----------------|---------|-------------|
| `<` | `&lt;` | Menor que |
| `<=` | `&lt;=` | Menor o igual |
| `>` | `&gt;` | Mayor que |
| `>=` | `&gt;=` | Mayor o igual |
| `==` | `==` | Igual |
| `!=` | `!=` | Diferente |

### Ejemplo con Operadores

Para resaltar sprints con duración menor a 15 días:

!!! example "views.xml"
    ```xml
    <list decoration-warning="duracion &lt; 15">
        <!-- campos -->
    </list>
    ```

### Operadores Lógicos

También podemos combinar condiciones con operadores lógicos:

!!! example "views.xml"
    ```xml
    <!-- Sprints cortos que ya han comenzado -->
    <list decoration-warning="duracion &lt; 10 and fecha_ini &lt;= current_date">
        <!-- campos -->
    </list>
    ```

Operadores lógicos disponibles:

- `and`: Ambas condiciones deben cumplirse
- `or`: Al menos una condición debe cumplirse
- `not`: Niega la condición

## Campo Computado para Estado Activo

Para hacer decoraciones más útiles, podemos crear un campo computado que determine si un sprint está actualmente en curso. Esto nos permite resaltar visualmente los sprints activos.

### Definir el Campo en el Modelo

En `models.py`, añadimos un campo booleano computado:

!!! example "models.py - Añadiendo campo `activo`"

    ```python
    from datetime import date

    class sprints_sergio(models.Model):
        _name = 'gestion_tareas_sergio.sprints_sergio'
        # ... campos existentes ...
        
        activo = fields.Boolean(
            compute='_compute_activo',
            string='En Curso',
            help='Indica si el sprint está actualmente en curso'
        )
        
        @api.depends('fecha_ini', 'fecha_fin')
        def _compute_activo(self):
            hoy = date.today()
            for sprint in self:
                if sprint.fecha_ini and sprint.fecha_fin:
                    # Sprint activo si hoy está entre fecha inicio y fin
                    fecha_ini_date = sprint.fecha_ini.date() if hasattr(sprint.fecha_ini, 'date') else sprint.fecha_ini
                    fecha_fin_date = sprint.fecha_fin.date() if hasattr(sprint.fecha_fin, 'date') else sprint.fecha_fin
                    sprint.activo = fecha_ini_date <= hoy <= fecha_fin_date
                else:
                    sprint.activo = False
    ```

**Explicación del código**:

- `activo`: Campo booleano que se calcula automáticamente
- `@api.depends('fecha_ini', 'fecha_fin')`: Se recalcula cuando cambian estas fechas
- `hoy = date.today()`: Obtiene la fecha actual
- La condición `fecha_ini <= hoy <= fecha_fin` verifica si hoy está dentro del periodo del sprint
- Manejamos conversión de `Datetime` a `date` si es necesario

### Usar el Campo en la Vista

Ahora podemos usar este campo para aplicar decoraciones, incluso si no lo mostramos como columna:

!!! example "views.xml"

    ```xml
    <list decoration-info="activo == True">
        <field name="name"/>
        <field name="duracion"/>
        <field name="fecha_ini"/>
        <field name="fecha_fin"/>
        <field name="activo" invisible="1"/>
    </list>
    ```

**Aspectos importantes**:

- `decoration-info="activo == True"`: Aplica color azul a sprints activos
- `<field name="activo" invisible="1"/>`: Incluimos el campo en la vista pero lo ocultamos
- Aunque invisible, el campo se carga y puede usarse en condiciones

Con esto, los sprints en curso se resaltarán en azul, facilitando su identificación visual.

## Combinar Múltiples Decoraciones

Podemos aplicar diferentes decoraciones según el estado del sprint para crear una vista muy informativa:

!!! example "views.xml"

    ```xml
    <list decoration-info="activo == True"
        decoration-muted="fecha_fin &lt; current_date"
        decoration-success="duracion == 14">
        <field name="name"/>
        <field name="descripcion"/>
        <field name="duracion"/>
        <field name="fecha_ini"/>
        <field name="fecha_fin"/>
        <field name="activo" invisible="1"/>
    </list>
    ```

Esta configuración:

- **Azul**: Sprints en curso (activos)
- **Gris**: Sprints finalizados
- **Verde**: Sprints con duración estándar de 14 días

!!! note "Recuerda"
    El orden de evaluación es importante. Si un sprint cumple múltiples condiciones, se aplicará la última que coincida.

## Vistas Lista Editables

Odoo permite hacer las vistas lista editables, permitiendo modificar registros directamente desde la lista sin abrir el formulario. Esto se hace añadiendo el atributo `editable` a la etiqueta `<list>`.

### Sintaxis

```xml
<list editable="top">
    <!-- campos -->
</list>
```

o

```xml
<list editable="bottom">
    <!-- campos -->
</list>
```

**Diferencia entre top y bottom**:

- `editable="top"`: Nuevos registros aparecen al principio de la lista
- `editable="bottom"`: Nuevos registros aparecen al final de la lista

### Ejemplo con Sprints

!!! example "views.xml"

    ```xml
    <list editable="top">
        <field name="name"/>
        <field name="duracion"/>
        <field name="fecha_ini"/>
    </list>
    ```

### Consideraciones sobre Vistas Editables

**Ventajas**:

- Edición rápida sin abrir formularios
- Útil para listas de configuración o catálogos simples
- Mejora la productividad en tareas repetitivas

**Desventajas**:

- Solo se pueden editar campos visibles en la lista
- No permite usar widgets complejos del formulario
- Pierde funcionalidad de validaciones complejas
- No recomendable para modelos con muchos campos o lógica compleja

**Recomendación**: Usa vistas editables solo para modelos muy simples o listas de configuración. Para modelos complejos como tareas o sprints, es mejor mantener la edición mediante formularios.

## Sumatorios en Columnas

Otra funcionalidad muy útil es mostrar sumatorios al pie de las columnas numéricas. Esto es especialmente práctico para campos como duraciones, presupuestos, cantidades, etc.

### Añadir un Sumatorio

Para mostrar el total de días programados en todos los sprints:

!!! example "views.xml"

    ```xml
    <list>
        <field name="name"/>
        <field name="duracion" sum="Total días"/>
        <field name="fecha_ini"/>
        <field name="fecha_fin"/>
    </list>
    ```

El atributo `sum="Total días"` hace que:

- Se sume automáticamente el valor de todos los registros visibles
- Se muestre el total al pie de la columna con la etiqueta "Total días"

### Múltiples Sumatorios

Podemos añadir sumatorios a varias columnas:

!!! example "views.xml"

    ```xml
    <list>
        <field name="name"/>
        <field name="duracion" sum="Total días"/>
        <field name="numero_tareas" sum="Total tareas"/>
        <field name="fecha_ini"/>
    </list>
    ```

### Otros Atributos de Agregación

Además de `sum`, Odoo soporta otros tipos de agregación:

- `sum`: Suma de valores
- `avg`: Promedio
- `min`: Valor mínimo
- `max`: Valor máximo


!!! example "views.xml - Ejemplo con promedio"

    ```xml
    <field name="duracion" avg="Promedio"/>
    ```

!!! note "Nota"
    Los sumatorios solo aplican a los registros visibles en la lista según los filtros activos. Si tienes 100 sprints pero solo muestras 10 filtrados, el sumatorio será de esos 10.


Para tus apuntes de clase, puedes definir un **Widget** en Odoo de la siguiente manera:

## Añadienteo Widgets a nuestras vistas

Un **Widget** es un componente de la interfaz de usuario que define **cómo se visualiza y cómo se interactúa** con un campo específico en la pantalla.

Por defecto, Odoo renderiza los campos según su tipo (un `Integer` como un número, un `Char` como texto plano). Sin embargo, al usar un widget, transformamos esa representación técnica en una herramienta visual funcional.

### Widgets en la Vista Listado (`tree` / `list`)

En las listas, los widgets suelen ser **pasivos o de visualización rápida**. Su objetivo es facilitar la lectura de datos sin tener que entrar en el registro.

* **Función:** Cambian la estética o añaden un icono.
* **Ejemplos comunes:**

| Widget | Tipo de campo | Descripción breve | Ejemplo Visual (Representación) |
| --- | --- | --- | --- |
| **`badge`** | Selection / Char | Encapsula el texto en un óvalo coloreado. | `[ Borrador ]` (Etiqueta azul) |
| **`many2many_tags`** | Many2many | Muestra relaciones como etiquetas de colores. | `( Urgente ) ( Odoo ) ( Dev )` |
| **`priority`** | Integer / Selection | Transforma el valor en estrellas doradas. | `⭐ ⭐ ⭐ ✩ ✩` |
| **`progressbar`** | Integer / Float | Muestra una barra de carga rellena. | `[██████░░░░] 60%` |
| **`boolean_toggle`** | Boolean | Sustituye el check por un interruptor. | `( ○━━━━) / (━━━━● )` |
| **`boolean_favorite`** | Boolean | Muestra una estrella de "favorito". | `⭐ (Rellena) / ✩ (Vacía)` |
| **`handle`** | Integer | Icono para arrastrar y reordenar filas. | `☰` (Icono de tres rayas) |
| **`image`** | Binary | Muestra la miniatura de la imagen subida. | `[ 📷 Miniatura ]` |
| **`url`** | Char | Convierte el texto en un enlace azul. | `www.odoo.com` (Subrayado) |
| **`email`** | Char | Enlace directo para enviar correo. | `📨 usuario@mail.com` |
| **`phone`** | Char | Enlace para realizar llamadas (Tel/Skype). | `📞 +34 600...` |
| **`remaining_days`** | Date / Datetime | Muestra cuánto falta para una fecha. | *"En 5 días"* o *"Hace 2 días"* |


!!!example "views.xml"
    ```xml
    <list decoration-info="activo == True"
        decoration-muted="fecha_fin &lt; current_date"
        decoration-success="duracion == 14">
        <field name="name"/>
        <field name="descripcion"/>
        <field name="duracion"/>
        <field name="fecha_ini"/>
        <field name="fecha_fin"/>
        <field name="activo" widget="boolean_toggle"/>
    </list>
    ```


En los módulos que vienen con Odoo encontrarás excelentes ejemplos de vistas lista avanzadas, especialmente en módulos como Ventas, Compras e Inventario.

!!!tip "Para saber los widgets que tenemos en nuestro odoo"

    ```bash
    docker exec -it odoo_dev_dam ls /usr/lib/python3/dist-packages/odoo/addons/web/static/src/views/fields
    ```


### Widgets en la Vista Formulario (`form`)

En los formularios, los widgets son mucho más **interactivos**. No solo muestran el dato, sino que cambian la forma en que el usuario introduce la información.

**Ejemplos comunes:**

- `widget="status_bar"`: Crea la flecha de flujo de trabajo en la parte superior del formulario.
- `widget="image"`: Permite previsualizar, subir o borrar una foto.
- `widget="selection"`: Convierte un campo relacional en un menú desplegable simple (quitando las opciones de "Crear y editar").
- `widget="url"`: Convierte un texto en un enlace clickeable.

!!! note "Diferencia **decoraciones** y **widgets**"

    Mientras que las **decoraciones** (`decoration-X`) cambian el color de la fila o campo según condiciones, el **widget** cambia la estructura y el comportamiento del control web.


## Vista Lista Completa y Mejorada

Juntando todas las técnicas vistas, aquí tienes una vista lista completa y funcional para Sprints:


!!! example "views.xml"

    ```xml
    <record model="ir.ui.view" id="sprints_list">
        <field name="name">gestion_tareas_sergio.sprints_sergio.list</field>
        <field name="model">gestion_tareas_sergio.sprints_sergio</field>
        <field name="arch" type="xml">
            <list decoration-info="activo == True"
                decoration-muted="fecha_fin &lt; current_date()"
                decoration-warning="duracion &lt; 7"
                decoration-success="duracion == 14">
                <field name="bane"/>
                <field name="descripcion"/>
                <field name="duracion" sum="Total días" avg="Promedio"/>
                <field name="fecha_ini"/>
                <field name="fecha_fin"/>
                <field name="activo" widget="boolean_toggle"/>
            </list>
        </field>
    </record>
    ```

Esta vista:

- Resalta en azul los sprints activos
- Muestra en gris los sprints finalizados
- Alerta en amarillo sobre sprints muy cortos (menos de 7 días)
- Destaca en verde sprints de duración estándar (14 días)
- Muestra el total y promedio de días planificados
- Incluye el campo activo para condiciones pero lo oculta

!!! tip "Buenas Prácticas para Vistas Lista"

    Al diseñar vistas lista personalizadas, ten en cuenta estas recomendaciones:

    **Selección de campos**:

    - Muestra solo campos relevantes para identificar registros
    - Evita columnas muy anchas (como Text) que rompen el diseño
    - Prioriza campos que el usuario usa para buscar o filtrar

    **Uso de decoraciones**:

    - No abuses de los colores, pueden saturar visualmente
    - Usa colores con significado consistente (rojo=problema, verde=ok)
    - Las decoraciones más importantes deben ir al final para que prevalezcan

    **Sumatorios**:

    - Úsalos solo en campos donde el total tenga sentido
    - No sumes campos que representan promedios o porcentajes
    - Considera si el sumatorio es de todos los registros o solo los filtrados

    **Campos invisibles**:

    - Incluye campos necesarios para condiciones aunque no se muestren
    - Usa `invisible="1"` en lugar de no incluirlos para poder usarlos en decoraciones

    **Vistas editables**:
    
    - Úsalas solo para modelos muy simples
    - Si el formulario tiene lógica compleja, no uses editable
    - Considera que limitas la validación y experiencia de usuario

## Recursos Adicionales

Para profundizar en la personalización de vistas, consulta:

- [Documentación oficial de vistas](https://www.odoo.com/documentation/18.0/es/developer/reference/backend/views.html)
- [Guía de vistas lista](https://www.odoo.com/documentation/18.0/es/developer/reference/backend/views.html#list)
- Código fuente de módulos estándar de Odoo para ver ejemplos reales


---

## 🧩 Tu Turno: Gestor de Restaurante

Ahora mejorarás las vistas lista de tu proyecto del restaurante aplicando las técnicas de decoraciones, sumatorios y campos computados.

### Objetivos y Contexto

Vas a personalizar las vistas lista de Platos, Menús e Ingredientes, añadiendo decoraciones condicionales basadas en su estado, sumatorios de precios y campos computados que faciliten la identificación visual de información importante.

### Pasos a Realizar

1. **Crear vista lista personalizada para Platos**
    
    Define una vista lista mostrando: nombre, categoría, precio, precio_final, disponible.
    
    Pistas:
    
    - ID sugerido: `platos_list`
    - Incluye todos los campos que usarás en decoraciones

2. **Añadir decoraciones a la vista de Platos**
    
    Aplica diferentes colores según el estado del plato:
    
    - Gris para platos no disponibles (`decoration-muted`)
    - Verde para platos con descuento (`decoration-success`)
    - Naranja para platos caros (>20€) usando un campo computado
    
    Pistas:
    
    - Para platos caros, crea primero un campo booleano `es_caro` computado
    - Recuerda incluir campos invisibles si solo los necesitas para condiciones

3. **Añadir sumatorio de precios**
    
    Muestra el total de ingresos potenciales al pie de la columna `precio_final`.
    
    Pistas:
    
    - Usa atributo `sum="Total ingresos"` en el campo
    - Esto suma todos los registros visibles en la lista

4. **Crear vista lista para Menús con decoraciones**
    
    Personaliza la vista de menús mostrando: nombre, fecha_inicio, fecha_fin, activo, precio_total.
    
    Aplica decoraciones:
    
    - Azul para menús activos
    - Gris para menús finalizados (fecha_fin pasada)
    - Naranja para menús próximos a vencer (menos de 3 días)
    
    Pistas:
    
    - Para menús próximos a vencer, necesitas un campo computado `proximo_vencimiento`
    - Usa `current_date` para comparar con fecha actual
    - Añade sumatorio al campo `precio_total`

5. **Crear vista lista para Ingredientes**
    
    Personaliza la vista mostrando: nombre, es_alergeno, descripcion.
    
    Aplica decoración roja a ingredientes alergenos.
    
    Pistas:
    
    - Usa `decoration-danger="es_alergeno == True"`
    - Esta identificación visual es crítica para seguridad alimentaria

6. **Experimentar con vista editable (opcional)**
    
    Prueba hacer la vista de ingredientes editable añadiendo `editable="top"` al tag `<list>`.
    
    Evalúa si te resulta más cómodo para este modelo simple.

### Verificaciones

Comprueba que:

- Los platos se muestran con colores según disponibilidad, descuento y precio
- Se muestra el total de ingresos al pie de la lista de platos
- Los menús se colorean según su estado (activo, finalizado, próximo a vencer)
- Se muestra el total de precios de menús
- Los ingredientes alergenos destacan en rojo
- Los sumatorios se calculan correctamente

!!!example "Datos de Prueba"

    **Platos**:
    
    - Ensalada César (8€, disponible) → Normal
    - Solomillo Wellington (35€, disponible) → Naranja (caro)
    - Pizza Margarita (12€, 10% descuento) → Verde
    - Paella (18€, NO disponible) → Gris

    **Menús**:
    
    - Menú del Día (activo, vence en 5 días) → Azul
    - Menú San Valentín (activo, vence mañana) → Naranja
    - Menú Navidad (fecha pasada) → Gris

    **Ingredientes**:
    
    - Tomate → Normal
    - Gluten (alergeno) → Rojo
    - Huevo (alergeno) → Rojo