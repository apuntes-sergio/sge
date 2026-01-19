---
title: Campos con valores por defecto
description: Establecer valores iniciales en campos de modelos Odoo
---

# Campos con Valores por Defecto

En este apartado abordaremos la creación de **campos con valores por defecto** en Odoo, diferenciándolos de los campos computados y mostrando diversas formas de establecer estos valores iniciales.

Un **valor por defecto** es el valor inicial que tendrá un campo cuando se crea un nuevo registro.

**Características**:

- Se asigna automáticamente al crear un registro
- El usuario **puede modificarlo** después de la creación
- Se calcula una sola vez (al crear el registro)
- Simplifica la entrada de datos
- Reduce errores de usuario

**Casos de uso comunes**:

- Fechas actuales (fecha de creación, fecha de alta)
- Valores booleanos (activo: True, disponible: True)
- Números predeterminados (cantidad: 1, descuento: 0)
- Estados iniciales (estado: 'borrador')

## Diferencia con Campos Computados

Es importante entender la diferencia entre campos con valores por defecto y campos computados:

| Característica | Valor por Defecto | Campo Computado |
|----------------|-------------------|-----------------|
| **Cuándo se calcula** | Solo al crear | Cada vez que se lee o cambian dependencias |
| **Editable por usuario** | Sí | No (salvo que se programe) |
| **Parámetro usado** | `default=` | `compute=` |
| **Almacenamiento** | Siempre | Opcional (`store=True`) |
| **Uso típico** | Valor inicial sugerido | Cálculo automático continuo |

!!! example "Ejemplo visual"

    ```python
    # Valor por defecto - usuario puede cambiar
    cantidad = fields.Integer(default=1)

    # Campo computado - se recalcula siempre
    total = fields.Float(compute='_compute_total')
    ```

## Formas de Definir Valores por Defecto

### 1. Asignación Directa (Valores Estáticos)

La forma más simple es asignar directamente un valor al parámetro `default`:

!!!example "Ejemplo de asignación de varios campos por defecto"

    ```python
    class sprints_sergio(models.Model):
        _name = 'gestion_tareas_sergio.sprints_sergio'
        
        duracion = fields.Integer(
            string="Duración (días)",
            default=14,
            help="Duración en días del sprint")
    ```

**Tipos de valores según el campo**:

!!! example "Ejemplos de varios tipos de valores"

    ```python
    # Char / Text - entre comillas
    nombre = fields.Char(default="Sin título")

    # Integer / Float - números directos
    cantidad = fields.Integer(default=1)
    precio = fields.Float(default=0.0)

    # Boolean - True o False
    activo = fields.Boolean(default=True)

    # Date / Datetime - mejor usar funciones (ver siguiente sección)

    # Selection - uno de los valores definidos
    estado = fields.Selection(
        [('borrador', 'Borrador'), ('activo', 'Activo')],
        default='borrador')
    ```

!!! warning "Tipo de dato correcto"
    El valor asignado a `default` debe coincidir con el tipo de dato del campo. Un error común es poner `default="1"` (string) en un campo Integer.

### 2. Valores Calculados con Función

Para valores que requieren cálculo, se define una función y se pasa como valor por defecto:

!!! example "Ejemplo de calculo por defecto calculado. La hecha actual"

    ```python
    from datetime import datetime

    class tareas_sergio(models.Model):
        _name = 'gestion_tareas_sergio.tareas_sergio'
        
        # 1. Primero definimos la función
        def _get_fecha_actual(self):
            return datetime.now()
        
        # 2. Después la usamos en el campo (SIN comillas)
        fecha_creacion = fields.Datetime(
            string="Fecha de Creación",
            default=_get_fecha_actual)
    ```

**Aspectos clave**:

- La función se define **antes** del campo
- Se pasa **sin comillas** (diferencia con `compute`)
- La función recibe `self` como parámetro
- Se ejecuta **una sola vez** al crear el registro
- El usuario puede modificar el valor después

