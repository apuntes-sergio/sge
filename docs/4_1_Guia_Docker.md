---
title: Guía básica de Docker
description: Guía básica de Docker
---

# Guía Completa de Docker para Alumnos de DAM

## Introducción a Docker
Docker es una plataforma que permite desarrollar, enviar y ejecutar aplicaciones dentro de contenedores. Un contenedor es una unidad estándar de software que empaqueta el código y todas sus dependencias para que la aplicación se ejecute de manera rápida y confiable en diferentes entornos.

### Primer paso: Hello World
```bash
# Ejecutar el contenedor hello-world
docker run hello-world
```
Este comando descarga la imagen `hello-world` y la ejecuta, mostrando un mensaje de bienvenida si Docker está correctamente instalado.

## Cómo se utiliza Docker
Para utilizar Docker, primero debes instalarlo en tu sistema operativo. Una vez instalado, puedes comenzar a crear contenedores a partir de imágenes.

```bash
# Verificar la instalación
docker --version

# Ejecutar el demonio de Docker (en sistemas Linux)
sudo systemctl start docker
```

## Comandos Básicos de Docker

- **`docker pull`**
Descarga una imagen desde Docker Hub.
```bash
docker pull ubuntu
```

- **`docker run`**
Ejecuta un contenedor a partir de una imagen.
```bash
docker run -it ubuntu
```

- **`docker ps`**
Lista los contenedores en ejecución.
```bash
docker ps
```

- **`docker ps -a`**
Lista todos los contenedores, incluyendo los detenidos.
```bash
docker ps -a
```

- **`docker stop`**
Detiene un contenedor.
```bash
docker stop <ID o nombre del contenedor>
```

- **`docker rm`**
Elimina un contenedor.
```bash
docker rm <ID o nombre del contenedor>
```

- **`docker rmi`**
Elimina una imagen.
```bash
docker rmi <nombre de la imagen>
```

- **`docker exec`**
Ejecuta un comando dentro de un contenedor en ejecución.
```bash
docker exec -it <contenedor> bash
```

- **`docker logs`**
Muestra los logs de un contenedor.
```bash
docker logs <contenedor>
```

- **`docker inspect`**
Muestra información detallada de un contenedor o imagen.
```bash
docker inspect <contenedor>
```

- **`docker network`**
Gestiona redes de Docker.
```bash
# Crear una red personalizada
docker network create mi_red
```

- **`docker volume`**
Gestiona volúmenes para persistencia de datos.
```bash
# Crear un volumen
docker volume create mi_volumen
```

- **`docker stats`**
Muestra estadísticas de uso de recursos de los contenedores.
```bash
docker stats
```

- **`docker cp`**
Copia archivos entre el host y el contenedor.
```bash
docker cp archivo.txt <contenedor>:/ruta/
```

- **`docker images`**
Lista las imágenes disponibles en el sistema.
```bash
docker images
```

- **`docker build`**
Construye una imagen a partir de un Dockerfile.
```bash
docker build -t mi_imagen .
```

- **`docker tag`**
Etiqueta una imagen.
```bash
docker tag mi_imagen usuario/mi_imagen:1.0
```

- **`docker history`**
Muestra el historial de capas de una imagen.
```bash
docker history ubuntu
```

| Comando | Descripción | Uso de ejemplo |
| --- | --- | --- | 
| `docker build` | Construye una imagen a partir de un Dockerfile | `docker build -t mi-app .` |
| `docker run` | Crea y ejecuta un contenedor desde una imagen | `docker run -p 8080:80 nginx` |
| `docker ps` | Lista los contenedores en ejecución | `docker ps` |
| `docker ps -a` | Lista todos los contenedores (incluidos los detenidos) | `docker ps -a` |
| `docker stop` | Detiene un contenedor en ejecución | `docker stop mi-contenedor` |
| `docker start` | Inicia un contenedor detenido | `docker start mi-contenedor` |
| `docker restart` | Reinicia un contenedor | `docker restart mi-contenedor` |
| `docker rm` | Elimina un contenedor detenido | `docker rm mi-contenedor` |
| `docker rmi` | Elimina una imagen | `docker rmi nginx:latest` |
| `docker images` | Lista todas las imágenes disponibles localmente | `docker images` |
| `docker exec` | Ejecuta un comando dentro de un contenedor activo | `docker exec -it mi-contenedor bash` |
| `docker logs` | Muestra los registros de un contenedor | `docker logs mi-contenedor` |
| `docker pull` | Descarga una imagen desde Docker Hub | `docker pull redis:latest` |
| `docker push` | Sube una imagen al repositorio (requiere login) | `docker push miusuario/miimagen` |
| `docker network ls` | Lista las redes disponibles | `docker network ls` |
| `docker volume ls` | Lista los volúmenes disponibles | `docker volume ls` |
| `docker inspect` | Muestra detalles internos de un contenedor o imagen | `docker inspect mi-contenedor` |
| `docker login` | Inicia sesión en Docker Hub | `docker login` |
| `docker tag` | Etiqueta una imagen con un nuevo nombre | `docker tag nginx:latest miusuario/nginx:v1` |


