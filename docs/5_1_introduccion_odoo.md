---
title: Introducción a Odoo.
description: Introducción al ecosistema de Odoo
---


## Odoo

Odoo es un **sistema ERP (Enterprise Resource Planning)** de código abierto que ofrece una suite completa de aplicaciones empresariales integradas. Desarrollado originalmente en 2005 bajo el nombre de TinyERP, posteriormente renombrado a OpenERP y finalmente a Odoo en 2014, se ha convertido en uno de los ERP más populares del mundo.

**Filosofía y Enfoque** : Odoo se distingue por su filosofía de **"todo en uno"**, ofreciendo una solución integral donde todos los módulos están perfectamente integrados y comparten la misma base de datos. Esto elimina los problemas típicos de sincronización entre diferentes sistemas que muchas empresas enfrentan.

!!! info "Datos relevantes"
    - Más de 7 millones de usuarios en todo el mundo
    - Disponible en más de 80 idiomas
    - Más de 30.000 aplicaciones en el Odoo App Store
    - Comunidad activa con miles de desarrolladores

## Características Principales

### 1. Modularidad

Odoo está construido sobre una **arquitectura modular** que permite a las empresas instalar únicamente las aplicaciones que necesitan. Esta modularidad facilita:

- Implementación progresiva según las necesidades del negocio
- Escalabilidad horizontal añadiendo módulos cuando sea necesario
- Personalización sin afectar al núcleo del sistema
- Mantenimiento simplificado de funcionalidades específicas

### 2. Código Abierto

Disponible bajo licencia **LGPL v3** para la versión Community y con licencia propietaria para la versión Enterprise. Esto proporciona:

- Transparencia total del código fuente
- Posibilidad de personalización sin restricciones
- Comunidad activa que contribuye constantemente
- Reducción de costes en licencias
- Independencia del proveedor

### 3. Tecnología Web Moderna

Construido con tecnologías actuales y robustas:

- **Backend**: Python (framework propietario de Odoo)
- **Frontend**: JavaScript (OWL Framework - Odoo Web Library)
- **Base de datos**: PostgreSQL
- **Arquitectura**: MVC (Modelo-Vista-Controlador)
- Interfaz responsive y moderna

### 4. Integración Total

Todos los módulos comparten:

- Una única base de datos
- Misma interfaz de usuario
- Flujos de trabajo interconectados
- Información en tiempo real entre módulos

## Ventajas de Odoo

### Flexibilidad y Personalización

La arquitectura de Odoo permite personalizar cualquier aspecto del sistema sin modificar el código base, mediante el uso de **módulos de extensión** que heredan y modifican funcionalidades existentes.

### Interfaz Intuitiva

La interfaz de usuario está diseñada pensando en la **experiencia del usuario**:

- Diseño limpio y moderno
- Navegación intuitiva
- Vistas adaptadas a cada tipo de dato (lista, formulario, kanban, calendario, etc.)
- Responsive design para uso en móviles y tablets

### Coste-Efectividad

- Versión Community completamente gratuita
- Versión Enterprise con precio por usuario competitivo
- Sin costes ocultos de infraestructura
- Posibilidad de hosting propio o en la nube

### Escalabilidad

Odoo crece con tu negocio:

- Desde pequeñas empresas hasta grandes corporaciones
- Desde un único módulo hasta implementaciones completas
- Soporta múltiples empresas y almacenes
- Infraestructura preparada para alto rendimiento

### Comunidad y Soporte

- Amplia documentación oficial
- Foros activos y comunidad de desarrolladores
- Odoo Experience (conferencia anual)
- Soporte profesional disponible
- Miles de partners certificados

## Módulos Principales de Odoo

### Módulos de Gestión Comercial

#### **Ventas (Sales)**

El módulo de ventas gestiona todo el ciclo comercial:

- Creación y seguimiento de presupuestos
- Conversión automática a pedidos de venta
- Gestión de productos y listas de precios
- Portal del cliente para seguimiento online
- Firma electrónica de documentos
- Integración con CRM, inventario y facturación

<figure markdown="span" align="center">
  ![Image title](./imgs/odoo/PresupuestoVenta.png){ width="85%"  }
  <figcaption>Módulo de Ventas. Presupuesto.</figcaption>
