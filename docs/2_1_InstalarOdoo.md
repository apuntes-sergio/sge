---
title: 2.1. Instalación de Odoo
description: Pasos para la instalación y configuración de Odoo
---

# Instalar `Odoo`

`Odoo` puede instalarse en cualquier sistema operativo. Sin embargo, se desarrolla pensando en Ubuntu o Debian y es el sistema en el que vamos a trabajar.

`Odoo`, en esencia, es un servidor web hecho en Python que se conecta con una base de datos PostgreSQL. Hay muchas maneras de instalar `Odoo`, desde las más avanzadas, que son descargar por *git* el repositorio y hacer que arranque al inicio, hasta las más simples, que son desplegar un **docker** con todo funcionando.


## Opciones para la instalación de `Odoo`

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


## Enlaces

-   [Manual technical training Odoo](https://github.com/odoo/technical-training/tree/12.0-99-sysadmin)
-   [Vídeo de instalar Odoo manualmente](https://www.youtube.com/watch?v=IPjqbwA814Q)
-   [Vídeo de configurar Odoo como servicio](https://www.youtube.com/watch?v=36XolOPbOFI)
-   [Vídeo de Copias de seguridad](https://www.youtube.com/watch?v=AvMXfYAD0PA)
