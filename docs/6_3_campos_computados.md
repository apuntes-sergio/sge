---
title: Campos computados
description: Creación y uso de campos computados en modelos de Odoo
---

# Campos Computados

En esta sección abordaremos la creación de **campos computados** en Odoo, es decir, aquellos campos cuyo valor **no** es introducido manualmente por el usuario, sino que se determina automáticamente en función de otros atributos del modelo. Este tipo de campos resulta fundamental para mantener la integridad y coherencia de los datos, así como para automatizar procesos dentro del sistema.

## ¿Qué es un Campo Computado?

Un campo computado es un campo cuyo valor se calcula mediante una función Python en lugar de ser ingresado directamente por el usuario.

**Características**:

- El valor se genera automáticamente
- Se calcula mediante un método del modelo
- Puede almacenarse en la base de datos o calcularse dinámicamente
- Puede depender de otros campos del modelo

**Casos de uso comunes**:

- Códigos automáticos (referencias, SKU)
- Totales y subtotales
- Fechas calculadas
- Estados derivados de otras condiciones
- Contadores y estadísticas

## Campo Computado Sin Almacenamiento

Como primer ejemplo, crearemos un campo computado llamado `codigo` que generará automáticamente un código único para cada tarea.

### Definición del Campo

```python
# En el modelo tareas_sergio
codigo = fields.Char(compute="_get_codigo")
```

El parámetro `compute` indica el nombre del método que calculará el valor del campo.

### Implementación del Método

!!! example "ejemplo de implementación de método"

    ```python
    def _get_codigo(self):
        for tarea in self:
            # Si la tarea no tiene un sprint asignado
            if not tarea.sprint:
                tarea.codigo = "TSK_" + str(tarea.id)
            else:
                # Si tiene sprint, usamos su nombre
                tarea.codigo = str(tarea.sprint.name).upper() + "_" + str(tarea.id)
    ```

**Explicación del código**:

- `for tarea in self`: Itera sobre todos los registros de la colección
- `tarea.id`: Es la clave primaria generada automáticamente por Odoo
- Si no hay sprint asignado: `TSK_<id>` (ejemplo: `TSK_42`)
- Si hay sprint: `<NOMBRE_SPRINT>_<id>` (ejemplo: `SPRINT1_42`)

!!! note "Campo id automático"
    El campo `id` es la clave primaria generada automáticamente en todos los modelos de Odoo. Puedes comprobarlo revisando las tablas en PostgreSQL.

### Características de este Campo

**Sin almacenamiento en base de datos**:

- El valor se recalcula dinámicamente cada vez que se visualiza
- No ocupa espacio en la base de datos
- Se actualiza automáticamente si cambian los valores de los que depende

**Para visualizarlo**:

- Debe añadirse explícitamente en la vista XML

```xml
<field name="codigo"/>
```

### Verificación

Para comprobar que el campo no se almacena:

1. Revisa la tabla en PostgreSQL
2. No encontrarás una columna `codigo`
3. El valor se genera al mostrar el registro

**Comportamiento**:

- Al crear una tarea sin sprint: código `TSK_<id>`
- Al asignar un sprint y guardar: código se actualiza a `<SPRINT>_<id>`

## Campo Computado con Almacenamiento y Dependencias

En situaciones más complejas, necesitamos campos computados que:

- Se almacenen en la base de datos
- Solo se recalculen cuando cambien sus dependencias
- Estén disponibles para búsquedas y filtros

Este enfoque es útil cuando el cálculo es costoso o cuando el valor debe ser indexable.

### Caso de Uso: Fecha Fin de Sprint

Supongamos que en el modelo `Sprint` tenemos:

- `fecha_ini`: Fecha de inicio (tipo Datetime)
- `duracion`: Duración en días (tipo Integer)
- `fecha_fin`: **Campo computado** que se calcula automáticamente

!!!example "Implementación"

    ```python
    from datetime import timedelta
    from odoo import models, fields, api

    class sprints_sergio(models.Model):
        _name = 'gestion_tareas_sergio.sprints_sergio'
        _description = 'Modelo de Sprints para Gestión de Proyectos'

        nombre = fields.Char(string="Nombre", required=True)
        fecha_ini = fields.Datetime(string="Fecha Inicio", required=True)
        duracion = fields.Integer(
            string="Duración", 
            help="Cantidad de días que tiene asignado el sprint")
        
        fecha_fin = fields.Datetime(
            compute='_compute_fecha_fin', 
            store=True,
            string="Fecha Fin")

        @api.depends('fecha_ini', 'duracion')
        def _compute_fecha_fin(self):
            for sprint in self:
                if sprint.fecha_ini and sprint.duracion and sprint.duracion > 0:
                    sprint.fecha_fin = sprint.fecha_ini + timedelta(days=sprint.duracion)
                else:
                    sprint.fecha_fin = sprint.fecha_ini
    ```

Aspectos Clave de la Implementación

**1. Decorador `@api.depends`**:
```python
@api.depends('fecha_ini', 'duracion')
```

