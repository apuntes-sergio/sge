---
title: Imágenes Docker
description: Guía básica de Docker. Imágenes
---


## Gestión de imágenes en Docker

Docker permite gestionar imágenes de forma flexible: listarlas, descargarlas, eliminarlas, exportarlas, versionarlas y construirlas desde cero. Esta unidad recoge los comandos esenciales para trabajar con imágenes locales y remotas, así como para crear imágenes propias mediante `commit` o `Dockerfile`.


### Listado de imágenes locales

Para consultar las imágenes almacenadas en el sistema:

```bash
docker images
```

Se puede filtrar por repositorio y etiqueta:

```bash
docker images ubuntu:22.04
```

También es posible aplicar filtros avanzados:

```bash
docker images -f=reference="u*:*04"
```

> Más documentación: [`docker images`](https://docs.docker.com/engine/reference/commandline/images/)


### Búsqueda de imágenes en Docker Hub

Para buscar imágenes disponibles en el registro por defecto (Docker Hub):

```bash
docker search ubuntu
```

> Más documentación: [`docker search`](https://docs.docker.com/engine/reference/commandline/search/)


### Descarga de imágenes

Para descargar una imagen sin necesidad de ejecutarla:

```bash
docker pull alpine:3.10
```

> Más documentación: [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/)


### Historial de una imagen

Para consultar las capas que componen una imagen:

```bash
docker history nginx
```

> Más documentación: [`docker history`](https://docs.docker.com/engine/reference/commandline/history/)


### Eliminación de imágenes

Para eliminar una imagen concreta:

```bash
docker rmi ubuntu:22.04
```

Para eliminar todas las imágenes no utilizadas por contenedores:

```bash
docker rmi $(docker images -q)
```

> Más documentación: [`docker rmi`](https://docs.docker.com/engine/reference/commandline/rmi/)


### Eliminación de contenedores

Para eliminar un contenedor detenido:

```bash
docker rm nombre_o_id
```

Para detener y eliminar todos los contenedores:

```bash
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

> Más documentación: [`docker rm`](https://docs.docker.com/engine/reference/commandline/rm/)


### Limpieza completa del sistema

Para eliminar imágenes, contenedores detenidos, redes no utilizadas y caché de construcción:

```bash
docker system prune -a
```

> Más documentación: [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/)


### Crear una imagen a partir de un contenedor

Es posible guardar el estado de un contenedor como una nueva imagen:

```bash
docker commit -a "Autor" -m "Comentario" nombre_o_id usuariodockerhub/imagen:2025
```

Para añadir una etiqueta adicional:

```bash
docker tag usuariodockerhub/imagen:2025 usuariodockerhub/imagen:latest
```

> Más documentación:  
> [`docker commit`](https://docs.docker.com/engine/reference/commandline/commit/)  
> [`docker tag`](https://docs.docker.com/engine/reference/commandline/tag/)


### Exportar e importar imágenes

Para guardar una imagen en un archivo:

```bash
docker save -o copia.tar usuariodockerhub/imagen
```

O bien:

```bash
docker save usuariodockerhub/imagen > copia.tar
```

Para importar una imagen desde archivo:

```bash
docker load -i copia.tar
```

O bien:

```bash
docker load < copia.tar
```

> Más documentación:  
> [`docker save`](https://docs.docker.com/engine/reference/commandline/save/)  
> [`docker load`](https://docs.docker.com/engine/reference/commandline/load/)


### Subida de imágenes a `Docker Hub`

1. Iniciar sesión:

    ```bash
    docker login
    ```

2. Crear el repositorio en https://hub.docker.com  
3. Etiquetar la imagen si es necesario:

    ```bash
    docker tag imagen_local usuariodockerhub/nombre_repo
    ```

4. Subir la imagen:

    ```bash
    docker push usuariodockerhub/nombre_repo
    ```

> Más documentación:  
> [`docker login`](https://docs.docker.com/engine/reference/commandline/login/)  
> [`docker push`](https://docs.docker.com/engine/reference/commandline/push/)




## Creación de contenedores personalizados

Docker permite construir imágenes propias a partir de instrucciones declarativas contenidas en un fichero llamado `Dockerfile`. Estas imágenes pueden incluir software preinstalado, configuraciones específicas, scripts de arranque, usuarios personalizados, variables de entorno, puertos expuestos y cualquier otra personalización que se desee aplicar al entorno de ejecución.

Este enfoque es especialmente útil en contextos educativos, ya que permite preparar entornos reproducibles para prácticas, simulaciones o despliegues controlados.

### Fundamentos del Dockerfile

Un `Dockerfile` es un archivo de texto plano que define, paso a paso, cómo debe construirse una imagen. Cada línea representa una instrucción que Docker ejecutará durante el proceso de construcción (`docker build`). Las instrucciones más comunes son:

- `FROM`: define la imagen base sobre la que se construye. Puede ser una distribución Linux (`ubuntu`, `alpine`) o una imagen especializada (`python`, `node`, etc.).
- `LABEL`: añade metadatos como el autor o el propósito de la imagen.
- `RUN`: ejecuta comandos en la imagen durante la construcción. Se utiliza para instalar paquetes, modificar archivos, crear usuarios, etc.
- `COPY` o `ADD`: copia archivos locales al contenedor o descarga recursos externos.
- `WORKDIR`: establece el directorio de trabajo dentro del contenedor.
- `ENV`: define variables de entorno persistentes.
- `EXPOSE`: declara puertos que el contenedor utilizará (no los publica en el host).
- `CMD`: indica el comando que se ejecutará por defecto al lanzar el contenedor.
- `ENTRYPOINT`: define el proceso principal del contenedor, útil para imágenes que deben comportarse como ejecutables.

> Más documentación: [`Dockerfile reference`](https://docs.docker.com/engine/reference/builder/)


!!!tip  "Buenas prácticas en la construcción de imágenes"

    - Utilizar imágenes base ligeras (`alpine`, `slim`) cuando sea posible.
    - Agrupar comandos en un solo `RUN` para reducir el número de capas.
    - Evitar copiar archivos innecesarios al contenedor.
    - Usar `CMD` para definir el comportamiento por defecto y `ENTRYPOINT` si se requiere control total del proceso.
    - Añadir etiquetas (`docker tag`) para versionar las imágenes y facilitar su gestión.
    - Documentar el propósito de cada instrucción dentro del Dockerfile.


### Flujo de trabajo típico

1. Crear un directorio de trabajo con los archivos necesarios (scripts, configuraciones, etc.).
2. Escribir el `Dockerfile` con las instrucciones deseadas.
3. Construir la imagen:

   ```bash
   docker build -t nombreimagen .
   ```

4. Verificar que la imagen se ha creado:

   ```bash
   docker images
   ```

5. Lanzar un contenedor basado en la imagen:

   ```bash
   docker run -it nombreimagen
   ```

6. (Opcional) Subir la imagen a Docker Hub:

   ```bash
   docker login
   docker push usuario/nombreimagen
   ```

---

### Ejemplos de contenedores personalizados

Los siguientes ejemplos ilustran distintos enfoques de personalización:

- **Servidor web con Apache y PHP sobre Alpine**: imagen ligera con entorno gráfico mínimo y configuración web básica.
- **Aplicación Flask en Python**: entorno de desarrollo web con dependencias gestionadas por `pip`.
- **Contenedor de diagnóstico de red en Ubuntu**: imagen con herramientas CLI para prácticas de redes.

!!!example "1 Servidor web con Apache y PHP sobre Alpine"

    El siguiente contenedor está pensado para servir una página PHP básica mediante Apache, con configuración mínima y sin entorno gráfico.

    ```Dockerfile
    FROM alpine
    LABEL maintainer="email@gmail.com"
    RUN apk update && apk add apache2 php php-apache2 openrc tar
    ADD ./start.sh /start.sh
    ADD https://gist.githubusercontent.com/SyntaxC4/5648247/raw/info.php /var/www/localhost/htdocs/index.php
    RUN adduser -u 82 -D -S -G www-data www-data
    RUN chown -R www-data:www-data /var/www/ && chmod -R 775 /var/www/ && chmod 755 /start.sh
    EXPOSE 80
    CMD /start.sh
    ```
    Explicación

    - **FROM alpine**: usa como base la imagen Alpine Linux, una distribución ligera ideal para contenedores.
    - **LABEL maintainer**: añade metadatos indicando el responsable de la imagen.
    - **RUN apk update && apk add ...**: actualiza el índice de paquetes e instala Apache2, PHP y herramientas necesarias.
    - **ADD ./start.sh /start.sh**: copia un script local al contenedor. Este script se usará para iniciar Apache.
    - **ADD https://.../info.php ...**: descarga un archivo PHP de ejemplo y lo coloca como página principal.
    - **RUN adduser ...**: crea un usuario `www-data` con UID 82 y lo asocia al grupo del mismo nombre.
    - **RUN chown/chmod**: asigna propiedad y permisos adecuados al directorio web y al script de arranque.
    - **EXPOSE 80**: declara el puerto 80 como expuesto para otros contenedores (no lo publica en el host).
    - **CMD /start.sh**: define el comando que se ejecutará por defecto al lanzar el contenedor.


!!!example "2: Contenedor con Python y Flask"

    Este contenedor es ideal para enseñar desarrollo web básico con Flask o para montar APIs REST en prácticas.

    ```Dockerfile
    FROM python:3.11-slim
    LABEL maintainer="docente@centro.edu"
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt
    COPY app.py .
    EXPOSE 5000
    CMD ["python", "app.py"]
    ```

    - Usa una imagen ligera de Python 3.11.
    - Instala dependencias desde `requirements.txt` (por ejemplo, Flask).
    - Copia el archivo principal `app.py` al contenedor.
    - Expone el puerto 5000 (por defecto en Flask).
    - Lanza la aplicación con `python app.py`.


!!!example "3: Contenedor de Ubuntu con herramientas de red"

    Este contenedor puede utilizarse en prácticas de redes, para enseñar comandos básicos o simular entornos de prueba.

    ```Dockerfile
    FROM ubuntu:24.04
    LABEL maintainer="admin@fp.edu"
    RUN apt update && apt install -y net-tools iputils-ping curl dnsutils
    CMD ["/bin/bash"]
    ```

    - Usa Ubuntu 24.04 como base.
    - Instala herramientas de red útiles para diagnóstico (`ping`, `netstat`, `dig`, `curl`, etc.).
    - Lanza una shell interactiva por defecto.








## Tabla resumen de comandos 

### Gestión de imágenes

| Comando | Descripción | Ejemplo |
|--------|-------------|---------|
| `docker images` | Lista las imágenes locales disponibles. | `docker images` |
| `docker search <nombre>` | Busca imágenes en Docker Hub. | `docker search ubuntu` |
| `docker pull <imagen>` | Descarga una imagen desde el registro. | `docker pull alpine` |
| `docker history <imagen>` | Muestra el historial de capas de una imagen. | `docker history alpine` |
| `docker rmi <imagen:tag>` | Elimina una imagen local. | `docker rmi ubuntu:14.04` |
| `docker rmi $(docker images -q)` | Elimina todas las imágenes no utilizadas. | `docker rmi $(docker images -q)` |
| `docker rm <id>` | Elimina un contenedor detenido. | `docker rm 3fb53852ce99` |
| `docker stop $(docker ps -a -q)` | Detiene todos los contenedores. | `docker stop $(docker ps -a -q)` |
| `docker rm $(docker ps -a -q)` | Elimina todos los contenedores detenidos. | `docker rm $(docker ps -a -q)` |
| `docker system prune -a` | Elimina imágenes, contenedores y redes no utilizados. | `docker system prune -a` |

### Creación y exportación de imágenes

| Comando | Descripción | Ejemplo |
|--------|-------------|---------|
| `docker commit -m "<comentario>" <id> <usuario/imagen:tag>` | Crea una imagen a partir de un contenedor. | `docker commit -m "Ubuntu modificado" 3fb5 usuario/ubuntu:mod` |
| `docker save -o <archivo>.tar <imagen>` | Exporta una imagen a un archivo `.tar`. | `docker save -o copiaSeguridad.tar alpine` |
| `docker load -i <archivo>.tar` | Importa una imagen desde un archivo `.tar`. | `docker load -i copiaSeguridad.tar` |

### Docker Hub

| Comando | Descripción | Ejemplo |
|--------|-------------|---------|
| `docker login` | Inicia sesión en Docker Hub. | `docker login` |
| `docker push <usuario/imagen:tag>` | Sube una imagen al repositorio. | `docker push usuario/ubuntu:mod` |


## 🧩 Ejemplo guiado de uso

!!!example "Práctica guiada: gestión de imágenes en Docker"

    Aprender a listar, descargar, eliminar, exportar, importar y versionar imágenes en Docker, utilizando exclusivamente la línea de comandos en entornos WSL con Ubuntu.

    1. Listar imágenes locales

        ```bash
        docker images
        ```

        Consulta todas las imágenes disponibles en tu sistema. Para filtrar por nombre y versión:

        ```bash
        docker images ubuntu:22.04
        ```

    2. Buscar imágenes en Docker Hub

        ```bash
        docker search alpine
        ```

        Muestra imágenes disponibles en el registro remoto.

    3. Descargar una imagen

        ```bash
        docker pull alpine:3.10
        ```

        Descarga la imagen `alpine` con la etiqueta `3.10`.

    4. Consultar el historial de una imagen

        ```bash
        docker history alpine
        ```

        Muestra las capas que componen la imagen.

    5. Eliminar imágenes

        Eliminar una imagen concreta:

        ```bash
        docker rmi ubuntu:22.04
        ```

        Eliminar todas las imágenes no utilizadas:

        ```bash
        docker rmi $(docker images -q)
        ```

    6. Eliminar contenedores

        Eliminar un contenedor por ID:

        ```bash
        docker rm 3fb53852ce99
        ```

        Detener y eliminar todos los contenedores:

        ```bash
        docker stop $(docker ps -a -q)
        docker rm $(docker ps -a -q)
        ```

    7. Limpieza completa del sistema

        ```bash
        docker system prune -a
        ```

        Elimina contenedores detenidos, imágenes sin uso, redes no utilizadas y caché de construcción.

    8. Crear una imagen a partir de un contenedor

        ```bash
        docker commit -m "Ubuntu modificado" 3fb5 usuario/ubuntu:mod
        ```

    9. Exportar e importar imágenes

        Guardar una imagen en archivo `.tar`:

        ```bash
        docker save -o copiaSeguridad.tar alpine
        ```

        Importar una imagen desde archivo:

        ```bash
        docker load -i copiaSeguridad.tar
        ```

    10.  Subir imágenes a Docker Hub

        Iniciar sesión:

        ```bash
        docker login
        ```

        Subir imagen:

        ```bash
        docker push usuario/ubuntu:mod
        ```