</figure>

#### **CRM (Customer Relationship Management)**

Gestión completa de la relación con clientes:

- Pipeline visual de oportunidades (kanban)
- Seguimiento de leads y prospección
- Gestión de actividades y llamadas
- Automatización de acciones
- Análisis de rendimiento del equipo comercial
- Integración con email y calendario

#### **Compras (Purchase)**

Control total del proceso de aprovisionamiento:

- Solicitudes de presupuesto a proveedores
- Gestión de órdenes de compra
- Control de recepciones
- Gestión de facturación de proveedores
- Análisis de rendimiento de proveedores
- Acuerdos de compra y contratos

### Módulos de Gestión Financiera

#### **Contabilidad (Accounting)**

Sistema contable completo y flexible:

- Contabilidad general y analítica
- Gestión de clientes y proveedores
- Conciliación bancaria automatizada
- Multi-moneda y multi-compañía
- Informes financieros personalizables
- Cumplimiento normativo por países
- Gestión de activos fijos

<figure markdown="span" align="center">
  ![Image title](./imgs/odoo/ContabilidadOdooCuentadeResultados.png){ width="85%"  }
  <figcaption>Contabilidad en Odoo. Cuenta de resultados.</figcaption>
</figure>


#### **Facturación (Invoicing)**

Emisión y gestión de facturas:

- Facturación automática desde ventas o compras
- Plantillas personalizables
- Gestión de pagos y cobros
- Facturas recurrentes (suscripciones)
- Portal del cliente
- Seguimiento de estado de facturas

### Módulos de Inventario y Logística

#### **Inventario (Inventory)**

Gestión avanzada de stock:

- Control de stock en tiempo real
- Múltiples almacenes y ubicaciones
- Trazabilidad completa (números de serie/lote)
- Reglas de reabastecimiento automático
- Valoración de inventario (FIFO, LIFO, Promedio)
- Operaciones de picking, packing y shipping
- Gestión de rutas logísticas

#### **Fabricación (Manufacturing)**

MRP completo para empresas productoras:

- Listas de materiales (BoM) multinivel
- Órdenes de fabricación
- Planificación de la producción
- Control de calidad
- Mantenimiento de equipos
- Gestión de centros de trabajo
- Subcontratación

### Módulos de Recursos Humanos

#### **Empleados (Employees)**

Gestión del personal:

- Ficha completa de empleados
- Organigrama de la empresa
- Gestión documental
- Evaluaciones de desempeño
- Directorio corporativo

#### **Ausencias (Time Off)**

Control de vacaciones y ausencias:

- Tipos de ausencias configurables
- Solicitudes y aprobaciones
- Calendario de ausencias
- Acumulación automática de días
- Informes de absentismo

#### **Asistencia (Attendance)**

Registro de jornada laboral:

- Check-in/Check-out desde kiosko o app
- Control de horas trabajadas
- Gestión de horas extras
- Integración con nóminas

### Módulos de Gestión de Proyectos

#### **Proyectos (Project)**

Gestión completa de proyectos:

- Proyectos y tareas con subtareas
- Vistas kanban, lista, gantt y calendario
- Asignación de recursos
- Seguimiento de tiempos
- Automatización de flujos de trabajo
- Portal del cliente para seguimiento
- Análisis de rentabilidad

<figure markdown="span" align="center">
  ![Image title](./imgs/odoo/ProyectoKanban.png){ width="85%"  }
  <figcaption>Vista kanban de un proyecto con diferentes etapas y tareas.</figcaption>
</figure>

!!! tip "Imagen sugerida"
    

#### **Partes de Horas (Timesheet)**

Control de imputación de tiempos:

- Registro de horas por proyecto/tarea
- Integración con facturación
- Análisis de productividad
- Facturación basada en tiempo
- Comparación planificado vs. real

### Módulos de Productividad

#### **Documentos (Documents)**

Gestor documental integrado:

- Organización por carpetas y etiquetas
- Automatización de flujos documentales
- OCR para procesamiento automático
- Versionado de documentos
- Compartición segura

