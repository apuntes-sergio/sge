---
title: Proyecto final Odoo. 'DRONIFY'
description: DRONIFY - SISTEMA DE GESTIÓN DE LOGÍSTICA CON DRONES
---


## 1. Introducción y Contextualización del Proyecto

**Dronify** es una empresa tecnológica puntera especializada en la **logística aeroespacial automatizada**. Su modelo de negocio se centra en revolucionar el transporte de mercancías mediante el uso de una flota de drones inteligentes, permitiendo realizar entregas rápidas y seguras en entornos donde el transporte terrestre es ineficiente.

La empresa opera como una plataforma integral de gestión que coordina cuatro pilares fundamentales:

* **Gestión de Flota Avanzada**: Controla el estado técnico, la capacidad de carga y los niveles de energía de cada dron en tiempo real.
* **Base de Datos de Capital Humano**: Administra un registro de clientes (incluyendo servicios preferenciales para clientes VIP) y una red de pilotos profesionales sujetos a certificaciones legales estrictas.
* **Procesamiento de Carga**: Gestiona la recepción de paquetes, su pesaje y la trazabilidad de los mismos desde el almacén hasta el destino final.
* **Planificación de Vuelos Inteligente**: Coordina la asignación de misiones de vuelo, realizando validaciones automáticas de seguridad que aseguran que el peso de la carga y la autonomía de la batería sean óptimos para cada trayecto.

---

## 2. Modelado de Datos y Relaciones

El sistema se basa en **cuatro modelos principales**, incluyendo la extensión del modelo de contactos de Odoo para la gestión tanto de pilotos como de clientes.

**Entrega Obligatoria Previa: Diagrama Entidad-Relación**: Antes de comenzar el desarrollo, debes diseñar y entregar un **Diagrama Entidad-Relación (E-R)** que incluya:

- Los 4 modelos del sistema
- Todas las relaciones entre ellos, indicando su tipo (Many2one, One2many, Many2many)
- Los campos principales de cada modelo
- Las cardinalidades de las relaciones


### Contactos: Pilotos y Clientes (`res.partner`)

Extiende el modelo de contactos de Odoo para gestionar tanto pilotos como clientes del servicio de drones. Se debes heredar el modelo `res.partner` **UNA SOLA VEZ**, añadiendo todos los campos necesarios. Posteriormente se crearán **vistas separadas** para gestionar pilotos y clientes de forma independiente.

Campos añadidos:

| Campo | Tipo | Propósito |
|-------|------|-----------|
| `es_cliente` | Boolean | Identifica si el contacto es cliente |
| `es_vip` | Boolean | Marca clientes premium (activa modo ahorro en vuelos) |
| `es_piloto` | Boolean | Identifica si el contacto es piloto |
| `licencia` | Char | Número de licencia del piloto (obligatorio solo para pilotos) |
| `dron_autorizado_ids` | Many2many | Lista de drones que el piloto está certificado para operar |

Consideraciones importantes:

- Un contacto puede ser piloto, cliente, o ambos simultáneamente
- La relación Many2many con drones debe ser **bidireccional**
- El campo `licencia` debe ser obligatorio únicamente cuando `es_piloto=True`


### Gestión de Flota: Drones (`dronify.dron`)

Representar los activos físicos de la empresa y su estado operativo.

Campos del modelo:

| Campo | Tipo | Propósito | Características especiales |
|-------|------|-----------|-------|
| `name` | Char | Nombre identificativo del dron | |
| `capacidad_max` | Float | Carga máxima en kilogramos | **Obligatorio**|
| `bateria` | Integer | Nivel de carga actual (0-100%) | **Defecto: 100** |
| `estado` | Selection | Estado operativo del dron | Valores: `disponible` (por **defecto**), `vuelo`, `taller` |
| `piloto_autorizado_ids` | Many2many | Pilotos certificados para este dron (relación inversa) | |


### Mercancía: Paquetes (`dronify.paquete`)

Gestiona los paquetes a transportar y su trazabilidad.

| Campo | Tipo | Propósito | Características especiales |
|-------|------|-----------|---------------------------|
| `codigo` | Char | Identificador único | Generado automáticamente, formato: `YYYYMMDDHHMMSS`. **Solo lectura** |
| `name` | Char | Descripción del contenido | **Obligatorio** |
| `peso` | Float | Peso en kilogramos | **Obligatorio** |
| `cliente_id` | Many2one | Cliente que envía el paquete | Solo contactos con `es_cliente=True` **Obligatorio**|
| `vuelo_id` | Many2one | Vuelo asignado | Se asigna desde el modelo Vuelo. **Solo lectura** |
| `dron_relacionado` | Char | Nombre del dron del vuelo | Campo Related, solo lectura **Solo lectura** |


### Operaciones: Registro de Vuelos (`dronify.vuelo`)

Coordina la planificación y ejecución de las misiones de entrega.