!!! example "Ejemplo con Lógica Más Compleja"

    ```python
    def _get_codigo_siguiente(self):
        """Genera el siguiente código secuencial"""
        ultimo = self.search([], order='id desc', limit=1)
        if ultimo:
            return f"TSK-{ultimo.id + 1:04d}"
        return "TSK-0001"

    codigo = fields.Char(
        string="Código",
        default=_get_codigo_siguiente)
    ```

### 3. Uso de Funciones Lambda

Cuando la lógica es sencilla, se puede usar una función *lambda* directamente:

!!! example "Asignación usando funcion lambda"

    ```python
    from datetime import datetime

    class tareas_sergio(models.Model):
        _name = 'gestion_tareas_sergio.tareas_sergio'
        
        fecha_creacion = fields.Datetime(
            string="Fecha de Creación",
            default=lambda self: datetime.now())
    ```

**Ventajas de lambda**:

- Código más conciso
- No necesitas definir función aparte
- Ideal para lógica simple

Cuándo usar lambda vs función normal:

!!! example "Uso de función normal vs lambda"

    ```python
    # Lambda - lógica simple (1 línea)
    fecha = fields.Date(default=lambda self: fields.Date.today())

    # Función normal - lógica compleja (múltiples líneas)
    def _get_valor_complejo(self):
        if condicion:
            return valor1
        else:
            return valor2

    campo = fields.Char(default=_get_valor_complejo)
    ```

### 4. Valores por Defecto del Usuario Actual

Un caso común es asignar el usuario actual como valor por defecto:

!!! example "Asignación del usuario actual a la acción realizada"

    ```python
    responsable = fields.Many2one(
        'res.users',
        string='Responsable',
        default=lambda self: self.env.user.id)
    ```

**Explicación**:

- `self.env`: Entorno de Odoo
- `self.env.user`: Usuario actual
- Se asigna automáticamente al crear el registro


### 5. Valores por Defecto con Búsquedas

También puedes buscar registros existentes para usar como valor por defecto:

!!! example "Resultado de búsqueda como valor por defecto"

    Añadimos al proyecto un campo para indicar si esta o no activo

    ```python
    activo = fields.Boolean(
        string= "Estado del proyecto",
        default = True
    )    
    ```

    y ahora en la tarea calculamos el proyecto activo 

    ```python
    def _get_proyecto_activo(self):
        """Retorna el proyecto marcado como activo"""
        return self.env['gestion_tareas_sergio.proyectos_sergio'].search(
            [('activo', '=', True)], 
            limit=1, order='create_date desc')

    proyecto = fields.Many2one(
        'gestion_tareas_sergio.proyectos_sergio',
        string='Proyecto',
        default=_get_proyecto_activo)
    ```

    - `create_date` es un campo propio de odoo donde tenemos la fecha de creación del registro.

## Comparativa: Default vs Compute

Veamos el siguiente ejemplo donde por una parte tenemos un valor por defecto en `fecha_alta` que se asigna la primera vez que se crea un registro y un campo computado `dias_activo` que se recalcula cada vez que se accede al un registro.

!!! example "Ejemplo Lado a Lado"

    ```python
    class Ejemplo(models.Model):
        _name = 'ejemplo'
        
        # VALOR POR DEFECTO
        # - Se asigna al crear
        # - Usuario puede cambiar
        # - Se calcula UNA vez
        fecha_alta = fields.Date(
            string="Fecha de Alta",
            default=lambda self: fields.Date.today())
        
        # CAMPO COMPUTADO
        # - Se recalcula siempre
        # - Usuario NO puede cambiar
        # - Depende de otros campos
        dias_activo = fields.Integer(
            string="Días Activo",
            compute='_compute_dias_activo')
        
        @api.depends('fecha_alta')
        def _compute_dias_activo(self):
            for record in self:
                if record.fecha_alta:
                    delta = fields.Date.today() - record.fecha_alta
                    record.dias_activo = delta.days
                else:
                    record.dias_activo = 0
    ```

## Visualización en Vistas

Para que el usuario vea el valor por defecto, simplemente añade el campo a la vista:

