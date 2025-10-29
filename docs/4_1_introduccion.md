---
title: Introducción
description: Guía básica de Docker. Introducción.
---

## Introducción a los contenedores

La unidad comienza explicando el propósito de introducir el concepto de contenedores, centrándose especialmente en los contenedores Linux y en la tecnología Docker. Se plantea como una alternativa moderna a otras formas de virtualización, con ventajas específicas para el desarrollo y despliegue de aplicaciones.

## Fundamentos de la virtualización

Se presenta la virtualización como una técnica que permite abstraer el hardware físico, simulando recursos virtuales que se comportan como reales. Esta tecnología es ampliamente utilizada en entornos de desarrollo, pruebas, análisis de malware y despliegue de sistemas, por su capacidad de reducir costes y facilitar la gestión de recursos.

Se describen cuatro enfoques principales:

- Las máquinas virtuales de proceso permiten ejecutar software diseñado para otra arquitectura como si fuera un proceso más del sistema anfitrión. Ejemplos destacados son la JVM y la plataforma .NET.
- Los emuladores reproducen hardware o APIs específicas, como los emuladores de consolas o Wine para ejecutar software Windows en otros sistemas.
- Los hipervisores permiten instalar sistemas operativos completos sobre hardware virtualizado. Herramientas como VirtualBox o VMWare son ejemplos comunes.
- Los contenedores, que se explican en profundidad más adelante, representan una forma más ligera de virtualización.

## Contenedores como alternativa ligera

Los contenedores se definen como entornos privados que comparten el núcleo del sistema anfitrión, sin necesidad de virtualizar el hardware completo. Esto permite aislar procesos, memoria, red y sistema de archivos, manteniendo eficiencia y velocidad.

Se utiliza una analogía con los contenedores marítimos: mientras cumplan un estándar, pueden transportarse independientemente del contenido. De forma similar, los contenedores virtuales pueden ejecutarse en cualquier sistema compatible, sin importar qué software contienen.

## Aplicaciones prácticas de los contenedores

Los contenedores son especialmente útiles en desarrollo y despliegue de aplicaciones. Permiten montar entornos de compilación con versiones específicas, realizar pruebas en configuraciones diversas y evitar problemas de compatibilidad al distribuir software.

También se emplean en el despliegue de servicios como servidores web, bases de datos o DNS. Facilitan la replicación exacta de entornos locales en la nube, eliminando el clásico problema de “funciona en mi máquina”. Además, permiten escalar horizontalmente servicios mediante orquestadores.

## Ventajas y limitaciones

Entre las ventajas destacan:

- Menor consumo de espacio al no replicar sistemas operativos.
- Ejecución rápida, cercana a la velocidad nativa.
- Amplio soporte por parte de empresas y disponibilidad de imágenes oficiales.

Entre las limitaciones se señalan:

- Rendimiento inferior al de ejecución directa sobre hardware.
- Gestión más compleja de datos persistentes.
- Uso predominantemente por línea de comandos, con entornos gráficos posibles pero poco prácticos.

## Contextos recomendados para su uso

El uso de contenedores es adecuado en varios escenarios:

- Para usuarios que desean probar servicios sin complicaciones.
- Para desarrolladores que buscan entornos portables y reproducibles.
- Para realizar pruebas con distintas configuraciones y recursos.
- Para escalar servicios distribuidos en clústeres.

## Contenedores en sistemas Linux

Se explica que el concepto de entornos privados no es nuevo en Unix. Herramientas como `chroot` y `jail` ya ofrecían aislamiento desde los años 80 y 90. Los contenedores modernos, como LXC y LXD, se basan en capacidades del kernel de Linux como los namespaces (para aislar procesos) y los cgroups (para limitar recursos).

También se menciona que es posible crear contenedores “a mano” usando scripts, y se recomiendan recursos como los de Julia Evans para entender mejor el funcionamiento interno.

## Compatibilidad con otros sistemas

Aunque los contenedores Linux están diseñados para ejecutarse en sistemas Linux, pueden funcionar en Windows y MacOS mediante virtualización. En Windows se usa WSL2 o Hyper-V, y en MacOS, Docker Desktop o Hyperkit. Estas soluciones permiten ejecutar contenedores, aunque con menor rendimiento.

## Introducción a Docker

Docker se presenta como una solución de contenedores Linux que aprovecha las capacidades del kernel para facilitar el desarrollo y despliegue. Existen dos versiones: Docker CE (Community Edition) y Docker EE (Enterprise Edition), ambas compatibles con servicios en la nube como AWS, Azure y Google Cloud.

