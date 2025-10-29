---
title: Redes y Volúmenes 
description: Guía básica de Docker. Redes y volúmenes
---

Docker permite configurar redes virtuales entre contenedores y gestionar la persistencia de datos mediante distintos tipos de volúmenes. Estas capacidades son fundamentales para construir entornos distribuidos, simular arquitecturas reales y garantizar que los datos sobrevivan al ciclo de vida de los contenedores.

## Redes en Docker

### Redes predefinidas

Al instalar Docker, se crean tres redes internas:

- `bridge`: red por defecto. Asigna una IP privada al contenedor y lo conecta mediante la interfaz virtual `docker0`.
- `host`: el contenedor comparte la pila de red del anfitrión. No hay aislamiento.
- `none`: el contenedor no tiene acceso a ninguna red externa, solo loopback.

Para inspeccionar la red `docker0`:

```bash
ip a show docker0
```

> Más documentación: [`docker network`](https://docs.docker.com/network/)

### Creación de redes personalizadas

Docker permite crear redes aisladas para agrupar contenedores:

```bash
docker network create redtest
```

Opciones adicionales:

- `--internal`: red sin acceso externo.
- `--subnet`: define la subred en formato CIDR.
- `--gateway`: especifica la puerta de enlace.
- `--ip-range`: delimita el rango de IPs asignables.

> Más documentación: [`docker network create`](https://docs.docker.com/engine/reference/commandline/network_create/)


### Inspección y eliminación de redes

Listar redes:

```bash
docker network ls
```

Inspeccionar una red:

```bash
docker network inspect redtest
```

Eliminar una red (solo si no hay contenedores conectados):

```bash
docker network rm redtest
```

## Asignación de redes a contenedores

Al crear un contenedor, se puede especificar la red:

```bash
docker run -it --network redtest ubuntu /bin/bash
```

También se puede asignar un alias DNS:

```bash
docker run -it --network redtest --network-alias miservidor --name prueba3 alpine
```

Esto permite que otros contenedores en la misma red accedan a `prueba3` usando el nombre `miservidor`.

### Conexión y desconexión dinámica

Conectar un contenedor a una red existente:

```bash
docker network connect redtest prueba1
```

Opciones útiles:

- `--alias`: asigna un nombre DNS adicional.
- `--ip`: asigna una IP fija.

Desconectar un contenedor:

```bash
docker network disconnect redtest prueba1
```

> Más documentación: [`docker network connect`](https://docs.docker.com/engine/reference/commandline/network_connect/)

## Persistencia de datos

La persistencia de datos en Docker permite conservar información más allá del ciclo de vida de los contenedores, asegurando que los archivos, configuraciones o resultados generados no se pierdan al detener o eliminar un contenedor. Esta funcionalidad es esencial cuando se trabaja con servicios que requieren almacenamiento duradero, como bases de datos, servidores web o aplicaciones que gestionan archivos. Docker ofrece varias estrategias para lograr esta persistencia, cada una con características técnicas y casos de uso específicos: los montajes directos desde el sistema anfitrión (binding mounts), los volúmenes gestionados por Docker y los volúmenes temporales en memoria (`tmpfs`). 

A continuación revisamos estas 3 formas:

- **Binding mount**: monta un directorio del anfitrión en el contenedor.
- **Volúmenes gestionados por Docker**: abstraen la ubicación física del almacenamiento.
- **Volúmenes `tmpfs`**: almacenan datos en memoria, sin persistencia.

> Más documentación: [`Docker storage`](https://docs.docker.com/storage/)

### Binding mount

Montaje directo de un directorio del anfitrión:

```bash
docker run -d -it --name appcontainer -v /home/alumno/target:/app nginx:latest
```

O con sintaxis explícita:

```bash
docker run -d -it --name appcontainer --mount type=bind,source=/home/alumno/target,target=/app nginx:latest
```

> Más documentación: [`bind mounts`](https://docs.docker.com/storage/bind-mounts/)

### Volúmenes Docker

Montaje de un volumen gestionado por Docker:

```bash
docker run -d -it --name appcontainer -v mivolumen:/app nginx:latest
```

O con sintaxis explícita:

```bash
docker run -d -it --name appcontainer --mount source=mivolumen,target=/app nginx:latest
```

Gestión de volúmenes:

```bash
docker volume create mivolumen
docker volume ls
docker volume rm mivolumen
```

> Más documentación: [`volumes`](https://docs.docker.com/storage/volumes/)

### Volúmenes `tmpfs`

Montaje en memoria (sin persistencia):

```bash
docker run -d -it --tmpfs /app nginx
```

O con sintaxis explícita:

```bash
docker run -d -it --mount type=tmpfs,destination=/app nginx
```

> Más documentación: [`tmpfs`](https://docs.docker.com/storage/tmpfs/)

### Copia de seguridad de volúmenes
Seguimos
Ejemplo para copiar el contenido de un volumen a un archivo `.tar`:

```bash
docker stop contenedor1
docker run --rm --volumes-from contenedor1 -v /home/alumno/backup:/backup ubuntu bash -c "cd /datos && tar cvf /backup/copiaseguridad.tar ."
```

Este comando lanza un contenedor temporal que accede al volumen montado en `/datos` y guarda su contenido en `/home/alumno/backup`.


## Tabla de comandos

### Redes

| Comando | Descripción | Ejemplo |
|--------|-------------|---------|
| `docker network create redtest` | Crea una red personalizada llamada `redtest`. | `docker network create redtest` |
| `docker network ls` | Lista todas las redes disponibles en el sistema. | `docker network ls` |
| `docker network rm redtest` | Elimina la red `redtest` (si no está en uso). | `docker network rm redtest` |
| `docker run -it --network redtest ubuntu /bin/bash` | Crea un contenedor conectado a la red `redtest`. | `docker run -it --network redtest ubuntu /bin/bash` |
| `docker network connect redtest prueba1` | Conecta el contenedor `prueba1` a la red `redtest`. | `docker network connect redtest prueba1` |
| `docker network disconnect redtest prueba1` | Desconecta el contenedor `prueba1` de la red `redtest`. | `docker network disconnect redtest prueba1` |


### Volúmenes

| Comando | Descripción | Ejemplo |
|--------|-------------|---------|
| `docker run -d -it --name appcontainer -v /home/alumno/target:/app nginx:latest` | Crea un contenedor con volumen tipo binding mount. | `docker run -d -it --name appcontainer -v /home/alumno/target:/app nginx:latest` |
| `docker run -d -it --name appcontainer -v micontenedor:/app nginx:latest` | Crea un contenedor con volumen gestionado por Docker. | `docker run -d -it --name appcontainer -v micontenedor:/app nginx:latest` |
| `docker volume create mivolumen` | Crea un volumen vacío llamado `mivolumen`. | `docker volume create mivolumen` |
| `docker volume ls` | Lista todos los volúmenes existentes. | `docker volume ls` |
| `docker volume rm mivolumen` | Elimina el volumen `mivolumen` (si no está en uso). | `docker volume rm mivolumen` |
| `docker volume rm $(docker volume ls -q)` | Elimina todos los volúmenes del sistema. | `docker volume rm $(docker volume ls -q)` |
| `docker run -d -it --tmpfs /app nginx` | Crea un contenedor con volumen temporal en memoria (`tmpfs`). | `docker run -d -it --tmpfs /app nginx` |
| `docker run --rm --volumes-from contenedor1 -v /home/alumno/backup:/backup ubuntu bash -c "cd /datos && tar cvf /backup/copiaseguridad.tar ."` | Realiza una copia de seguridad del volumen montado en `/datos` del contenedor `contenedor1`. | Ver comando completo a la izquierda |

## 🧩 Ejemplo de uso guiado

!!!example "Práctica guiada: redes y volúmenes en Docker"

    Configurar redes personalizadas entre contenedores y aplicar distintos tipos de volúmenes para gestionar la persistencia de datos. Esta práctica se realiza íntegramente desde la terminal en entornos WSL con Ubuntu.

    1. Crear una red personalizada

        ```bash
        docker network create redtest
        ```

        Verifica que se ha creado:

        ```bash
        docker network ls
        ```

    2. Crear dos contenedores en la misma red

        ```bash
        docker run -it --network redtest --name prueba1 alpine
        ```

        Dentro del contenedor, sal con `exit` y vuelve a iniciarlo:

        ```bash
        docker start prueba1
        ```

        Lanza el segundo contenedor:

        ```bash
        docker run -it --network redtest --name prueba2 alpine
        ```

        Desde `prueba2`, comprueba la conectividad con `prueba1`:

        ```bash
        ping prueba1
        ```

    3. Asignar alias DNS a un contenedor

        ```bash
        docker run -it --network redtest --network-alias miservidor --name prueba3 alpine
        ```

        Desde otro contenedor en la misma red, prueba:

        ```bash
        ping miservidor
        ```

    4. Conectar y desconectar contenedores de redes

        Conectar un contenedor a otra red:

        ```bash
        docker network connect redtest prueba1
        ```

        Desconectarlo:

        ```bash
        docker network disconnect redtest prueba1
        ```

    5. Crear un volumen tipo binding mount

        ```bash
        docker run -d -it --name appcontainer -v /home/alumno/target:/app nginx:latest
        ```

        Verifica que el contenido de `/home/alumno/target` está accesible en `/app` dentro del contenedor.

    6. Crear un volumen gestionado por Docker

        ```bash
        docker volume create mivolumen
        docker run -d -it --name appcontainer2 -v mivolumen:/app nginx:latest
        ```

        Verifica que el volumen aparece en la lista:

        ```bash
        docker volume ls
        ```

    7. Crear un volumen temporal en memoria (`tmpfs`)

        ```bash
        docker run -d -it --tmpfs /app nginx
        ```

        Este volumen no se guarda en disco y desaparece al detener el contenedor.

    8. Realizar copia de seguridad de un volumen

        ```bash
        docker stop contenedor1
        docker run --rm --volumes-from contenedor1 -v /home/alumno/backup:/backup ubuntu bash -c "cd /datos && tar cvf /backup/copiaseguridad.tar ."
        ```

        Verifica que el archivo `copiaseguridad.tar` se ha creado en `/home/alumno/backup`.

