---
title: Excepciones y mensajes de log
description: Manejo de errores y registro de mensajes en Odoo
---

# Excepciones y Mensajes de Log

En este apartado abordaremos el manejo de excepciones y la gestión de mensajes de log en Odoo, aspectos fundamentales para garantizar la robustez y mantenibilidad del código. Aprenderemos a lanzar excepciones de manera controlada y a registrar información relevante en el log, utilizando los distintos niveles de severidad que ofrece el framework.

## Manejar Excepciones

El manejo adecuado de excepciones permite:

- Proporcionar mensajes claros al usuario cuando algo va mal
- Evitar que la aplicación se comporte de forma inesperada
- Facilitar la depuración durante el desarrollo
- Mantener la integridad de los datos
- Mejorar la experiencia de usuario

**Sin manejo de excepciones**:
```
Error interno del servidor
Traceback (most recent call last):
  File "/usr/lib/python3/...", line 1234, in method
    result = object.field.name
AttributeError: 'NoneType' object has no attribute 'name'
```

**Con manejo de excepciones**:
```
Validación fallida
Debe asignar un sprint antes de guardar la tarea.
```

## Excepciones en Odoo

Odoo proporciona excepciones específicas para diferentes situaciones. Para más información, consulta la [documentación oficial sobre gestión de errores](https://www.odoo.com/documentation/18.0/es/developer/reference/frontend/error_handling.html).

### Tipos de Excepciones Principales

En el desarrollo de módulos de Odoo, el manejo de errores no se limita a detener la ejecución del código. El framework utiliza excepciones específicas definidas en `odoo.exceptions` para gestionar la **integridad de la base de datos** (mediante *rollbacks* automáticos) y la **comunicación con la interfaz de usuario**.

Utilizar la excepción correcta garantiza que el usuario reciba un mensaje claro y que el sistema se comporte de manera previsible según los estándares de Odoo.

| Excepción | Impacto en la Interfaz (UI) | Caso de Uso Correcto |
| --- | --- | --- |
| **`ValidationError`** | Ventana emergente de error de validación. | **Reglas de negocio estrictas.** Ej: "La fecha de fin no puede ser anterior a la de inicio". Se usa mucho con `@api.constrains`. |
| **`UserError`** | Ventana emergente de advertencia. | **Acciones ilógicas del usuario.** Ej: Intentar cancelar una factura que ya está pagada o confirmar un pedido sin productos. |
| **`AccessError`** | Mensaje de restricción de seguridad. | **Permisos y ACLs.** Ej: Un usuario intenta modificar un registro de un departamento al que no pertenece. |
| **`RedirectWarning`** | Ventana con un botón de acción. | **Errores con solución guiada.** Ej: "No has configurado una cuenta de ingresos. [Ir a configuración]". |
| **`MissingError`** | Mensaje de registro no encontrado. | **Registros eliminados.** Cuando el código intenta acceder a un ID que ya no existe en la base de datos. |

!!!tip "Notas sobre los diferentes tipos de excepciones"
    * **Diferencia entre `ValidationError` y `UserError`:** Aunque visualmente son similares, `ValidationError` se lanza automáticamente al intentar guardar un registro que no cumple los requisitos (normalmente mediante restricciones de Python), mientras que `UserError` se suele lanzar dentro de botones o métodos de acción.
    * **La excepción `Warning` (Obsoleta):** En versiones modernas de Odoo (v13+), se prefiere el uso de `UserError` en lugar de `Warning`, ya que esta última ha sido marcada para desaparecer en el núcleo del framework.


### Importar Excepciones

Primero, importa las excepciones necesarias al inicio de `models.py`:

```python
from odoo import models, fields, api
from odoo.exceptions import ValidationError, UserError
```

## Ejemplo Práctico: Validación en Campo Computado

Vamos a mejorar el método `_get_codigo` del modelo de Tareas para manejar correctamente los errores.

### Sin Manejo de Excepciones (Problemático)

```python
def _get_codigo(self):
    for tarea in self:
        # Si no hay sprint, esto fallará con un error críptico
        tarea.codigo = str(tarea.sprint.nombre).upper() + "_" + str(tarea.id)
```

**Problema**: Si `tarea.sprint` es `None` (no hay sprint asignado), obtendremos un error poco claro.

### Con Manejo de Excepciones (Correcto)

```python
def _get_codigo(self):
    for tarea in self:
        try:
            # Generamos el código
            tarea.codigo = str(tarea.sprint.nombre).upper() + "_" + str(tarea.id)
            
        except Exception as e:
            raise ValidationError(f"Error al generar el código: {str(e)}")
```

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/15_EjemploExcpecionValidacion.png){ width="75%" }
  <figcaption>Mensaje de excepción por validación</figcaption>