```xml
<record model="ir.ui.view" id="tareas_form">
    <field name="name">tareas.form</field>
    <field name="model">gestion_tareas_sergio.tareas_sergio</field>
    <field name="arch" type="xml">
        <form>
            <sheet>
                <group>
                    <field name="fecha_creacion"/>
                    <field name="responsable"/>
                    <field name="estado"/>
                </group>
            </sheet>
        </form>
    </field>
</record>
```

Al crear un nuevo registro, estos campos aparecerán pre-rellenados con sus valores por defecto.

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/18_EjemploValoresDefecto.png){ width="75%" }
  <figcaption>Campos con valores por defecto pre-rellenados</figcaption>
</figure>

## Casos de Uso Comunes

!!! example "1. Fecha Actual"

    ```python
    fecha_creacion = fields.Date(
        string="Fecha de Creación",
        default=lambda self: fields.Date.today())

    fecha_hora_creacion = fields.Datetime(
        string="Fecha y Hora de Creación",
        default=lambda self: fields.Datetime.now())
    ```

!!! example "2. Usuario Actual"

    ```python
    creado_por = fields.Many2one(
        'res.users',
        string='Creado por',
        default=lambda self: self.env.user.id
        readonly=True)
    ```

!!! example "3. Booleanos"

    ```python
    activo = fields.Boolean(
        string="Activo",
        default=True)

    publicado = fields.Boolean(
        string="Publicado",
        default=False)
    ```

!!! example "4. Estados Iniciales"

    ```python
    estado = fields.Selection([
        ('borrador', 'Borrador'),
        ('revision', 'En Revisión'),
        ('aprobado', 'Aprobado'),
        ('rechazado', 'Rechazado')
    ], default='borrador')
    ```

!!! example "5. Valores Numéricos"

    ```python
    descuento = fields.Float(
        string="Descuento (%)",
        default=0.0)

    cantidad = fields.Integer(
        string="Cantidad",
        default=1)

    prioridad = fields.Selection([
        ('1', 'Baja'),
        ('2', 'Normal'),
        ('3', 'Alta')
    ], default='2')
    ```

!!!tip "Buenas Prácticas"

    !!! example "1. Usa el Tipo Correcto"
        ```python
        # Mal - tipo incorrecto
        cantidad = fields.Integer(default="1")  # String en lugar de int

        # Bien
        cantidad = fields.Integer(default=1)
        ```

    !!! example "2. Funciones vs Lambda"
        ```python
        # Usa lambda para lógica simple
        fecha = fields.Date(default=lambda self: fields.Date.today())

        # Usa función para lógica compleja
        def _get_codigo(self):
            # Múltiples líneas de código
            # ...
            return codigo

        codigo = fields.Char(default=_get_codigo)
        ```

    !!! example "3. No Calcules Valores Caros"
        ```python
        # Mal - operación costosa en valor por defecto
        def _get_estadisticas_complejas(self):
            # Consulta pesada a BD
            # Cálculos complejos
            return resultado

        campo = fields.Char(default=_get_estadisticas_complejas)  # Lento

        # Bien - usa campo computado para cálculos costosos
        campo = fields.Char(compute='_compute_estadisticas')
        ```

    !!! example "4. Documenta Valores No Obvios"
        ```python
        duracion = fields.Integer(
            string="Duración",
            default=14,
            help="Duración predeterminada del sprint en días (2 semanas)")
        ```

    !!! example "5. Considera la Zona Horaria"
        ```python
        # Para fechas con hora, considera la zona horaria del usuario
        def _get_fecha_hora(self):
            return fields.Datetime.now()  # UTC por defecto

        # O usa la zona horaria del usuario
        def _get_fecha_hora_local(self):
            from datetime import datetime
            import pytz
            
            tz = pytz.timezone(self.env.user.tz or 'UTC')
            return datetime.now(tz)
        ```