- Indica qué campos disparan el recálculo
- El campo solo se recalcula cuando cambian `fecha_ini` o `duracion`
- Optimiza el rendimiento evitando cálculos innecesarios

**2. Parámetro `store=True`**:
```python
fecha_fin = fields.Datetime(compute='_compute_fecha_fin', store=True)
```

- El valor calculado se almacena en la base de datos
- Permite usar el campo en búsquedas y filtros
- El valor persiste y no se recalcula en cada lectura

**3. Lógica del Método**:
```python
for sprint in self:
    if sprint.fecha_ini and sprint.duracion and sprint.duracion > 0:
        sprint.fecha_fin = sprint.fecha_ini + timedelta(days=sprint.duracion)
    else:
        sprint.fecha_fin = sprint.fecha_ini
```

- Verifica que existan los valores necesarios
- Suma la duración en días a la fecha de inicio
- Si falta algún dato, usa la fecha de inicio como valor por defecto

**4. Importación de `timedelta`**:
```python
from datetime import timedelta
```

- Necesaria para sumar días a una fecha
- `timedelta(days=10)` representa 10 días

### Verificación en Base de Datos

Tras actualizar el módulo:

1. Revisa la tabla en PostgreSQL
2. **Ahora sí** encontrarás una columna `fecha_fin`
3. El valor está almacenado y persiste

### Comportamiento

Al crear o editar un sprint:

1. Usuario introduce `fecha_ini`: 2024-01-01
2. Usuario introduce `duracion`: 14 (días)
3. Odoo calcula automáticamente `fecha_fin`: 2024-01-15
4. El valor se guarda en la base de datos

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/14_EjemploCampoCalculado.png){ width="75%" }
  <figcaption>Ejemplo de funcionamiento de campo calculado con almacenamiento</figcaption>
</figure>

## Comparativa: Con y Sin Almacenamiento

| Característica | Sin `store` | Con `store=True` |
|----------------|-------------|------------------|
| **Almacenamiento** | No se guarda en BD | Se guarda en BD |
| **Cálculo** | Cada vez que se lee | Solo cuando cambian dependencias |
| **Búsquedas** | No disponible | Disponible |
| **Filtros** | No disponible | Disponible |
| **Rendimiento** | Más lento en lectura | Más rápido en lectura |
| **Uso de disco** | Ninguno | Ocupa espacio |
| **Uso típico** | Cálculos simples y rápidos | Cálculos costosos o indexables |

**Cuándo Usar Cada Tipo:** 

- Campos **computados sin almacenamiento** (sin `store`):
    - Cálculos muy simples y rápidos
    - Valores que cambian constantemente
    - No necesitas buscar o filtrar por ese campo
    - Ejemplo: Formateo de texto, concatenaciones simples

- Campos **computados con almacenamiento** (`store=True`):
    - Cálculos complejos o costosos
    - Necesitas buscar o filtrar por ese campo
    - El valor no cambia frecuentemente
    - Quieres usar el campo en vistas pivot o gráficas
    - Ejemplo: Totales, fechas calculadas, contadores

## Solución de Problemas

!!! warning "Errores al actualizar modelos"
    En ocasiones, los cambios en campos computados pueden generar inconsistencias en la base de datos. Si el ORM no se refresca correctamente tras reiniciar el servicio, la solución es eliminar toda la base de datos:
    
    ```bash
    docker compose down -v
    docker compose up -d
    ```
    
    **Advertencia**: Esto eliminará todos los datos. Haz una copia de seguridad si los necesitas.

Errores Comunes

**1. Campo no se actualiza**:

- Verifica que el decorador `@api.depends` incluya todos los campos necesarios
- Asegúrate de que la sintaxis del método es correcta

**2. Error "field 'X' does not exist"**:

- El campo debe estar definido en el modelo antes del método compute
- Reinicia el servidor después de añadir el campo

**3. El campo aparece vacío**:

- Verifica que la lógica del método asigna un valor en todos los casos
- Comprueba que los campos dependientes tienen valores

**4. Error de recursión**:

- No uses el propio campo computado dentro de `@api.depends`
- No modifiques campos que disparan el mismo método compute

!!! tip "Buenas Prácticas"

    1. **Nombra los métodos con prefijo `_compute_`**:
       ```python
       def _compute_total(self):
       def _compute_codigo(self):
       ```

    2. **Usa `@api.depends` siempre que sea posible**:
        - Mejora el rendimiento
        - Hace el código más mantenible
        - Evita cálculos innecesarios

    3. **Maneja casos nulos o vacíos**:
       ```python
       if sprint.fecha_ini and sprint.duracion:
           # cálculo
       else:
           # valor por defecto
       ```

    4. **Almacena solo cuando sea necesario**:
        - Solo usa `store=True` si necesitas búsquedas/filtros
        - Considera el espacio en disco

    5. **Documenta la lógica compleja**:
       ```python
       def _compute_total(self):
           """Calcula el total sumando precio * cantidad para cada línea"""
           for record in self:
               # ...
       ```

