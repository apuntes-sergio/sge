---
title: Arquitectura de Odoo
description: Comprensión de la arquitectura y componentes fundamentales de Odoo
---

Antes de comenzar a desarrollar módulos en **Odoo**, es fundamental comprender su arquitectura para entender cómo funciona el sistema y cómo debemos programar utilizando el framework que nos proporciona.

El conocimiento de la arquitectura nos permitirá:

- Comprender el flujo de datos en el sistema
- Utilizar correctamente el framework de desarrollo
- Diseñar módulos eficientes y bien estructurados
- Diagnosticar y resolver problemas más fácilmente

## Visión General de la Arquitectura

### Arquitectura de Tenencia Múltiple

Odoo implementa una **arquitectura de tenencia múltiple** (Multi-Tenancy), lo que significa que:

- Un único servidor de Odoo puede servir a múltiples empresas/clientes
- Cada empresa tiene su propia base de datos independiente
- Los recursos del servidor (código, memoria) se comparten entre todas las empresas
- Cada base de datos está completamente aislada de las demás

```text
                    ┌─────────────────────────┐
                    │   Servidor Odoo         │
                    │   (Código compartido)   │
                    └───────────┬─────────────┘
                                │
                ┌───────────────┼───────────────┐
                │               │               │
          ┌─────▼──────┐   ┌────▼──────┐  ┌─────▼─────┐
          │ Empresa A  │   │ Empresa B │  │ Empresa C │
          │  (BD 1)    │   │  (BD 2)   │  │  (BD 3)   │
          └────────────┘   └───────────┘  └───────────┘
```

!!! info "Ventajas de Multi-Tenancy"
    - Menor coste de infraestructura
    - Actualizaciones centralizadas
    - Mantenimiento simplificado
    - Escalabilidad horizontal

### Arquitectura en Capas

Odoo está organizado en tres capas principales siguiendo el patrón **MVC (Modelo-Vista-Controlador)**:

```
          ┌─────────────────────────────────────────────────────┐
          │              CAPA DE PRESENTACIÓN                   │
          │        (Vista - Interfaz de Usuario)                │
          │  ┌──────────────────────────────────────────────┐   │
          │  │  Templates XML, QWeb, CSS, JavaScript (OWL)  │   │
          │  └──────────────────────────────────────────────┘   │
          └─────────────────────────┬───────────────────────────┘
                                    │
          ┌─────────────────────────▼───────────────────────────┐
          │           CAPA DE LÓGICA DE NEGOCIO                 │
          │            (Controlador - Servidor)                 │
          │  ┌──────────────────────────────────────────────┐   │
          │  │  Métodos Python, Business Logic, API, ORM    │   │
          │  └──────────────────────────────────────────────┘   │
          └─────────────────────────┬───────────────────────────┘
                                    │
          ┌─────────────────────────▼───────────────────────────┐
          │              CAPA DE DATOS                          │
          │           (Modelo - PostgreSQL)                     │
          │  ┌──────────────────────────────────────────────┐   │
          │  │        Tablas, Relaciones, Datos             │   │
          │  └──────────────────────────────────────────────┘   │
          └─────────────────────────────────────────────────────┘
```

Otra forma de ver esta arquitectura es la que representa la siguiente imagen:

<figure markdown="span" align="center">
  ![Image title](./imgs/desarrollo/Diagrama_MVC.jpg){ width="65%"  }
  <figcaption>Diagrama Modelo - Vista - Controlador en Odoo</figcaption>
</figure>

## Componentes Fundamentales

### 1. ORM (Object-Relational Mapping)

El **ORM** es el corazón de Odoo. Proporciona una capa de abstracción entre las clases Python y las tablas de PostgreSQL. Las **funciones principales** son 

- **Mapea automáticamente** clases Python → tablas PostgreSQL
- **Elimina SQL manual**: No necesitas escribir consultas SQL
- **Gestiona relaciones**: One2many, Many2one, Many2many automáticamente
- **Valida datos**: Aplica constraints y validaciones
- **Control de acceso**: Gestiona permisos a nivel de registro

<figure markdown="span" align="center">
  ![Image title](./imgs/odoo/odoo_ORM.png){ width="75%"  }
  <figcaption>Diagrama del ORM de Odoo</figcaption>
</figure>

!!!example "Ejemplo conceptual"

    ```python
    # En lugar de escribir SQL...
    SELECT * FROM sale_order WHERE state = 'draft'

    # Con el ORM de Odoo escribes...
    orders = self.env['sale.order'].search([('state', '=', 'draft')])
    ```