</figure>

**Ventajas**:

- Mensaje claro al usuario
- Control explícito del flujo de errores
- Fácil de depurar

### Provocar un Error para Pruebas

Vamos a provocar un error en nuestro código, para ver como funciona esta gestión de excepciones.

Vamos a modificar el cálculo de la fecha final de y vamos poner mal el nombre del campo. En el código inicial

```python
@api.depends('fecha_ini', 'duracion')
def _compute_fecha_fin(self):
    for sprint in self:
        if sprint.fecha_ini and sprint.duracion and sprint.duracion > 0:
            sprint.fecha_fin = sprint.fecha_ini + timedelta(days=sprint.duracion)
        else:
            sprint.fecha_fin = sprint.fecha_ini
```

cambiamos cualquier variablem para ver cómo reacciona e incluimos el bloque `try-excetp`

```python
@api.depends('fecha_ini', 'duracion')
def _compute_fecha_fin(self):
    for sprint in self:
        try:
            if sprint.fecha_ini and sprint.duracion and sprint.duracion > 0:
                sprint.fecha_fin = sprint.fecha_inicial + timedelta(days=sprint.duracion)
            else:
                sprint.fecha_fin = sprint.fecha_inicial #campo erróneo
        except Exception as e:            
            raise ValidationError(f"Error al generar el código: {str(e)}")
            # raise UserError(f"Error al generar el código: {str(e)}")
```

Esto provocará un error que será capturado por el `except` y mostrado al usuario de forma controlada.

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/16_EjemploExcpecionError.png){ width="75%" }
  <figcaption>Mensaje de excepción capturando un error</figcaption>
</figure>

## Validaciones con Decorador `@api.constrains`

Otra forma de validar datos es usando el decorador `@api.constrains`, que se ejecuta automáticamente cuando cambian ciertos campos.

### Ejemplo: Validar Fechas de Sprint

```python
@api.constrains('fecha_ini', 'fecha_fin')
def _check_fechas(self):
    for sprint in self:
        if sprint.fecha_fin and sprint.fecha_ini:
            if sprint.fecha_fin < sprint.fecha_ini:
                raise ValidationError(
                    "La fecha de fin no puede ser anterior a la fecha de inicio."
                )
```

**Características**:

- Se ejecuta automáticamente al guardar
- Solo cuando cambian los campos especificados
- Lanza `ValidationError` si algo está mal

### Ejemplo: Validar Precio Positivo

```python
@api.constrains('precio')
def _check_precio(self):
    for plato in self:
        if plato.precio and plato.precio < 0:
            raise ValidationError("El precio debe ser mayor o igual a cero.")
```

## Mensajes de Log

El registro de mensajes en el log es esencial para el seguimiento y diagnóstico de la aplicación. Odoo utiliza el módulo estándar de Python `logging`.

### Niveles de Log

| Nivel | Uso | Cuándo Usarlo |
|-------|-----|---------------|
| **DEBUG** | Información detallada | Depuración durante desarrollo |
| **INFO** | Mensajes informativos | Eventos importantes del flujo normal |
| **WARNING** | Advertencias | Situaciones anormales pero no críticas |
| **ERROR** | Errores | Fallos que impiden completar una operación |
| **CRITICAL** | Errores críticos | Fallos graves que pueden detener el sistema |

### Configurar el Logger