#### **Sitio Web (Website)**

Constructor de sitios web integrado:

- Editor drag & drop
- Responsive design automático
- SEO optimizado
- Integración con eCommerce
- Blog y gestión de contenidos
- Formularios personalizables

#### **eCommerce**

Tienda online integrada:

- Catálogo de productos sincronizado
- Carrito de compra y checkout
- Múltiples métodos de pago
- Gestión de envíos
- Cross-selling y up-selling
- Sincronización total con inventario y contabilidad

### Módulos de Comunicación

#### **Conversaciones (Discuss)**

Sistema de mensajería interna:

- Chat individual y grupal
- Canales públicos y privados
- Videoconferencias integradas
- Historial de conversaciones
- Notificaciones inteligentes
- Integración con email

#### **Calendario (Calendar)**

Gestión de agenda:

- Calendarios personales y compartidos
- Sincronización con Google Calendar
- Invitaciones a reuniones
- Recordatorios automáticos
- Vista de disponibilidad del equipo

## Arquitectura de Odoo

### Estructura de Tres Capas

```
┌─────────────────────────────────┐
│     Capa de Presentación        │
│   (Cliente Web - JavaScript)    │
└─────────────────────────────────┘
              ↕
┌─────────────────────────────────┐
│      Capa de Negocio            │
│      (Servidor - Python)         │
└─────────────────────────────────┘
              ↕
┌─────────────────────────────────┐
│      Capa de Datos              │
│     (PostgreSQL Database)        │
└─────────────────────────────────┘
```

### Componentes Clave

**ORM (Object-Relational Mapping)**

- Abstracción de la base de datos
- Definición de modelos mediante clases Python
- Gestión automática de relaciones entre modelos
- Validaciones y constraints

**Motor de Vistas**

- XML para definición de vistas
- Múltiples tipos: form, tree, kanban, calendar, pivot, graph
- QWeb para templates
- Herencia de vistas para personalización

**Sistema de Permisos**

- Control de acceso basado en roles
- Reglas de registro (record rules)
- Permisos a nivel de campo
- Multi-compañía

## Ediciones de Odoo

### Community Edition

- **Licencia**: LGPL v3 (código abierto)
- **Coste**: Gratuito
- **Características**:
    - Módulos base completos
    - Auto-hosting requerido
    - Soporte de la comunidad
    - Ideal para pequeñas empresas o desarrollo

### Enterprise Edition

- **Licencia**: Propietaria (Odoo S.A.)
- **Coste**: Por usuario/mes (aprox. 20-30€)
- **Características adicionales**:
    - Módulos exclusivos (Estudio, Firma, IoT, etc.)
    - Interfaz mejorada
    - Hosting en Odoo.sh opcional
    - Soporte oficial
    - Actualizaciones automáticas

!!! warning "Diferencias clave"
    La versión Enterprise incluye módulos exclusivos y mejoras de interfaz, pero ambas versiones comparten el núcleo funcional. Para el desarrollo de módulos personalizados, las diferencias son mínimas.

## Casos de Uso

Odoo es versátil y se adapta a múltiples sectores:

- **Comercio**: Tiendas online, retail, distribución
- **Servicios**: Consultorías, agencias, estudios profesionales
- **Industria**: Manufactura, logística, producción
- **Educación**: Centros educativos, academias
- **Salud**: Clínicas, centros médicos
- **Hostelería**: Restaurantes, hoteles
- **Organizaciones**: ONGs, asociaciones













## Instalar `Odoo`

`Odoo` puede instalarse en cualquier sistema operativo. Sin embargo, se desarrolla pensando en Ubuntu o Debian y es el sistema en el que vamos a trabajar.

`Odoo`, en esencia, es un servidor web hecho en Python que se conecta con una base de datos PostgreSQL. Hay muchas maneras de instalar `Odoo`, desde las más avanzadas, que son descargar por *git* el repositorio y hacer que arranque al inicio, hasta las más simples, que son desplegar un **docker** con todo funcionando.


### Opciones para la instalación de `Odoo`

Cada empresa tiene unas necesidades y cada necesidad se puede cubrir de diferente forma.

