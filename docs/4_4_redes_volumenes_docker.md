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


## 🧩 Ejemplo de uso guiado

!!!example "Comprobar comunicación entre dos contenedores"
  
    Vamos a usar dos contenedores ubuntu (imagen `ubuntu`) y una **red bridge propia** para que las IPs estén en el mismo segmento y tengas resolución por nombre.

    1. Crear la red y arrancar los dos contenedores (cada uno en un terminal)

        En **una terminal** (Terminal A), crea la red y lanza los contenedores en segundo plano:

        ```bash
        # Crear una red bridge aislada
        docker network create demo-net

        # Lanzar dos contenedores en esa red, en background
        docker run -d --name c1 --network demo-net ubuntu sleep infinity
        docker run -d --name c2 --network demo-net ubuntu sleep infinity

        # Verifica que están en marcha
        docker ps 
        ```

        - Usamos `sleep infinity` para que queden ejecutándose en segundo plano.
        - Con `--name` fijas nombres para luego poder entrar por `docker exec`.
        - Ambos están conectados a `demo-net`.

    2. Entrar en **cada contenedor desde dos terminales**

        - **Terminal A** → entra en `c1`:
        ```bash
        docker exec -it c1 bash
        ```

        - **Terminal B** → entra en `c2`:
        ```bash
        docker exec -it c2 bash
        ```

    3. Instalar utilidades dentro de cada contenedor

        En **cada** contenedor (tanto en `c1` como en `c2`), ejecuta:

        ```bash
        apt update && apt install -y iproute2 iputils-ping
        ```

        - `iproute2` → para usar `ip a` o `ip -brief addr`.
        - `iputils-ping` → para poder hacer `ping`.

        Este paso se hace una sola vez por contenedor (si los destruyes y recreas, habrá que repetirlo; o puedes construir una imagen propia que ya lo traiga instalado).

    4. Ver la IP de cada contenedor

        En cada terminal:
        ```bash
        hostname -I
        ip -brief addr
        ping -c 3 c2
        ```

        - La interfaz típica será `eth0` con una IP del rango de `demo-net`.

    5. Limpieza (cuando termines la demo)
        ```bash
        docker rm -f c1 c2
        docker network rm demo-net
        ```

    !!!tip "Variante rápida con Alpine (sin `apt`)"

        Si prefieres evitar la instalación, Alpine trae `ping` (busybox) y puedes usar `ip` instalando `iproute2` vía `apk` si lo necesitas:

        ```bash
        docker network create demo-net
        docker run -d --name c1 --network demo-net alpine sleep infinity
        docker run -d --name c2 --network demo-net alpine sleep infinity

        # En cada una de las contenedores entramos y ejecutamos el ping
        docker exec -it c1 sh
        ```



!!!tip "Listar las IPs de los contenedores activos"

    Hay varias formas de ver las IPs de los servidores activos, por ejemplo aquí tenemos suna: 
    ```bash
    # IP por defecto de cada contenedor (todas las redes)
    docker ps -q | xargs -n1 -I {} docker inspect -f '{{.Name}} -> {{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' {}
    ```


!!!note "Mantener contenedores en ejecución"

    Cuando se ejecuta un contenedor, si no tiene nada que hacer quedará parado automáticamente. Si al hacer el `run` entramos en su terminal, finalizará al salir de el. 
    Para que esto no ocurra tenemos el truco `sleep infinity` que dejará el contenedor activo hasta que lo finalicemos explicitamente

    ```bash
    docker run -d --name c1 --network demo-net alpine sleep infinity
    ```






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


???exercise "Crea una carpeta en y arranca un contenedor ubuntu enlazando esta carpeta a una carpeta del contendor, por ejemplo `/home`. Añade un fichero a la carpeta y comprueba que es accesible desde el contenedor."

    ```bash
    mkdir carpeta
    echo "Fichero visible desde contenedor" > carpeta/fichero.txt
    docker run -it --name my_ubuntu -v ./carpeta:/home ubuntu /bin/bash
    ```

    Comprueba que si creas un fichero en tu carpeta esta

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
docker volume create mivolumen      # crea un volumen
docker volume ls                    # lista todos los volúmenes
docker volume inspect mivolumen     # obtiene información sobre un volumen
docker volume rm mivolumen          # elimina un volumen
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

| Comando Docker | Descripción |
|----------------|-------------|
| `docker network ls` | Lista todas las redes disponibles en Docker. |
| `docker network inspect <nombre_red>` | Muestra detalles de una red, incluyendo contenedores conectados y configuración IP. |
| `docker network create <nombre_red>` | Crea una nueva red personalizada. |
| `docker network create --driver bridge <nombre_red>` | Crea una red tipo bridge (la más común). |
| `docker network create --subnet 192.168.100.0/24 <nombre_red>` | Crea una red con una subred específica en formato CIDR. |
| `docker network rm <nombre_red>` | Elimina una red Docker. |
| `docker run --network <nombre_red> <imagen>` | Ejecuta un contenedor conectado a una red específica. |
| `docker network connect <nombre_red> <contenedor>` | Conecta un contenedor existente a una red. |
| `docker network disconnect <nombre_red> <contenedor>` | Desconecta un contenedor de una red. |

### Volúmenes

| Comando Docker | Descripción |
|----------------|-------------|
| `docker volume ls` | Lista todos los volúmenes existentes. |
| `docker volume create <nombre_volumen>` | Crea un nuevo volumen. |
| `docker volume inspect <nombre_volumen>` | Muestra detalles del volumen, incluyendo su ruta en el sistema. |
| `docker volume rm <nombre_volumen>` | Elimina un volumen (solo si no está en uso). |
| `docker volume prune` | Elimina todos los volúmenes no utilizados. |
| `docker run -v <nombre_volumen>:/ruta/en/contenedor <imagen>` | Ejecuta un contenedor montando un volumen en una ruta específica. |
| `docker run --mount source=<nombre_volumen>,target=/ruta <imagen>` | Alternativa moderna para montar volúmenes usando `--mount`. |
| `docker inspect <contenedor>` | Permite ver qué volúmenes están montados en un contenedor. |


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

