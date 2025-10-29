---
title: Docker básico
description: Guía básica de Docker. Docker básico
---


## Operaciones fundamentales con Docker desde terminal

Una vez instalado Docker y verificada su correcta ejecución, se puede comenzar a trabajar con contenedores e imágenes utilizando exclusivamente la línea de comandos. Esta unidad recoge las acciones más habituales que se realizarán durante el curso, con ejemplos reproducibles y enlaces a documentación oficial.

### Imágenes y contenedores

Una imagen de Docker es una plantilla de solo lectura que contiene el sistema de archivos y las instrucciones necesarias para crear un contenedor. Las imágenes pueden derivarse unas de otras mediante capas, lo que permite reutilizar configuraciones y construir nuevas versiones de forma incremental.

Un contenedor es una instancia en ejecución (o parada) de una imagen. Cada contenedor tiene un identificador único (64 caracteres) y puede referenciarse por su nombre o por una versión abreviada del ID, siempre que sea unívoca.

Ejemplo de analogía: si una imagen es el DVD de instalación de una distribución Linux, el contenedor sería el sistema operativo instalado y funcionando.

### Almacenamiento interno de Docker

Docker almacena sus datos en el directorio `/var/lib/docker`, incluyendo imágenes, contenedores y configuraciones. El driver de almacenamiento más habitual en sistemas Ubuntu es `overlay2`, que organiza las capas de las imágenes en subdirectorios.

Para consultar esta información:

```bash
docker info
```