Para el caso de ***Odoo*** tenemos diferentes formas de instalarlo, o en un servidor dedicado o mediante virtualización o utilizando las diferentes opciones que nos encontramos en la nube.

Vamos a hacer una comparativa no rigurosa de las distintas opciones:

| **Lugar** | **Tecnología** | **Propósito** |
| --- | --- | --- |
| **Servidor local** | Directamente instalado en Ubuntu Server | En función de la potencia, capacidad y seguridad del servidor, puede servir para cualquier empresa. Instalar directamente da menos flexibilidad, pero aprovecha toda la potencia de la máquina y la compatibilidad con todo. La empresa tiene control total de los datos y es responsable de la seguridad. También controla los gastos. Es poco escalable y migrable. |
| **Servidor local** | En Docker, Proxmox o VirtualBox  | Igual que la opción anterior, pero con más posibilidades de ser escalable y migrable. Permite compartir mejor los recursos de la máquina con otros servicios. |
| **En la nube**     | VPS | No es necesario pensar en la seguridad física, pero sí en la lógica. Es escalable si el proveedor permite ampliar la máquina. El precio suele estar predeterminado, pero a la larga es más caro que los servidores propios. En el caso de proveedores principales como AWS, Azure, Google Cloud... el precio es por uso y es peligroso no controlarlo. |
| **En la nube**     | SAAS (Odoo.sh) | Ya no es necesario preocuparse tanto por la seguridad lógica, solo por los usuarios de Odoo. Es escalable, pero mucho más caro. No es personalizable.  |
| **En la nube**     | PAAS (Render, Railway, Fly.io...)      | Cada servicio tiene sus características, pero combinan las ventajas de Docker en cuanto a personalización con una interfaz muy cómoda para DevOps y CI/CD. El despliegue se trata como si fuera código. |

La respuesta sigue sin ser clara. Cada empresa tiene sus posibilidades y necesidades. 

Una microempresa con pocos ordenadores y solo necesidad de intranet puede instalar en un Docker en un servidor de bajo consumo con copias de seguridad periódicas. Una empresa pequeña puede desplegar en VPS o PAAS con precios predefinidos y controlados e ir aumentando según lo necesite. 

Una empresa más grande puede optar por nubes más de bajo nivel (IAAS) o por una instalación on-premise más seria con alta disponibilidad en la propia empresa. 

Un SAAS puede ser útil para empresas que no necesitan ninguna personalización. 

Si el negocio es ofrecer el propio servicio de Odoo, se puede optar por contratar un IAAS y, sobre él, dar un servicio SAAS con personalizaciones a medida y cobrar por el servicio y por las personalizaciones.

Si estamos en una empresa que no puede depender de tener o no red por el sistema productivo, entonces eliminaremos todas las opciones de la nube.


## Referencias y Recursos

- [Vídeo de instalar Odoo manualmente](https://www.youtube.com/watch?v=IPjqbwA814Q)
- [Vídeo de configurar Odoo como servicio](https://www.youtube.com/watch?v=36XolOPbOFI)
- [Vídeo de Copias de seguridad](https://www.youtube.com/watch?v=AvMXfYAD0PA)

- **Documentación oficial**: [https://www.odoo.com/documentation](https://www.odoo.com/documentation)
- **Código fuente**: [https://github.com/odoo/odoo](https://github.com/odoo/odoo)
- **Foros de la comunidad**: [https://www.odoo.com/forum](https://www.odoo.com/forum)
- **Odoo Apps Store**: [https://apps.odoo.com](https://apps.odoo.com)
- **Tutoriales**: [https://www.odoo.com/slides](https://www.odoo.com/slides)
- **Manual technical training Odoo**: [https://github.com/odoo/technical-training/tree/12.0-99-sysadmin](https://github.com/odoo/technical-training/tree/12.0-99-sysadmin)
- [Vídeo de instalar Odoo manualmente](https://www.youtube.com/watch?v=IPjqbwA814Q)
- [Vídeo de configurar Odoo como servicio](https://www.youtube.com/watch?v=36XolOPbOFI)
- [Vídeo de Copias de seguridad](https://www.youtube.com/watch?v=AvMXfYAD0PA)