!!! tip "Ventajas del ORM"
    - Código más legible y mantenible
    - Prevención de inyección SQL
    - Gestión automática de transacciones
    - Cache inteligente de consultas
    - Compatibilidad entre versiones

> **Documentación**: [ORM Methods in Odoo](https://sdlccorp.com/post/an-overview-of-orm-methods-in-odoo-18/)

### 2. Modelo-Vista-Controlador (MVC)

#### Modelo (Model)

- **Definición**: Clases Python que representan entidades de negocio
- **Ubicación**: Archivos `.py` en la carpeta `models/`
- **Responsabilidad**: Estructura de datos + lógica de negocio
- **Persistencia**: El ORM mapea automáticamente a PostgreSQL

**Ejemplo**: Un modelo `sale.order` define qué es un pedido de venta, sus campos (cliente, líneas, total, etc.) y métodos (confirmar, cancelar, etc.)

#### Vista (View)

- **Definición**: Archivos XML que definen cómo se visualizan los datos
- **Ubicación**: Archivos `.xml` en la carpeta `views/`
- **Responsabilidad**: Presentación de la información al usuario
- **Tipos**: Formularios, listas, kanban, calendario, gráficos, pivote, etc.

**Ejemplo**: Una vista de formulario de `sale.order` define cómo se muestra el pedido en pantalla (campos visibles, orden, agrupaciones, etc.)

#### Controlador (Controller)

- **Definición**: Métodos Python que procesan la lógica de negocio
- **Ubicación**: Dentro de las clases de modelo o en `controllers/`
- **Responsabilidad**: Procesar acciones, validar datos, coordinar el flujo
- **Tipos**: Métodos de modelo, controladores web (HTTP)

**Ejemplo**: Un método `action_confirm()` en `sale.order` que valida y confirma el pedido

### 3. WSGI y Werkzeug

**WSGI** (Web Server Gateway Interface) es el estándar Python para comunicación entre servidores web y aplicaciones.

**Werkzeug** es la biblioteca que Odoo utiliza para:

- Recibir peticiones HTTP/HTTPS
- Enrutar las peticiones a los controladores correctos
- Generar las respuestas HTTP
- Gestionar sesiones y cookies

```
Cliente Web → HTTP Request → Werkzeug → Controlador Odoo → Respuesta
```

Aquí tienes un esquema **en texto plano (ASCII)** que explica la relación entre **WSGI**, **Werkzeug** y **Odoo**:

```text
          +------------------------------------------------------+
          |                      Odoo                            |
          |  (Aplicación Python para ERP)                        |
          +------------------------------------------------------+
                                │
                                │ usa
                                ▼
          +------------------------------------------------------+
          |                    WSGI                              |
          |  (Web Server Gateway Interface)                      |
          |  Estándar que conecta servidores web con apps        |
          |  Python. Define cómo se comunican.                   |
          +------------------------------------------------------+
                                │
                                │ implementado por
                                ▼
          +------------------------------------------------------+
          |                   Werkzeug                           |
          |  (Biblioteca WSGI en Python)                         |
          |  Proporciona utilidades para manejar peticiones,     |
          |  respuestas y middlewares.                           |
          +------------------------------------------------------+
```
Explicación rápida del esquema:

*   **Odoo** es la aplicación ERP escrita en Python.
*   Para servir contenido web, Odoo necesita hablar con un servidor (como Gunicorn, uWSGI).
*   **WSGI** es el protocolo estándar que define cómo se comunican aplicaciones Python con servidores web.
*   **Werkzeug** es una implementación de WSGI que Odoo utiliza internamente para gestionar peticiones HTTP y respuestas.




### 4. Business Objects

Los **Business Objects** son los modelos que contienen tanto datos como lógica de negocio.

**Características**:  

- Heredan de `models.Model`
- Combinan estructura (campos) y comportamiento (métodos)
- Representan entidades del mundo real (clientes, productos, facturas)
- Encapsulan la lógica relacionada con esa entidad

!!!example "Ejemplo conceptual"
    ```python
    class Cliente(models.Model):
        _name = 'res.partner'
        
        # DATOS (Estructura)
        name = fields.Char('Nombre')
        email = fields.Char('Email')
        
        # LÓGICA (Comportamiento)
        def enviar_bienvenida(self):
            # Lógica para enviar email de bienvenida
            pass
    ```

### 5. Wizards (Asistentes)

Los **Wizards** son formularios temporales que guían al usuario en procesos complejos paso a paso.

**Características**:
- Heredan de `models.TransientModel`
- No persisten permanentemente en la BD (se auto-eliminan)
- Simplifican tareas complejas dividiéndolas en pasos
- Suelen tener botones de "Siguiente", "Anterior", "Finalizar"

**Ejemplos de uso**:
- Asistente de facturación masiva
- Importador de datos CSV
- Configurador de productos con múltiples opciones
- Generador de informes personalizados

### 6. Cliente Web (OWL + Widgets)

El **cliente web** de Odoo está construido con tecnologías modernas:

- **OWL** (Odoo Web Library)

    - Framework JavaScript propio de Odoo (similar a React/Vue)
    - Basado en componentes
    - Reactivo y eficiente
    - Sistema de plantillas con QWeb


- **Widgets**

Los **Widgets** son componentes visuales que se adaptan al tipo de dato:

| Tipo de Campo | Widget | Comportamiento |
|---------------|--------|----------------|
| Date | Calendario | Muestra selector de fecha |
| Many2one | Select2 | Autocompletado con búsqueda |
| Boolean | Checkbox | Casilla de verificación |
| Selection | Dropdown | Lista desplegable |
| Image | Image | Visor/cargador de imágenes |
| HTML | HTML Editor | Editor de texto enriquecido |

!!! info "Personalización"
    Puedes crear **widgets personalizados** para comportamientos específicos de tu negocio

### Resumen de Componentes

| Componente | Descripción | Tecnología |
|------------|-------------|------------|
| **Base de Datos** | Almacenamiento persistente de datos | PostgreSQL |
| **ORM** | Mapeo objeto-relacional, abstrae SQL | Python |
| **Business Objects** | Modelos con datos + lógica de negocio | Python (models.Model) |
| **WSGI/Werkzeug** | Servidor HTTP, enrutamiento | Python/Werkzeug |
| **Wizards** | Asistentes temporales | Python (TransientModel) |
| **Cliente Web** | Interfaz de usuario interactiva | JavaScript (OWL) + QWeb |
| **Widgets** | Componentes visuales adaptables | JavaScript |

## La Base de Datos en Odoo

### Diseño Automático

En Odoo **no existe un diseño explícito de base de datos**. En su lugar:

- El desarrollador **define clases Python** (modelos)
- El **ORM mapea** automáticamente esas clases a tablas
- **PostgreSQL** almacena los datos resultantes

!!! warning "Sin diagramas ER"
    Odoo no proporciona diagramas Entidad-Relación tradicionales. La "base de datos" es el resultado del mapeo automático de los modelos.

### Nomenclatura de Tablas y Campos

#### Reglas de Nomenclatura

**Clases Python → Tablas PostgreSQL**

```python
# Clase Python en Odoo
class SaleOrder(models.Model):
    _name = 'sale.order'  # Nombre del modelo
```

Se mapea a:

```sql
-- Tabla en PostgreSQL
CREATE TABLE sale_order (...)
```

**Regla**: Los puntos (`.`) se convierten en guiones bajos (`_`)

**Atributos Python → Columnas PostgreSQL**

```python
class SaleOrder(models.Model):
    _name = 'sale.order'
    
    date_order = fields.Datetime('Fecha Pedido')
    amount_total = fields.Float('Total')
```

Se mapea a:

```sql
CREATE TABLE sale_order (
    date_order TIMESTAMP,
    amount_total NUMERIC,
    ...
)
```

**Regla**: Los nombres de atributos Python se mantienen idénticos como nombres de columna

#### Ejemplos de Nomenclatura

| Modelo Python | Tabla PostgreSQL | Descripción |
|---------------|------------------|-------------|
| `res.partner` | `res_partner` | Contactos (clientes/proveedores) |
| `sale.order` | `sale_order` | Pedidos de venta |
| `sale.order.line` | `sale_order_line` | Líneas de pedido |
| `account.move` | `account_move` | Asientos contables |
| `product.product` | `product_product` | Productos |

### Inspección desde la Interfaz Web

Odoo permite **descubrir el nombre técnico** de modelos y campos desde la interfaz:

1. **Activar Modo Desarrollador**: 
    - Configuración → Activar modo desarrollador
    - O añadir `?debug=1` a la URL

2. **Ver información técnica**:
    - Al pasar el ratón sobre campos, aparece el nombre técnico
    - Menú "Ver metadatos" muestra modelo y campos
    - Vista de formulario muestra la estructura XML

!!! tip "Ejemplo práctico"
    Si en un formulario de cliente ves el campo "Nombre" y activas el modo desarrollador, verás que:
    - Modelo: `res.partner`
    - Campo: `name`
    - Tabla: `res_partner`
    - Columna: `name`

## Estructura de Carpetas de Odoo

```
odoo/
├── addons/              # Módulos oficiales de Odoo
│   ├── sale/
│   ├── purchase/
│   ├── account/
│   └── ...
├── odoo/                # Núcleo del framework
│   ├── models/          # Definición del ORM
│   ├── addons/          # Módulos integrados en el core
│   ├── tools/           # Utilidades y helpers
│   └── http.py          # Gestión de peticiones HTTP
├── config/              # Archivos de configuración
│   └── odoo.conf
├── data/                # Datos persistentes (filestore)
├── setup/               # Scripts de instalación
├── tests/               # Tests automatizados
├── doc/                 # Documentación técnica
├── static/              # Archivos estáticos (CSS, JS, imágenes)
└── web/                 # Módulo de interfaz web
```

### `addons/` - Módulos Oficiales

Contiene todos los módulos estándar de Odoo:

- Cada módulo en su propia carpeta
- Estructura completa: models, views, controllers, data, security
- Ejemplos: `sale`, `account`, `crm`, `inventory`, `hr`

**Uso**: Estos módulos se instalan desde la interfaz de Odoo

### `odoo/` - Núcleo del Framework

El corazón de Odoo con todos los componentes esenciales:

**Subcarpetas importantes**:

- `odoo/models/`: Clases base del ORM (`Model`, `TransientModel`, `AbstractModel`)
- `odoo/fields/`: Definición de tipos de campos
- `odoo/tools/`: Funciones auxiliares (validaciones, conversiones, etc.)
- `odoo/http.py`: Manejo de peticiones HTTP y controladores web
- `odoo/api.py`: Decoradores y API del ORM

**Uso**: Los desarrolladores rara vez modifican esta carpeta directamente

### `config/` - Configuración

Archivos de configuración del servidor:

- `odoo.conf`: Configuración principal (puertos, BD, addons_path, workers, etc.)
- Parámetros de conexión a PostgreSQL
- Rutas de módulos personalizados

**Uso**: Editar para personalizar el comportamiento del servidor

### `data/` - Datos Persistentes

Almacenamiento de archivos subidos por usuarios:

- Imágenes de productos
- Documentos adjuntos
- Archivos de importación/exportación
- Avatar de usuarios

**Uso**: Backup regular de esta carpeta es crucial

### `custom_addons/` - Módulos Personalizados

Esta carpeta **no existe por defecto**, pero es una convención común:

- Separar módulos personalizados de los oficiales
- Facilitar actualizaciones de Odoo sin perder cambios
- Mejor organización del código

**Configuración en `odoo.conf`**:
```ini
addons_path = /mnt/custom_addons,/usr/lib/python3/dist-packages/odoo/addons
```

### Relación con Docker Compose

Las carpetas que mapeamos en nuestro `docker-compose.yml`:

```yaml
volumes:
  - ./data/addons:/mnt/extra-addons                 # ← Nuestros módulos personalizados
  - ./data/odoo_config:/etc/odoo                    # ← Archivo odoo.conf
  - ./data/odoo/filestore:/var/lib/odoo/filestore   # ← Archivos subidos
  - ./data/odoo/sessions:/var/lib/odoo/sessions     # ← Sesiones de usuario
```

**Correspondencia**:

| Volumen Docker | Ruta en Contenedor | Propósito |
|----------------|-------------------|-----------|
| `./data/addons` | `/mnt/extra-addons` | Módulos personalizados |
| `./data/odoo_config` | `/etc/odoo` | Configuración (odoo.conf) |
| `./data/odoo/filestore` | `/var/lib/odoo/filestore` | Archivos adjuntos |
| `./data/odoo/sessions` | `/var/lib/odoo/sessions` | Sesiones web |

!!! success "Ventaja de los volúmenes"
    Al mapear estas carpetas, nuestros cambios persisten incluso si eliminamos los contenedores

## Recursos y Referencias

Documentación Oficial:

- [Odoo Developer Documentation](https://www.odoo.com/documentation/19.0/developer.html)
- [ORM API Reference](https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html)
- [Views Reference](https://www.odoo.com/documentation/19.0/developer/reference/backend/views.html)
- [Controllers & Routing](https://www.odoo.com/documentation/19.0/developer/reference/backend/http.html)

Tutoriales y Guías:

- [An Overview of ORM Methods in Odoo](https://sdlccorp.com/post/an-overview-of-orm-methods-in-odoo-18/)
- [Odoo Module Manifest Reference](https://www.odoo.com/documentation/8.0/reference/module.html#reference-module-manifest)