Al inicio del archivo `models.py`, después de las importaciones:

```python
from odoo import models, fields, api
from odoo.exceptions import ValidationError
import logging

_logger = logging.getLogger(__name__)
```

**Explicación**:

- `logging.getLogger(__name__)`: Crea un logger con el nombre del módulo actual
- `_logger`: Variable para acceder al logger en todo el archivo
- Permite identificar fácilmente de dónde provienen los mensajes

### Usar el Logger

```python
def _get_codigo(self):
    _logger.info("Iniciando generación de códigos de tareas")

    for tarea in self:
        try:
            if not tarea.sprint:
                _logger.warning(f"Tarea {tarea.id} sin sprint asignado")
                tarea.codigo = "TSK_" + str(tarea.id)

            else:
                # Si tiene sprint, usamos su nombre
                tarea.codigo = str(tarea.sprint.name).upper() + "_" + str(tarea.id)

            _logger.debug(f"Código generado: {tarea.codigo}")

        except Exception as e:
            _logger.error(f"Error generando código para tarea {tarea.id}: {str(e)}")
            raise ValidationError(f"Error al generar el código: {str(e)}")
```

Por cierto, ejecutando al anterior código y revisando los logs, veremos que constantemente estamos cambiando el código de la tarea, lo cual no parece muy eficiente y por lo tanto deberíamos poner remedio, por ejemplo indicando que solo se calculará el código en caso de que el cambie el sprint asociado. Así añadimos la siguiente linea con un `@api.depends`:

```python
    @api.depends('sprint', 'sprint.name')   # solo se ejecuta si cambia el sprint.
    def _get_codigo(self):
        ....
```

!!!tip "Compute & Depends en la Creación (Create)"
    
    Odoo ejecuta **todos** los métodos `compute` durante la creación de un registro, incluso si los campos del `@api.depends` no han cambiado o están vacíos. Esto lo hace para dar un valor inicial al campo.

    Así pues, lo que sucede en el método anterior con el campo `id` de propia tarea es que antes de guardar, el `id` es un "Virtual ID" (ej. `NewId_1`). El compute se ejecuta para mostrar una previsualización, y al al Guardar (Flush) Odoo asigna el ID real de la base de datos y **vuelve a ejecutar** el `compute` automáticamente antes de hacer el commit final.

    O sea, Odoo gestiona la creación como un evento especial. El `depends` solo sirve para **actualizaciones futuras** de registros que ya existen.


### Configurar Nivel de Log

Por defecto, Odoo solo muestra mensajes de nivel `INFO` o superior. Para ver mensajes `DEBUG`, modifica el archivo `odoo.conf`:

```ini
[options]
log_level = debug
```

Después de modificar el archivo, reinicia el servidor:

```bash
docker compose restart odoo_dev_dam
```

### Ver los Logs

Los logs se muestran en la consola o en archivos según tu configuración:

**Ver logs en tiempo real**:
```bash
docker logs odoo_dev_dam -f
```

**Filtrar logs de tu módulo**:
```bash
docker logs odoo_dev_sge -f 2>&1 | grep --line-buffered DEBUG | grep "gestion_tareas"
```

!!! tip "Filtrado de salida de docker"

    Para filtrar logs de un contenedor en tiempo real de forma efectiva, la estructura ideal es:

    `docker logs <contenedor> -f 2>&1 | grep --line-buffered "FILTRO1" | grep "FILTRO2"`

    donde:

    - **`2>&1` (Redirección de stderr a stdout):**
        * Docker y muchas aplicaciones (como Odoo/Python) envían los mensajes de log y errores al flujo de **Error Estándar (stderr)**, mientras que `grep` solo analiza la **Salida Estándar (stdout)** por defecto.
        * Este comando une ambos flujos para que `grep` pueda "leerlo" todo.


    - **`--line-buffered`:**
        * Por defecto, cuando `grep` detecta que su salida va hacia una tubería (`|`), espera a llenar un búfer de memoria (unos 4KB) antes de soltar los datos. Esto causa un retraso o "congelamiento" aparente.
        * Esta bandera fuerza a `grep` a imprimir cada línea en el momento exacto en que la encuentra, permitiendo el monitoreo en tiempo real.


    - **Encadenamiento de `grep`:**
        * Al usar varios `| grep`, actúan como un operador lógico **AND**. Solo se mostrarán las líneas que contengan estrictamente todos los términos buscados.



