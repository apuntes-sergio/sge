---
title: Orquestación de contenedores con Docker Compose
description: Guía básica de Docker. Orquestación de contenedores con Docker Compose
---

**Docker Compose** es una herramienta que permite definir y gestionar múltiples contenedores como una única aplicación. Utiliza un fichero de configuración en formato YAML (`docker-compose.yml`) para describir los servicios, redes, volúmenes y dependencias que conforman el entorno. Esta funcionalidad resulta especialmente útil en entornos educativos y de desarrollo, donde se requiere levantar arquitecturas completas de forma reproducible y automatizada.

## Ventajas de Docker Compose

- Permite definir múltiples servicios en un solo archivo.
- Automatiza la creación, conexión y configuración de contenedores.
- Facilita la gestión de redes, volúmenes y variables de entorno.
- Admite escalado local de servicios mediante múltiples instancias.
- Simplifica la puesta en marcha de entornos complejos como WordPress, bases de datos, APIs, etc.

> Más documentación: [`Docker Compose`](https://docs.docker.com/compose/)

## Estructura del fichero `docker-compose.yml`

El fichero `docker-compose.yml` se escribe en formato YAML y puede incluir:

- `services`: definición de cada contenedor.
- `volumes`: volúmenes compartidos entre servicios.
- `networks`: redes personalizadas.
- `environment`: variables de entorno.
- `depends_on`: orden de arranque entre servicios.
- `build`: ruta al Dockerfile si se desea construir una imagen personalizada.
- `ports`: mapeo de puertos entre contenedor y anfitrión.
- `restart`: política de reinicio automático.
- `command`: sobreescritura del comando por defecto.

> Más documentación: [`Compose file reference`](https://docs.docker.com/compose/compose-file/compose-file-v3/)

## Ejemplo completo: WordPress + MariaDB

El siguiente fichero `docker-compose.yml` define dos servicios: una base de datos MariaDB y una instancia de WordPress conectada a ella. Incluye volúmenes persistentes, variables de entorno y mapeo de puertos.

```yaml
version: "3.9"

services:
  db:
    image: mariadb:10.11.2
    container_name: wordpress-db
    volumes:
      - db_data:/var/lib/mysql
      - db_config:/etc/mysql/conf.d
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    networks:
      backend:
        aliases:
          - mariadb

  wordpress:
    image: wordpress:latest
    container_name: wordpress-app
    depends_on:
      - db
    ports:
      - "8000:80"
    volumes:
      - wp_data:/var/www/html
      - wp_plugins:/var/www/html/wp-content/plugins
    restart: always
    environment:
      WORDPRESS_DB_HOST: mariadb:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    networks:
      frontend:
      backend:

volumes:
  db_data:
  db_config:
  wp_data:
  wp_plugins:

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

Aspectos a tener en cuenta

- **Redes**

    - **frontend**: conecta WordPress con el exterior (puerto 8000).
    - **backend**: conecta WordPress con la base de datos MariaDB.
    - Separar redes mejora la seguridad y el aislamiento lógico.

- Volúmenes

    - `db_data`: almacena los datos persistentes de MariaDB.
    - `db_config`: permite añadir configuraciones personalizadas a MariaDB.
    - `wp_data`: guarda el contenido principal de WordPress.
    - `wp_plugins`: separa los plugins para facilitar su gestión y copia.

- Alias de red

    Se define `mariadb` como alias en la red `backend`, lo que permite que WordPress se conecte usando `mariadb:3306` como host.

- Contenedores nombrados

    Se asignan nombres explícitos (`wordpress-db`, `wordpress-app`) para facilitar la inspección y depuración.





Para lanzar el entorno:

```bash
docker compose up -d
```

Para detenerlo:

```bash
docker compose down
```


## Comandos principales de Docker Compose

| Comando | Descripción |
|--------|-------------|
| `docker compose up -d` | Lanza los servicios en segundo plano. |
| `docker compose down` | Detiene y elimina los servicios. |
| `docker compose build` | Construye las imágenes definidas en el fichero. |
| `docker compose pull` | Descarga las imágenes necesarias. |
| `docker compose start` / `stop` | Inicia o detiene servicios específicos. |
| `docker compose exec <servicio> <comando>` | Ejecuta un comando dentro de un contenedor. |
| `docker compose ps` | Lista los contenedores activos del proyecto. |
| `docker compose rm` | Elimina contenedores detenidos. |



| Comando | Descripción | Uso de ejemplo |
| --- | --- | --- |
| `docker-compose up` | Crea y inicia contenedores | `docker-compose up` |
| `docker-compose up -d` | Ejecutar en segundo plano | `docker-compose up -d` |
| `docker-compose exec` | Ejecutar un comando en un contenedor en ejecución | `docker-compose exec web bash` |
| `docker-compose build` | Construir/reconstruir imágenes | `docker-compose build` |
| `docker-compose down` | Detener y eliminar contenedores, redes, volúmenes e imágenes | `docker-compose down` |
| `docker-compose logs -f` | Ver y seguir los registros | `docker-compose logs -f` |
| `docker-compose ps` | Listar contenedores en ejecución | `docker-compose ps` |
| `docker-compose run` | Ejecutar comandos de una sola vez (ignora el comando en el archivo Compose) | `docker-compose run web python manage.py migrate` |
| `docker-compose stop` | Detener contenedores en ejecución (se pueden reiniciar con start) | `docker-compose stop` |
| `docker-compose restart` | Reiniciar servicios | `docker-compose restart web` |
| `docker-compose pull` | Descargar imágenes de servicio | `docker-compose pull` |
| `docker-compose rm` | Eliminar contenedores detenidos de servicios | `docker-compose rm web` |
| `docker-compose config` | Validar y ver el archivo Compose | `docker-compose config` |
| `docker-compose up --scale web=3` | Iniciar múltiples instancias de un servicio | `docker-compose up --scale web=3` |

> Más documentación: [`Compose CLI`](https://docs.docker.com/compose/reference/)


## Ejemplos con redes personalizadas y volúmenes

!!!example "Ejemplo 1: NGINX + Redis con red personalizada y volúmenes"

    ```yaml
    version: '3.8'

    services:
      nginx:
        image: nginx:latest
        ports:
          - "8080:80"
        volumes:
          - ./nginx/html:/usr/share/nginx/html
        networks:
          - appnet

      redis:
        image: redis:7
        volumes:
          - redis_data:/data
        networks:
          - appnet

    volumes:
      redis_data:

    networks:
      appnet:
        driver: bridge
    ```

    - **nginx:** sirve archivos estáticos desde `./nginx/html` en el host.
    - **redis:** almacena datos persistentes en el volumen `redis_data`.
    - Ambos servicios están en la red `appnet`, lo que permite que se comuniquen por nombre de servicio (`redis`, `nginx`).

!!!example "Ejemplo 2: PostgreSQL + Adminer con volúmenes y red"

    ```yaml
    version: '3.8'

    services:
      db:
        image: postgres:15
        environment:
          POSTGRES_USER: admin
          POSTGRES_PASSWORD: secret
          POSTGRES_DB: demo
        volumes:
          - pgdata:/var/lib/postgresql/data
        networks:
          - backend

      adminer:
        image: adminer
        ports:
          - "8081:8080"
        networks:
          - backend

    volumes:
      pgdata:

    networks:
      backend:
    ```

    - **PostgreSQL:** base de datos con credenciales configuradas.
    - **Adminer:** interfaz web para gestionar la base de datos.
    - Ambos comparten la red `backend` y el volumen `pgdata` mantiene los datos persistentes.

!!!example "Ejemplo 3 Simulación de XAMPP con Apache + PHP + MySQL"

    ```yaml
    version: '3.8'

    services:
      apache:
        image: webdevops/php-apache:latest
        ports:
          - "80:80"
        volumes:
          - ./htdocs:/app
        networks:
          - xamppnet
        depends_on:
          - mysql

      mysql:
        image: mysql:5.7
        environment:
          MYSQL_ROOT_PASSWORD: rootpass
          MYSQL_DATABASE: xamppdb
          MYSQL_USER: xamppuser
          MYSQL_PASSWORD: xampppass
        volumes:
          - mysql_data:/var/lib/mysql
        networks:
          - xamppnet

    volumes:
      mysql_data:

    networks:
      xamppnet:
    ```


    - **Apache + PHP:** sirve archivos PHP desde `./htdocs`.
    - **MySQL:** base de datos con credenciales configuradas.
    - Ambos servicios están conectados por la red `xamppnet` y usan volúmenes para persistencia.

!!!example "Ejemplo 4: node.js con postgre"

    ```yaml
    version: '3.8'
    services:
      app:
        image: node:14
        volumes:
          - ./app:/usr/src/app
        networks:
          - red-backend
        working_dir: /usr/src/app
        command: node index.js

      db:
        image: postgres
        environment:
          POSTGRES_PASSWORD: ejemplo123
        volumes:
          - datos-db:/var/lib/postgresql/data
        networks:
          - red-backend

    volumes:
      datos-db:

    networks:
      red-backend:
        driver: bridge
    ```

    Explicación del ejemplo
    - **app**: Servicio que ejecuta una aplicación Node.js, montando el código fuente desde el host.
    - **db**: Servicio de base de datos PostgreSQL con persistencia de datos usando volúmenes.
    - **volumes**: Define un volumen llamado `datos-db` para guardar los datos de PostgreSQL.
    - **networks**: Crea una red personalizada `red-backend` para que los servicios se comuniquen de forma aislada.


---

## Repositorios útiles con ejemplos

- [Ejemplos variados de docker-compose.yml](https://github.com/johnsanabria/docker-compose_by_example)
- [Guía rápida con ejemplos anotados](https://www.glukhov.org/es/post/2025/07/docker-compose-cheatsheet/)
- [Uso de volúmenes en Docker Compose](https://kinsta.com/es/blog/volumenes-docker-compose/)