La arquitectura de Docker se divide en tres componentes:

- El cliente, que envía órdenes.
- El servidor (Docker Host), que gestiona contenedores e imágenes.
- El registro (Registry), que almacena imágenes, siendo Docker Hub el más popular.

## Docker en Windows y MacOS

Docker puede ejecutarse en sistemas no Linux mediante Docker Desktop, que instala todo lo necesario para lanzar contenedores. En Windows se apoya en WSL2 o Hyper-V, y en MacOS en Hyperkit. Aunque el rendimiento no es nativo, estas soluciones permiten trabajar con contenedores de forma práctica.

## Contenedores no Linux con Docker

Además de contenedores Linux, Docker puede lanzar contenedores con Windows Server Core en sistemas Windows, y contenedores MacOS en sistemas Linux mediante KVM. Aunque no son los casos más comunes, amplían las posibilidades de uso en entornos específicos.

Perfecto, Sergio. Aquí tienes la documentación técnica de instalación de Docker, redactada como continuación directa de los apuntes anteriores. He eliminado cualquier introducción redundante sobre qué es Docker y he reforzado que en el entorno de clase se trabajará con WSL y Ubuntu desde la línea de comandos.


## Instalación de Docker en entornos educativos con WSL y Ubuntu

Durante el curso, utilizaremos Docker sobre Ubuntu instalado en WSL (Windows Subsystem for Linux). Esta configuración permite trabajar con contenedores Linux desde sistemas Windows, manteniendo compatibilidad con el kernel y acceso a herramientas CLI. Todas las instrucciones que siguen están verificadas para distribuciones basadas en Ubuntu 22.04 o 24.04, y se ejecutarán desde la terminal de WSL.

### Eliminación de versiones anteriores

Antes de instalar Docker, es recomendable eliminar cualquier versión previa que pueda generar conflictos. Ejecuta el siguiente comando en tu terminal WSL:

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

Este paso garantiza que no haya interferencias con versiones obsoletas o instaladas desde repositorios no oficiales.

### Configuración del repositorio oficial de Docker

Docker CE debe instalarse desde su repositorio oficial para garantizar acceso a versiones actualizadas y soporte completo.

1. Actualiza el índice de paquetes e instala dependencias necesarias:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
```

2. Añade la clave GPG del repositorio:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

3. Añade el repositorio de Docker CE. En Ubuntu puro, puedes usar:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

⚠️ En distribuciones derivadas como Linux Mint, `lsb_release -cs` puede devolver un nombre no reconocido por Docker. En ese caso, sustituye manualmente por el nombre de la versión base de Ubuntu (por ejemplo, `noble` para Ubuntu 24.04).

### Instalación del motor Docker

Una vez añadido el repositorio, instala Docker Engine CE y sus componentes:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Verifica la instalación con:

```bash
docker version
```

Este comando debe mostrar tanto la versión del cliente como del servidor Docker. Si aparece un error de permisos, consulta el apartado siguiente.

### Uso sin privilegios de superusuario

Por defecto, Docker requiere permisos de root para ejecutarse. En entornos educativos, es útil permitir que los alumnos usen Docker sin `sudo`. Para ello:

1. Crea el grupo `docker` (si no existe):

```bash
sudo groupadd docker
```

2. Añade el usuario actual al grupo:

```bash
sudo usermod -aG docker $USER
```

3. Cierra y vuelve a iniciar sesión en WSL para que los cambios surtan efecto.

Si aparecen errores relacionados con `~/.docker/config.json`, puedes solucionarlo con:

```bash
sudo rm -rf ~/.docker/
```

O bien:

```bash
sudo chown "$USER":"$USER" ~/.docker -R
sudo chmod g+rwx ~/.docker -R
```

### Control del arranque automático

Aunque en WSL el arranque automático no siempre es relevante, en sistemas Linux completos puedes configurar el servicio Docker para que se inicie al arrancar:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

Para desactivarlo:

```bash
sudo systemctl disable docker.service
sudo systemctl disable containerd.service
```

Para gestionar manualmente el servicio:

```bash
sudo systemctl start docker.service
sudo systemctl stop docker.service
sudo systemctl restart docker.service
```

### Desinstalación completa

Si necesitas eliminar Docker por completo:

```bash
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

Y para eliminar contenedores, imágenes y configuraciones:

```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
sudo rm /etc/apt/sources.list.d/docker.list
sudo rm /etc/apt/keyrings/docker.asc
```