### Ejemplo de Salida de Logs

```
2026-01-08 18:12:45,816 1 DEBUG odoo odoo.addons.gestion_tareas_sergio.models.models: Código generado: SPRINT SEMANA 1_2 
2026-01-08 18:21:04,123 1 DEBUG odoo odoo.service.model: call gestion_tareas_sergio.tareas_sergio(4,).web_read(specification={'codigo': {}, 'name': {}, 'descripcion': {}, 'sprint': {'fields': {'display_name': {}}}, 'fecha_creacion': {}, 'fecha_ini': {}, 'fecha_fin': {}, 'finalizado': {}, 'rel_tecnologias': {'fields': {'display_name': {}}}, 'display_name': {}}) 
2026-01-08 18:21:04,129 1 DEBUG odoo odoo.addons.gestion_tareas_sergio.models.models: Código generado: SPRINT SEMANA 1_4 

```

!!!tip "Buenas Prácticas"

    **Usa Excepciones Específicas**

    ```python
    # Mal
    raise Exception("Error")

    # Bien
    raise ValidationError("La fecha de fin debe ser posterior a la fecha de inicio")
    ```

    **Mensajes Claros y Útiles**

    ```python
    # Mal
    raise ValidationError("Error en los datos")

    # Bien
    raise ValidationError(
        f"El sprint '{sprint.nombre}' ya ha finalizado. "
        "No se pueden añadir tareas a sprints finalizados."
    )
    ```

    **Registra Información Relevante**

    ```python
    # Registra antes de operaciones importantes
    _logger.info(f"Creando sprint: {sprint.nombre}, duración: {sprint.duracion} días")

    # Registra advertencias ante situaciones anormales
    if not tarea.descripcion:
        _logger.warning(f"Tarea {tarea.id} creada sin descripción")

    # Registra errores con contexto
    _logger.error(f"Fallo al calcular fecha_fin para sprint {sprint.id}: {str(e)}")
    ```

    **No Registres Información Sensible**

    ```python
    # Mal - expone contraseñas
    _logger.info(f"Usuario login: {user.login}, password: {user.password}")

    # Bien
    _logger.info(f"Usuario login exitoso: {user.login}")
    ```

    **Usa el Nivel Apropiado**

    ```python
    # DEBUG - solo en desarrollo
    _logger.debug(f"Valor de variable x: {x}")

    # INFO - eventos normales importantes
    _logger.info("Módulo inicializado correctamente")

    # WARNING - situación anormal pero no crítica
    _logger.warning("Base de datos sin datos de demo")

    # ERROR - operación fallida
    _logger.error("No se pudo conectar al servidor externo")
    ```

!!!example "Validaciones Comunes"

    **Validar Campos Obligatorios**

    ```python
    @api.constrains('nombre')
    def _check_nombre(self):
        for record in self:
            if not record.nombre or not record.nombre.strip():
                raise ValidationError("El nombre no puede estar vacío")
    ```

    **Validar Rangos Numéricos**

    ```python
    @api.constrains('duracion')
    def _check_duracion(self):
        for sprint in self:
            if sprint.duracion and (sprint.duracion < 1 or sprint.duracion > 30):
                raise ValidationError(
                    "La duración del sprint debe estar entre 1 y 30 días"
                )
    ```

    **Validar Unicidad**

    ```python
    @api.constrains('nombre')
    def _check_nombre_unico(self):
        for sprint in self:
            duplicado = self.search([
                ('nombre', '=', sprint.nombre),
                ('id', '!=', sprint.id)
            ])
            if duplicado:
                raise ValidationError(
                    f"Ya existe un sprint con el nombre '{sprint.nombre}'"
                )
    ```