| Campo | Tipo | Propósito | Características |
|-------|------|-----------|-----------------|
| `codigo` | Char | Código único del vuelo | Generado automáticamente, formato: `YYMMDDHHMMSS` *Solo lectura** |
| `name` | Char | Denominación de la misión | Valor por defecto: `YYYYMMDD_Vuelo` **Obligatorio** |
| `dron_id` | Many2one | Dron asignado | **Obligatorio** |
| `piloto_id` | Many2one | Piloto responsable | **Obligatorio**, solo pilotos |
| `paquetes_ids` | One2many | Paquetes a transportar | - |
| `preparado` | Boolean | Indica si el vuelo está listo para ejecutarse | - |
| `realizado` | Boolean | Indica si el vuelo se ha completado | - |
| `peso_total` | Float | Suma del peso de todos los paquetes asignados | **Campo computado** |
| `consumo_estimado` | Float | Porcentaje de batería que consumirá el vuelo | **Campo computado** |

Apuntes:

- Deben almacenarse en la base de datos (`store=True`)
- Deben recalcularse automáticamente cuando cambien los paquetes o sus datos
- El `consumo_estimado` se calcula mediante función del fichero proporcinado `logica_dronify.py`
- Los campos computados se pueden calcula con funciones `lambda`

---

## 3. Vistas y Experiencia de Usuario (UI)

Las vistas que se describen a continuación deben ser definidas explícitamente. Las vistas no mencionadas pueden generarse automáticamente por Odoo.

### Vistas de Vuelos (`dronify.vuelo`)

**Vista de Listado**

- Debe mostrar una tabla con la información esencial de cada vuelo.
- Campos: codigo, name, dron_id, piloto_id, peso_total, consumo_estimado, preparado, realizado
- Requisitos visuales:
    - El campo `consumo_estimado` debe mostrar el símbolo '%'
    - Los campos `preparado` y `realizado` deben usar el widget boolean y estar centrados


**Vista de Formulario**

Centro de control operativo del vuelo.

- **Header (zona de botones)**:
    - Tres botones que ejecuten las acciones: `action_preparar_vuelo`, `action_desbloquear`, `action_finalizar_vuelo`
    - El botón "Preparar Vuelo" debe tener estilo destacado
    - Cada botón debe mostrarse u ocultarse según el estado actual del vuelo
    - Utilizar IA para ver cómo se define un `Button` y se asigna una *función* 

- **Cuerpo del formulario**:
    - Organizar campos en grupos lógicos: "Planificación" y "Estado y Control"
    - Los campos de planificación (name, dron_id, piloto_id) deben bloquearse cuando el vuelo esté preparado
    - Los campos `preparado` y `realizado` deben ser siempre de solo lectura

- **Pestaña de Paquetes**:
    - Lista de paquetes asignados (bloqueada cuando el vuelo esté preparado)
    - Al pie de la lista, mostrar los totales: peso_total y consumo_estimado


### Vistas de Drones (`dronify.dron`)

**Vista de Listado**

- Monitoreo del estado de la flota.
- Campos: name, capacidad_max, bateria, estado
- Requisitos visuales especiales:
    - Las filas deben marcarse en **rojo** si la batería está por debajo del 25%
    - El campo `bateria` debe mostrarse con el **widget progressbar**
    - El campo `estado` debe usar el **widget badge** con colores según el valor


### Vistas de Paquetes (`dronify.paquete`)

**Vista de Listado**

- Trazabilidad simple de la mercancía. No se modifica nada y **no hay vista de formulario**
- Campos: codigo, name, peso, cliente_id, vuelo_id, dron_relacionado


### Vistas de Pilotos y Clientes (`res.partner`)

Crear **dos vistas independientes** de formulario para el mismo modelo, una para clientes y otra para pilotos, diferenciados mediante filtros.

#### Configuración para Pilotos:

- Mostrar solo contactos con `es_piloto=True`
- Al crear nuevo piloto, marcar automáticamente `es_piloto=True`
- Ocultar: campo "Puesto de trabajo", campo "Website", pestañas "Contactos", "Ventas y Compras" e "Internal Notes"
- Añadir sección visible con los campos: es_piloto, es_cliente, licencia, dron_autorizado_ids, es_vip
- Lógica de visibilidad:
    - `es_piloto`: readonly (siempre True)
    - `licencia`: visible siempre, obligatorio solo si es piloto
    - `dron_autorizado_ids`: usar widget many2many_tags
    - `es_cliente`: editable (puede ser piloto y cliente a la vez)
    - `es_vip`: visible solo si es cliente


#### Configuración para Clientes:

- Mostrar solo contactos con `es_cliente=True`
- Al crear nuevo cliente, marcar automáticamente `es_cliente=True`
- Ocultar los mismos campos que en Pilotos
- Misma estructura que pilotos pero con lógica inversa:
    - `es_cliente`: readonly (siempre True)
    - `es_vip`: visible solo si es cliente
    - `es_piloto`: editable (puede ser cliente y piloto a la vez)
    - `licencia`: visible solo si es piloto, obligatorio en ese caso

---

## 4. Lógica de Funcionamiento y Procesos

### Módulo proporcionado `logica_dronify.py`

Para separar la lógica de cálculo en un módulo independiente y fomentar buenas prácticas de código modular, se proporciona este módulo a modo de *caja negra* que contiene las siguientes funciones