!!! warning "Errores Comunes"

    !!! example "1. Pasar Función con Paréntesis"
        ```python
        # Mal - ejecuta la función inmediatamente
        fecha = fields.Date(default=fields.Date.today())

        # Bien - pasa la función sin ejecutar
        fecha = fields.Date(default=lambda self: fields.Date.today())
        ```

    !!! example "2. Usar Comillas con Funciones"
        ```python
        # Mal - esto es para compute, no para default
        fecha = fields.Date(default='_get_fecha')

        # Bien
        fecha = fields.Date(default=_get_fecha)
        # o
        fecha = fields.Date(default=lambda self: fields.Date.today())
        ```

    !!! example "3. Valores Mutables"
        ```python
        # Mal - lista mutable compartida entre instancias
        valores = fields.Char(default=[])  # ¡Peligro!

        # Bien - usa lambda para crear nueva instancia
        valores = fields.Char(default=lambda self: [])
        ```

---

## 🧩 Tu Turno: Gestor de Restaurante

Ahora añadirás valores por defecto útiles a tu proyecto del restaurante.

### Objetivos y Contexto

Añadirás valores por defecto a varios campos para mejorar la usabilidad:

- Fechas automáticas de creación
- Estados iniciales predeterminados
- Valores numéricos por defecto
- Usuario actual como responsable

### Pasos a Realizar

1. **Fecha de alta automática en Platos**
    
    Añade un campo `fecha_alta` de tipo `Date` con valor por defecto de la fecha actual.
    
    Pistas:

    - Usa `lambda self: fields.Date.today()`
    - O define una función que retorne `fields.Date.today()`

2. **Disponibilidad por defecto en Platos**
    
    Modifica el campo `disponible` (Boolean) para que por defecto sea `True`.
    
    Pista: `default=True`

3. **Descuento por defecto**
    
    El campo `descuento` debe tener valor por defecto de 0.0.
    
    Pista: Usa valor numérico directo

4. **Categoría por defecto para Platos**
    
    Crea una función que busque la categoría "Sin Clasificar" y la asigne por defecto.
    
    Pistas:

    - Define función `_get_categoria_defecto`
    - Usa `self.env['modelo.categoria'].search([('nombre', '=', 'Sin Clasificar')], limit=1)`
    - Primero crea la categoría "Sin Clasificar" para que exista

5. **Precio base por defecto**
    
    El campo `precio` en Platos debe tener un valor mínimo sugerido de 5.0.


6. **Estado inicial de Menús**
    
    El campo `activo` en Menús debe ser `False` por defecto (para que se active manualmente).

7. **Usuario creador en Menús**
    
    Añade un campo `creado_por` de tipo `Many2one` a `res.users` con valor por defecto del usuario actual.
    
    Haz que este campo no se pueda cambiar
    
    Pistas:

    - función lambda: `self.env.user`
    - Marca el campo como `readonly`

8. **Duración por defecto de disponibilidad de Menú**
    
    Si añades un campo `dias_disponible` (Integer) a Menú, establece default=7 (una semana).

9. **Calcula automáticamente la fecha de finalización del menú**

    A partir del campo anteior `dias_disponible` calcula de forma automática el valor de `fecha_fin`


10. **Actualizar vistas**
    
    Añade los nuevos campos a las vistas para que sean visible.

### Verificaciones y Resultado Esperado

Comprueba que:

- Al crear un nuevo plato, la fecha de alta se rellena automáticamente con la fecha actual
- Al crear un nuevo plato, `disponible` está marcado como True
- Al crear un nuevo menú, aparece tu usuario en `creado_por`
- Los valores numéricos tienen sus valores por defecto correctos
- Puedes modificar todos estos valores después de crearlos (no son readonly salvo `creado_por`)

**Pruebas sugeridas**:

1. Crea un nuevo plato sin tocar ningún campo
2. Verifica que `fecha_alta`, `disponible`, `descuento` y `precio` tienen valores automáticos
3. Modifica el precio - debe permitirte cambiarlo
4. Crea un nuevo menú
5. Verifica que `creado_por` muestra tu usuario y no puedes cambiarlo