> Más detalles sobre drivers de almacenamiento: [Docker: Select a storage driver](https://docs.docker.com/storage/storagedriver/select-storage-driver/) 


### Registro de imágenes: Docker Hub

Docker Hub es el registro público por defecto utilizado por Docker. Permite buscar, descargar y publicar imágenes, tanto oficiales como de terceros. La mayoría de imágenes utilizadas en el curso se obtendrán desde este repositorio.

> Buscador de imágenes: [https://hub.docker.com/search?q=&type=image](https://hub.docker.com/search?q=&type=image)

También es posible configurar registros privados o alternativos si se requiere.

> Guía para crear un registro privado: [DigitalOcean: Cómo configurar un registro de Docker privado en Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-18-04-es) 


### Creación y ejecución de contenedores

El comando más utilizado es `docker run`, que crea un contenedor a partir de una imagen y lo arranca:

```bash
docker run hello-world
```

Este ejemplo descarga la imagen `hello-world` (si no está presente), crea un contenedor y ejecuta un programa que verifica el funcionamiento de Docker. La imagen se almacena localmente tras la primera ejecución.

Para crear un contenedor sin arrancarlo:

```bash
docker create ubuntu
```

> Documentación oficial:  
> - [`docker run`](https://docs.docker.com/engine/reference/commandline/run/)  
> - [`docker create`](https://docs.docker.com/engine/reference/commandline/create/)

### Listado de contenedores

Para ver los contenedores en ejecución:

```bash
docker ps
```

Para ver todos los contenedores, incluidos los detenidos:

```bash
docker ps -a
```

Este listado incluye información como el ID, la imagen base, el comando de arranque, el estado, los puertos expuestos y el nombre asignado.

> Documentación: [docker container ls](https://docs.docker.com/engine/reference/commandline/ps/) 


### Gestión del ciclo de vida de contenedores

Para arrancar un contenedor ya creado:

```bash
docker start nombre_o_id
```

Para detenerlo:

```bash
docker stop nombre_o_id
```

Para reiniciarlo:

```bash
docker restart nombre_o_id
```

> Documentación:  
> - [`start`](https://docs.docker.com/engine/reference/commandline/start/)  
> - [`stop`](https://docs.docker.com/engine/reference/commandline/stop/)  
> - [`restart`](https://docs.docker.com/engine/reference/commandline/restart/)

### Inspección de contenedores

Para obtener información detallada de un contenedor:

```bash
docker inspect nombre_o_id
```

Este comando devuelve un objeto JSON con detalles sobre configuración, red, volúmenes, imagen base, etc.

> Documentación: [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) 


### Ejecución de comandos en contenedores

Para ejecutar comandos dentro de un contenedor en ejecución:

```bash
docker exec -it nombre_o_id bash
```

También se pueden establecer variables de entorno:

```bash
docker exec -it -e VAR1=valor nombre_o_id bash
```

Ejemplo en segundo plano:

```bash
docker exec -d nombre_o_id touch /tmp/prueba
```

> Más documentación: [`docker container exec`](https://docs.docker.com/engine/reference/commandline/exec/) 


### Transferencia de archivos

Para copiar archivos entre el sistema anfitrión y un contenedor:

```bash
docker cp contenedor:/ruta/origen ./destino_local
docker cp ./archivo_local contenedor:/ruta/destino
```

No se permite la copia directa entre contenedores.

Mas documentación: [`docker container cp`](https://docs.docker.com/engine/reference/commandline/cp/) 



### Acceso a procesos en ejecución

Para enlazar la entrada/salida estándar de un contenedor en ejecución:

```bash
docker attach nombre_o_id
```

Este comando permite observar directamente lo que está generando el proceso activo.

> Más documentación: [`docker container attach` ](https://docs.docker.com/engine/reference/commandline/attach/)


### Consulta de logs

Para obtener los logs generados por un contenedor:

```bash
docker logs nombre_o_id
```

Ejemplo con seguimiento en tiempo real:

```bash
docker logs -f --until=2s nombre_o_id
```

> Más documentación: [`docker container logs`](https://docs.docker.com/engine/reference/commandline/logs/) 


### Renombrar contenedores

Para cambiar el nombre de un contenedor:

```bash
docker rename antiguo_nombre nuevo_nombre
```

> Más Documentación: [`docker container rename`](https://docs.docker.com/engine/reference/commandline/rename/) 


## Tabla resumen de comandos básicos de Docker

| Comando | Descripción |
|--------|-------------|
| `docker run -d --name=cont1 ubuntu tail -f /dev/null` | Crea un contenedor llamado `cont1` con la imagen `ubuntu` y ejecuta un proceso que no termina. |
| `docker run -d -p 1200:80 nginx` | Crea un contenedor con la imagen `nginx` y expone el puerto 80 del contenedor en el puerto 1200 del anfitrión. |
| `docker run -it -e MENSAJE=HOLA ubuntu:24.04 bash` | Crea un contenedor interactivo con Ubuntu 24.04 y define la variable de entorno `MENSAJE`. |
| `docker ps` | Muestra los contenedores en ejecución. |
| `docker ps -a` | Muestra todos los contenedores, incluidos los detenidos. |
| `docker start micontenedor` | Arranca un contenedor detenido. |
| `docker start -ai micontenedor` | Arranca un contenedor y enlaza su entrada/salida a la terminal. |
| `docker exec -it idcont bash` | Ejecuta una shell interactiva dentro del contenedor `idcont`. |
| `docker exec -it -e FICHERO=prueba cont bash` | Ejecuta una shell en `cont` con la variable de entorno `FICHERO=prueba`. |
| `docker exec -d cont touch /tmp/prueba` | Ejecuta en segundo plano la creación de un archivo dentro del contenedor. |
| `docker attach idcontainer` | Enlaza la terminal al proceso principal del contenedor. |
| `docker logs -n 10 idcontainer` | Muestra las últimas 10 líneas del log del contenedor. |
| `docker cp idcontainer:/tmp/prueba ./` | Copia un archivo del contenedor al anfitrión. |
| `docker cp ./miFichero idcontainer:/tmp` | Copia un archivo del anfitrión al contenedor. |

---

## 🧩 Ejemplo guiado de uso

!!!example "Práctica guiada: primeros pasos con contenedores"

    1. Crear un contenedor persistente

        ```bash
        docker run -d --name=cont1 ubuntu tail -f /dev/null
        ```

        - Comprueba que el contenedor está en ejecución con `docker ps`.
        - Este contenedor permanecerá activo sin hacer nada, útil para pruebas.


    2. Crear un contenedor con puerto expuesto

        ```bash
        docker run -d -p 1200:80 nginx
        ```

        - Abre un navegador en Windows y accede a `http://localhost:1200` para ver la página de bienvenida de Nginx.

    3. Crear un contenedor con variable de entorno

        ```bash
        docker run -it -e MENSAJE=HOLA ubuntu:24.04 bash
        ```

        - Dentro del contenedor, ejecuta `echo $MENSAJE` para comprobar el valor.

    4. Listar contenedores

        ```bash
        docker ps        # Solo los activos
        docker ps -a     # Todos, incluidos los detenidos
        ```


    5. Detener y volver a arrancar un contenedor

        ```bash
        docker stop cont1
        docker start -ai cont1
        ```

        - Observa cómo se enlaza la terminal al proceso activo.

    6. Ejecutar comandos dentro de un contenedor

        ```bash
        docker exec -it cont1 bash
        ```

        - Accede a una shell dentro del contenedor `cont1`.

        ```bash
        docker exec -it -e FICHERO=prueba cont1 bash
        ```

        - Comprueba que la variable `FICHERO` está disponible con `echo $FICHERO`.

        ```bash
        docker exec -d cont1 touch /tmp/prueba
        ```

        - Verifica que el archivo se ha creado con `ls /tmp`.

    7. Adjuntar y consultar logs

        ```bash
        docker attach cont1
        ```

        - Sal de la sesión con `Ctrl+P` seguido de `Ctrl+Q`.

        ```bash
        docker logs -n 10 cont1
        ```

        - Muestra las últimas líneas del log del contenedor.

    8. Copiar archivos entre anfitrión y contenedor

        ```bash
        docker cp cont1:/tmp/prueba ./
        docker cp ./miFichero cont1:/tmp
        ```

        - Verifica los archivos copiados dentro del contenedor con `ls /tmp`.