---

## 🧩 Tu Turno: Gestor de Restaurante

Ahora aplicarás validaciones y logging al proyecto del restaurante.

### Objetivos y Contexto

Añadirás validaciones para asegurar la integridad de los datos y agregarás mensajes de log para facilitar la depuración y el seguimiento de operaciones.

Implementarás:
- Validaciones de campos obligatorios
- Validaciones de rangos numéricos
- Manejo de excepciones en campos computados
- Logging de operaciones importantes

### Pasos a Realizar

1. **Importar excepciones y logging**
    
    Al inicio de `models.py`:
    ```python
    from odoo.exceptions import ValidationError, UserError
    import logging
    
    _logger = logging.getLogger(__name__)
    ```

2. **Validar que el precio sea positivo**
    
    En el modelo Plato, crea un método con `@api.constrains('precio')` que:

    - Verifique que el precio sea mayor o igual a 0
    - Lance `ValidationError` si no cumple
    
    Pistas:

    - El mensaje debe ser claro para el usuario
    - Recuerda iterar sobre `self`

3. **Validar que el tiempo de preparación sea razonable**
    
    En el modelo Plato, valida que `tiempo_preparacion`:

    - Si existe, esté entre 1 y 240 minutos (4 horas)
    - Lance `ValidationError` con mensaje descriptivo

4. **Validar fechas del menú**
    
    En el modelo Menú, valida que:

    - Si existe `fecha_fin`, sea posterior a `fecha_inicio`
    - Use `@api.constrains` con ambos campos
    
    Pista: Compara fechas con operadores `<` y `>`

5. **Añadir manejo de excepciones al campo computado `codigo`**
    
    Modifica el método `_get_codigo` del modelo Plato:

    - Envuélvelo en un bloque `try-except`
    - Verifica que la categoría existe antes de usarla
    - Lanza `ValidationError` si algo falla
    
    Pista: Usa `if not plato.categoria` para verificar

6. **Añadir logging a operaciones importantes**
    
    En el modelo Plato, añade logs en momentos clave:

    - `_logger.debug()` al generar códigos
    - `_logger.info()` cuando se validan precios correctamente
    - `_logger.warning()` si un plato no tiene categoría
    - `_logger.error()` cuando falla una operación

7. **Validar que un menú tenga al menos un plato**
    
    En el modelo Menú, crea una validación que:

    - Verifique que el menú tenga al menos un plato cuando esté activo
    - Use `@api.constrains('platos', 'activo')`
    - Lance `ValidationError` si está activo sin platos
    
    Pista: Usa `len(menu.platos)` para contar platos

8. **Configurar nivel de log en debug**
    
    En `data/odoo_config/odoo.conf`, añade o modifica:
    ```ini
    log_level = debug
    ```

9. **Probar las validaciones**
    
    Intenta crear:

    - Un plato con precio negativo (debe fallar)
    - Un plato con tiempo de preparación de 300 minutos (debe fallar)
    - Un menú con fecha_fin anterior a fecha_inicio (debe fallar)
    - Un menú activo sin platos (debe fallar)

10. **Ver los logs generados**
    
    Ejecuta:
    ```bash
    docker logs odoo_dev_dam -f | grep "restaurante"
    ```
    
    Verifica que aparecen tus mensajes de log.

### Verificaciones y Resultado Esperado

Comprueba que:

- No puedes guardar un plato con precio negativo
- No puedes guardar un plato con tiempo de preparación fuera del rango
- No puedes guardar un menú con fechas inconsistentes
- No puedes activar un menú sin platos
- Los mensajes de error son claros y útiles para el usuario
- Los logs muestran información relevante de las operaciones

**Pruebas sugeridas**:

1. Intenta crear un plato con precio -5€ → *Debe mostrar error de validación*
2. Intenta crear un plato con tiempo de preparación de 500 minutos → *Error*
3. Crea un menú con fecha_fin antes de fecha_inicio → *Error*
4. Activa un menú vacío → *Error*
5. Verifica que en los logs aparece información de cada operación