- `calcular_consumo_vuelo(peso_total, es_vip)`
    - Calcular el porcentaje de batería que consumirá un vuelo.
    - Devuelve gloat
    - Se pasa el `peso_total` (float) y si `es_vip` (bool) el cliente para su cálculo

- `validar_estado_bateria(bateria_actual, consumo_estimado)`
    - Verificar si el dron tiene batería suficiente para completar el vuelo. Fr
    - Retorno Boolean
    - Parámetros la `bateria_actual` y el `consumo_estimado`.


### Campos Computados y Automatismos

- `peso_total`
    - Debe sumar automáticamente el peso de todos los paquetes vinculados
    - Debe recalcularse cuando se añadan, eliminen o modifiquen paquetes
    - Debe almacenarse en la base de datos

- `consumo_estimado`
    - Debe invocar la función `calcular_consumo_vuelo()` del módulo externo
    - Debe recalcularse cuando cambien los paquetes o sus clientes
    - Debe verificar si alguno de los clientes de los paquetes es VIP
    - Debe almacenarse en la base de datos
    - *Pista*: Usa el método `mapped()` para obtener la lista de clientes y verificar el campo `es_vip`.


### Flujo de Estados y Validaciones

El ciclo de vida de un vuelo consta de **tres estados**:

```
[EDICIÓN] → [PREPARADO] → [REALIZADO]
```

- Estado 1: EDICIÓN
    - Estado inicial al crear el vuelo
    - Todos los campos son modificables
    - El dron asociado permanece en estado 'disponible'
- Estado 2: PREPARADO
    - Se activa mediante el botón "Preparar Vuelo"
    - Los campos de planificación quedan bloqueados
    - El dron cambia a estado 'vuelo'
    - Se puede volver a EDICIÓN mediante el botón "Modificar Datos" (si no se ha realizado)
- Estado 3: REALIZADO
    - Se activa mediante el botón "Marcar como Realizado"
    - Es irreversible
    - Se descuenta el consumo de la batería del dron
    - El dron vuelve a estado 'disponible'

Para conseguir la gestión de estos estados, se utilizarán ĺos `Buttons` indicados y se deben implementar **tres métodos** que se ejecutarán al pulsar los botones del formulario:

- `action_preparar_vuelo`
    - Validar que se cumplen todos los requisitos de seguridad (ver lista abajo)
    - Si las validaciones pasan: marcar `preparado=True`
    - Cambiar el estado del dron a 'vuelo'

- `action_desbloquear`
    - Verificar que el vuelo NO ha sido realizado (si lo está, lanzar error)
    - Marcar `preparado=False` para permitir edición

- `action_finalizar_vuelo`
    - Verificar que el vuelo está preparado (si no, lanzar error)
    - Marcar `realizado=True`
    - Descontar el `consumo_estimado` de la batería actual del dron
    - Devolver el dron a estado 'disponible'

**Para que un vuelo pueda ser preparado**, debe cumplir todas estas condiciones:

1. **Asignaciones básicas**: Debe tener dron y piloto asignados
2. **Existencia de carga**: Debe tener al menos un paquete asignado
3. **Capacidad de carga**: El peso total no puede superar la capacidad máxima del dron
4. **Disponibilidad del dron**: El dron debe estar en estado 'disponible'
5. **Batería suficiente**: La batería actual del dron debe ser mayor o igual al consumo estimado
6. **Certificación del piloto**: El piloto debe tener el dron asignado en su lista de "Drones Autorizados"

Estas validaciones deben implementarse en un método con el decorador `@api.constrains` que se ejecute cuando el campo `preparado` cambie a `True`.Si alguna validación falla, debes lanzar una `ValidationError` con un mensaje descriptivo del problema.

---

## 5. Entrega y Evaluación

Aspectos Generales de la Aplicación

- **Nombre en `__manifest__.py`**: Debe seguir el formato: **"Dronify - [Tu Nombre]"**
- **Identificación en pantalla**: El nombre de la aplicación debe coincidir con el del manifest
- **Logotipo**: La aplicación debe tener un icono personalizado
- **Calidad del código**: 
    - Código ordenado y bien estructurado
    - Comentarios que identifiquen las secciones principales
    - Nombres de variables descriptivos

Control de Versiones (GitHub)

- Se deben realizar commits progresivos del desarrollo.
- **Mínimo 1 commit al finalizar cada sesión de clase**
- Cada commit debe tener un mensaje descriptivo de los cambios realizados
- Se debe completar el fichero `README.md` a modo de diario con el trabajo realizado cada día


**Entregas Obligatorias**

- Primera Sesión:
    - Diagrama Entidad-Relación*
        - Debe mostrar los 4 modelos y sus relaciones
        - Debe incluir los campos principales de cada modelo
        - Debe ser validado antes de continuar con el código
- Durante el Desarrollo:
    - Commits diarios en GitHub

--- 
- **Entrega final Domingo 15 de Febrero de 2026**
- **Presentación y defensa en clase: Martes 17 de Febrero de 2026**
---