---

## 🧩 Tu Turno: Gestor de Restaurante

Ahora se aplicarán campos computados en el proyecto del restaurante.

### Objetivos y Contexto

Añadir tres campos computados al modelo Plato y uno al modelo Menú:

1. **Código de plato** (sin almacenar)
2. **Precio con IVA** (sin almacenar)
3. **Precio final** (almacenado, con descuento opcional)
4. **Precio total del menú** (almacenado)

### Pasos a Realizar

1. **Campo computado sin almacenamiento: Código del plato**
    
    En el modelo Plato, crea un campo `codigo` de tipo `Char` con `compute="_get_codigo"`.
    
    El método debe generar un código basado en:  

    - Si no tiene categoría: `PLT_<id>`
    - Si tiene categoría: `<CATEGORÍA>_<id>` (primeras 3 letras en mayúsculas)
    
    Pistas:

    - Itera sobre `self` en el método
    - Usa `plato.categoria[:3].upper()` para obtener las primeras 3 letras en mayúsculas
    - Usa `plato.id` como identificador único

2. **Campo computado sin almacenamiento: Precio con IVA**
    
    En el modelo Plato, crea un campo `precio_con_iva` de tipo `Float` con `compute="_compute_precio_con_iva"`.
    
    El método debe calcular el precio con un IVA del 10% (multiplica por 1.10).
    
    Pistas:

    - Verifica que `plato.precio` existe antes de operar
    - Si no existe precio, asigna 0.0

3. **Añadir campo descuento al modelo Plato**
    
    Añade un campo `descuento` de tipo `Float` con `string="Descuento (%)"`.
    
    Este campo será editable manualmente por el usuario (no computado).

4. **Campo computado con almacenamiento: Precio final**
    
    En el modelo Plato, crea un campo `precio_final` de tipo `Float` que:

    - Sea computado (`compute=...`)
    - Se almacene en BD (`store=True`)
    - Dependa de `precio` y `descuento` (usa `@api.depends`)
    
    El método debe:

    - Si hay descuento: aplicarlo al precio (restando el porcentaje)
    - Si no hay descuento: usar el precio original
    
    Pistas:

    - Convierte el porcentaje a decimal: `descuento / 100.0`
    - Fórmula: `precio * (1 - descuento_decimal)`
    - Maneja el caso donde no hay precio definido

5. **Campo computado en Menú: Precio total**
    
    En el modelo Menú, crea un campo `precio_total` de tipo `Float` que:

    - Sea computado
    - Se almacene en BD
    - Dependa de los platos del menú y su precio_final
    
    El método debe sumar el `precio_final` de todos los platos del menú.
    
    Pistas:

    - Usa `@api.depends('platos', 'platos.precio_final')`
    - Usa la función `sum()` de Python
    - Itera sobre `self` para cada menú
    - Recuerda que `platos` es una relación One2many
    !!!tip "calculos de suma"

        ```python
        # Suma el precio_final de todos los platos relacionados
        # La función map() extrae los valores y sum() los agrega.
        precios = menu.platos_ids.mapped('precio_final')
        menu.precio_total = sum(precios)
        ``` 

6. **Actualizar vistas para mostrar los campos computados**
    
    Modifica las vistas XML para incluir los nuevos campos:
    
    Vista de Plato (list): Añade `codigo`, `precio_con_iva` y `precio_final`
    
    Vista de Plato (form): Organiza todos los campos de precio en el formulario
    
    Vista de Menú (form): Añade el campo `precio_total`
    
    Pistas:

    - Los campos computados se añaden como cualquier otro campo
    - Decide qué campos mostrar en lista vs formulario

### Verificaciones y Resultado Esperado

Comprueba que:

- El código del plato se genera automáticamente según su categoría
- El precio con IVA se calcula correctamente (sin almacenarse en BD)
- El precio final aplica el descuento correctamente (y sí se almacena en BD)
- El precio total del menú suma todos sus platos
- Al cambiar el descuento de un plato, se recalcula el precio final
- Al añadir/quitar platos de un menú, se actualiza su precio total


!!!example "Sugerencia Datos de prueba"

    **Plato 1**: Pizza Margarita

    - Precio: 10€
    - Descuento: 10%
    - Código esperado: `PRI_<id>` (si categoría es "principal")
    - Precio con IVA: 11€
    - Precio final: 9€

    **Plato 2**: Ensalada César

    - Precio: 8€
    - Descuento: 0%
    - Código esperado: `ENT_<id>` (si categoría es "entrante")
    - Precio con IVA: 8.8€
    - Precio final: 8€

    **Menú del Día**:
    
    - Contiene: Pizza Margarita + Ensalada César
    - Precio total esperado: 17€ (9 + 8)


Luego verifica:

- ¿Se generan los códigos correctamente según categoría?
- ¿El precio con IVA es el 110% del precio original?
- ¿El precio final aplica correctamente el descuento?
- ¿El precio total del menú es la suma de sus platos?