## Ejemplos Prácticos de Uso

### Crear un contenedor de Ubuntu e instalar software
```bash
docker run -it ubuntu
apt update
apt install curl
```

### Crear una imagen personalizada
```Dockerfile
# Dockerfile
FROM ubuntu
RUN apt update && apt install -y nginx
CMD ["nginx", "-g", "daemon off;"]
```
```bash
docker build -t mi-nginx .
docker run -d -p 8080:80 mi-nginx
```

## Docker Compose


## ¿Qué es Docker Compose y para qué se utiliza
Docker Compose es una herramienta que permite definir y ejecutar aplicaciones multicontenedor. Utiliza un archivo YAML (`docker-compose.yml`) para configurar los servicios, redes, volúmenes y otras opciones necesarias para que una aplicación funcione correctamente.

### Ventajas de Docker Compose
- **Simplicidad**: Un solo archivo para definir toda la infraestructura.
- **Automatización**: Levanta múltiples contenedores con un solo comando.
- **Escalabilidad**: Permite escalar servicios fácilmente.
- **Portabilidad**: El mismo archivo puede usarse en distintos entornos.

### Comandos principales
```bash
# Levantar los servicios definidos en docker-compose.yml
docker-compose up

# Detener y eliminar contenedores, redes y volúmenes creados
docker-compose down
```

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

## Ejemplo con redes personalizadas y volúmenes
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

### Explicación del ejemplo
- **app**: Servicio que ejecuta una aplicación Node.js, montando el código fuente desde el host.
- **db**: Servicio de base de datos PostgreSQL con persistencia de datos usando volúmenes.
- **volumes**: Define un volumen llamado `datos-db` para guardar los datos de PostgreSQL.
- **networks**: Crea una red personalizada `red-backend` para que los servicios se comuniquen de forma aislada.

Este tipo de configuración es ideal para proyectos que requieren múltiples servicios que interactúan entre sí.



Docker Compose permite definir y ejecutar aplicaciones multicontenedor.

### Archivo `docker-compose.yml`
```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: ejemplo123
```

## Ejemplo Completo con Flask y MySQL
```yaml
version: '3'
services:
  web:
    build: ./web
    ports:
      - "5000:5000"
    depends_on:
      - db
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: ejemplo123
```

Estructura de directorios:
```
project/
├── docker-compose.yml
└── web/
    ├── Dockerfile
    └── app.py
```

### Dockerfile para Flask
```Dockerfile
FROM python:3.8
WORKDIR /app
COPY app.py .
RUN pip install flask
CMD ["python", "app.py"]
```

### app.py
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hola desde Flask en Docker!"

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

Perfecto, Sergio 🙌. Vamos a armar ejemplos de `docker-compose.yml` que incluyan **volúmenes**, **redes** y **servicios básicos** directamente disponibles en [Docker Hub](https://hub.docker.com/). Te explico cada parte para que puedas adaptarlos fácilmente a tus proyectos.

---

## 🔧 Ejemplo 1: NGINX + Redis con red personalizada y volúmenes

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

### 🧠 ¿Qué hace?

- **nginx:** sirve archivos estáticos desde `./nginx/html` en el host.
- **redis:** almacena datos persistentes en el volumen `redis_data`.
- Ambos servicios están en la red `appnet`, lo que permite que se comuniquen por nombre de servicio (`redis`, `nginx`).

---

## 🐘 Ejemplo 2: PostgreSQL + Adminer con volúmenes y red

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

### 🧠 ¿Qué hace?

- **PostgreSQL:** base de datos con credenciales configuradas.
- **Adminer:** interfaz web para gestionar la base de datos.
- Ambos comparten la red `backend` y el volumen `pgdata` mantiene los datos persistentes.

---

## 🧁 Ejemplo 3 Simulación de XAMPP con Apache + PHP + MySQL

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

### 🧠 ¿Qué hace?

- **Apache + PHP:** sirve archivos PHP desde `./htdocs`.
- **MySQL:** base de datos con credenciales configuradas.
- Ambos servicios están conectados por la red `xamppnet` y usan volúmenes para persistencia.

---

## 📚 Repositorios útiles con ejemplos

- [Ejemplos variados de docker-compose.yml](https://github.com/johnsanabria/docker-compose_by_example)
- [Guía rápida con ejemplos anotados](https://www.glukhov.org/es/post/2025/07/docker-compose-cheatsheet/)
- [Uso de volúmenes en Docker Compose](https://kinsta.com/es/blog/volumenes-docker-compose/)

