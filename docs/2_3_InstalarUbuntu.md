---
title: 2.3. Instalación en Ubuntu
description: Pasos para la instalación y configuración de Odoo usando Ubuntu Server
---

# Instalar en Debian y Ubuntu

Antes de nada, hay que preparar un poco el sistema:

En el caso de Ubuntu o Debian, que es el que nos interesa, Odoo proporciona unos repositorios llamados **Nightly**, que pueden ser añadidos al **sources.list** para instalar de manera automática todo. Estos repositorios se actualizan cada noche, por lo que es posible que, con el tiempo, algunas funciones o archivos cambien si actualizamos.

En principio, todo debería funcionar como indican los manuales, pero si necesitamos utilizar utf-8 por el idioma, hay que hacer algunos pasos previos.

Es posible que Debian o Ubuntu no tenga bien configurados los **locales**. Se puede hacer con:

```bash
dpkg-reconfigure locales
```

Y seleccionar los de es_ES y el de UTF8 por defecto. Es necesario cerrar la sesión y volver a entrar.

Si `dpkg-reconfigure` no muestra un asistente, puedes hacer:

```bash
locale-gen "es_ES.UTF-8"
dpkg-reconfigure locales
```

Enlace a los repositorios: <https://nightly.odoo.com/>

Y como indica el propio manual, se puede hacer todo con estos comandos (el primero si estamos en Debian):

```bash
sudo apt-get install ca-certificates
wget -O - https://nightly.odoo.com/odoo.key | sudo gpg --dearmor -o /usr/share/keyrings/odoo-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/odoo-archive-keyring.gpg] https://nightly.odoo.com/16.0/nightly/deb/ ./' | sudo tee /etc/apt/sources.list.d/odoo.list
sudo apt-get update && sudo apt-get install odoo
```

Estos comandos instalan los certificados que los navegadores o, en este caso, `wget` necesitan para admitir HTTPS, después descargan la clave, añaden el repositorio e instalan Odoo.

A continuación, hay que ir a la dirección en el navegador:

```
http://<ip o url>:8069
```

[Asciinema. Video del proceso](https://asciinema.org/a/122875).


## Configuración de la ruta de los módulos

Como la instalación de Odoo crea el usuario **odoo**, que es el que debemos utilizar para el desarrollo, vamos a asignarle una contraseña y le pondremos como shell `bash`:

```bash
sudo passwd odoo
sudo usermod -s /bin/bash odoo
```

!!! warning
     A partir de este momento, todos los comandos deben ejecutarse con el usuario *odoo*.
     O nos logeamos como usuario *odoo* o mejor hacemos `su odoo`


La configuración del servidor Odoo tiene una opción llamada **addons-path**. Podemos añadir más rutas para nuestros addons personalizados. Se puede dejar de forma definitiva en el archivo de configuración o iniciar el servidor indicando la ruta de los addons:

```bash
odoo -d demodb --addons-path="<ruta>"
```

Si queremos que quede guardado de forma definitiva, hay que añadir **--save** al comando. Los comandos serían, por tanto:

```bash
mkdir /var/lib/odoo/modules
odoo scaffold pruebas /var/lib/odoo/modules
odoo --addons-path="/var/lib/odoo/modules,/usr/lib/python3/dist-packages/odoo/addons" --save
```

!!! tip

     La opción --save guarda la configuración en `$HOME/.odoorc`, que es un archivo para el usuario odoo. Si queremos que sea para todos los usuarios que puedan ejecutar el servidor odoo, se puede poner en el archivo de `/etc/odoo`.

## Depurar Odoo
Para crear módulos o ver los problemas que están ocurriendo, es necesario leer los archivos de log, pero existe una manera más eficiente de hacerlo. Si observamos el comando que realmente está ejecutando Odoo:

```bash
python3 /usr/bin/odoo --config /etc/odoo/odoo.conf --logfile /var/log/odoo/odoo-server.log
```

Podemos ver que la opción `--logfile` envía la salida a un archivo. Si detenemos el servicio con:

```bash
systemctl stop odoo
```
o
```bash
/etc/init.d/odoo stop
```

Podemos iniciar sesión con el usuario `odoo` (es necesario que pueda iniciar sesión en Linux) y ejecutar:

```bash
odoo --config /etc/odoo/odoo.conf
```

De esta manera, los mensajes del servidor aparecen en tiempo real en la consola.

Si además queremos actualizar un módulo al arrancar, podemos especificar la base de datos y el módulo a actualizar:

```bash
odoo --config /etc/odoo/odoo.conf -u modulo -d empresa
```

Puede que nuestro usuario `odoo` tenga una configuración personalizada. En ese caso, habría que hacer, por ejemplo:

```bash
odoo --config /var/lib/odoo/.odoorc -d empresa -u modulo
```

De hecho, una vez hecho el `--save`, cada vez que ejecutamos el comando **odoo**, busca el archivo `.odoorc` en el directorio personal del usuario. Por tanto, solo hay que hacer:

```bash
odoo -d empresa -u modulo
```

Además, podemos modificar el nivel de log con la opción `--log-level`, por ejemplo: `--log-level=debug`.

> [Asciinema con todos los pasos para depurar.](https://asciinema.org/a/122901)

Para saber más, puedes consultar la ayuda:

```bash
odoo --help
```

O la documentación oficial:
<https://www.odoo.com/documentation/12.0/reference/cmdline.html>

## Mejorar la salida de Log

Para añadir salida de log en nuestros métodos y facilitar la depuración, se puede utilizar la API de Odoo:

Al principio del archivo `.py`:

```python
from openerp import models, fields, api
import logging

_logger = logging.getLogger(__name__)
```

Dentro de las funciones:

```python
_logger.debug("Usa _logger.debug para depuración, solo para este propósito.")
_logger.info("Usa _logger.info para mensajes informativos. Se utiliza para notificar algo importante.")
_logger.warning("Usa _logger.warning para problemas menores, que no harán fallar tu módulo.")
_logger.error("Usa _logger.error para informar de una operación fallida.")
_logger.critical("Usa _logger.critical para mensajes críticos: si esto aparece, el módulo dejará de funcionar.")
# ¿Quieres incluir datos de tu campo? Pásalos en el contexto, obténlos del pool o del diccionario.
_logger.critical("El nombre '" + str(record.get('name')) + "' no es válido para nosotros.")
```

## Opciones de log

Por defecto, Odoo envía su log a un archivo en **/var/log/odoo/**, pero se puede redirigir a otro archivo con **--log-file=LOGFILE**.

Si queremos más detalle en determinadas acciones de Odoo, podemos añadir al comando las siguientes opciones:

- **--log-request**: Muestra las peticiones **RPC** (remote procedure call) hechas por http desde el cliente.
- **--log-response**: Muestra el contenido de la respuesta que da el servidor a las peticiones anteriores. Muy útil para saber qué está enviando y cómo lo interpreta el cliente.
- **--log-web**: Da más detalles de todas las peticiones GET o POST que se hacen a la web.
- **--log-sql**: Muestra el SQL que lanza al servidor PostgreSQL. Esta opción ayuda a entender cómo funciona el ORM.
- **--log-level=LOG_LEVEL**

```python
['info', 'debug_rpc', 'warn', 'test', 'critical', 'debug_sql', 'error', 'debug', 'debug_rpc_answer', 'notset']
```

## Modo debug

Odoo permite entrar en modo debug con **--debug**.

También se puede importar la biblioteca **pdb** y colocar un *trace* en el código que nos interesa:

```python
import pdb
...
pdb.set_trace()
```

Una vez dentro, se pueden utilizar los comandos de pdb:  
<https://docs.python.org/3/library/pdb.html>

## Poner en producción en Ubuntu

[https://www.odoo.com/documentation/17.0/administration/install/deploy.html?highlight=workers](
https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot-with-systemd)

## Odoo por HTTPS

El servidor Odoo, por defecto, ofrece su web en el puerto 8069 y en HTTP, es decir, sin capa de seguridad SSL.

Para dotarlo de esa seguridad, necesitamos utilizar un servidor web que actúe como proxy y proporcione la conectividad por HTTPS.

Situación inicial:

```
 ------------               -----------
|            |        8069|            |
|  Cliente   |<---------->|  Servidor  |
|            |            |   Odoo     |
 ------------              ------------
```

Situación que buscamos:

```
 ------------              -------------------
|            |        443 |                   |
|  Cliente   |<---------->|   Nginx <---┐     |
|            |            |              |    |
 ------------             |              |8069|
                          |              v    |
                          |            ------ |
                          |            |Odoo| |
                          |            ------ |
                           -------------------
```

Todos los servicios que utilizan SSL necesitan un certificado. En una situación ideal, disponemos de un certificado firmado por una entidad certificadora. Si no es así, debemos crear uno autosignado.

[Tutorial para crear el certificado](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04)

```bash
    # openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/odoo-selfsigned.key -out /etc/ssl/certs/odoo-selfsigned.crt
    Generating a 2048 bit RSA private key
    ...........................................................................................................+++
    ...........................................................................................+++
    writing new private key to '/etc/ssl/private/odoo-selfsigned.key'
    -----
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:ES
    State or Province Name (full name) [Some-State]:Valencia
    Locality Name (eg, city) []: Xàtiva
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:
    Organizational Unit Name (eg, section) []:
    Common Name (e.g. server FQDN or YOUR name) []: nom o domini
    Email Address []: correu@servidor.com
```


A continuación se debe configurar el https en Nginx para hacer el proxy en Odoo. [manual oficial](https://www.odoo.com/documentation/9.0/setup/deploy.html#https)

En /etc/odoo.conf:

```js
proxy_mode = True 
```
En /etc/nginx/sites-enabled/odoo.conf:

```bash

    #odoo server
    upstream odoo {
     server 127.0.0.1:8069;
    }
    upstream odoochat {
     server 127.0.0.1:8072;
    }
    # S'han definit els upstream a localhost i als port determinats

    # http -> https (totes les peticions per HTTP se reformulen a HTTPS)
    server {
       listen 80;
       server_name _;                            
       # Si tinguerem nom de domini el ficariem, en altre cas: _
       rewrite ^(.*) https://$host$1 permanent;
    }

    server {
     listen 443;
     server_name _;
     # La _ perquè en l'exemple no tenim domini, com dalt
     proxy_read_timeout 720s;
     proxy_connect_timeout 720s;
     proxy_send_timeout 720s;

     # Add Headers for odoo proxy mode
     proxy_set_header X-Forwarded-Host $host;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header X-Forwarded-Proto $scheme;
     proxy_set_header X-Real-IP $remote_addr;

     # SSL parameters
     ssl on;
     ssl_certificate /etc/ssl/certs/odoo-selfsigned.crt;
     ssl_certificate_key /etc/ssl/private/odoo-selfsigned.key ;
     # IMPORTANT: ficar bé les rutes dels certificats
     ssl_session_timeout 30m;
     ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
     ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
     ssl_prefer_server_ciphers on;

     # log
     access_log /var/log/nginx/odoo.access.log;
     error_log /var/log/nginx/odoo.error.log;

     # Redirect requests to odoo backend server
     location / {
       proxy_redirect off;
       proxy_pass http://odoo;
     }
     location /longpolling {
         proxy_pass http://odoochat;
     }

     # common gzip
     gzip_types text/css text/less text/plain text/xml application/xml application/json application/javascript;
     gzip on;
    }
```

También es necesario borrar el **default** de sites-enabled de nginx o modificarlo para que no afecte al puerto 80 de Odoo.

Ahora se reinician tanto Odoo como nginx.

# Seguridad en Odoo

Cuando hablamos de seguridad en Odoo nos referimos tanto a cuestiones generales que afectan a cualquier aplicación web como a aspectos específicos de Odoo. En general, cualquier aplicación web debe cumplir unos mínimos de seguridad, como ofrecer el servicio solo por HTTPS, controlar los puertos, evitar ataques DDOS, utilizar contraseñas seguras, etc.

En cuanto al HTTPS, en los apartados correspondientes se explica cómo configurarlo, aunque lo ideal sería contratar un certificado válido. La seguridad y alta disponibilidad del servidor es un tema complejo que excede el alcance de este módulo y puede ser tarea del ciclo de ASIX. Sin embargo, sí podemos realizar acciones específicas en Odoo para evitar problemas o recuperarnos rápidamente de ellos.

## Persistencia de los datos

Una empresa no puede permitirse perder datos. Existen múltiples formas de evitar la pérdida de datos en Odoo. Si tenemos una instalación on-premise, debemos controlar todos los factores, tanto físicos como lógicos. Esto implica varias tareas imprescindibles:

### Copias de seguridad

Odoo, en su interfaz gráfica, permite exportar tablas y realizar copias de seguridad manuales de tablas individuales. Esto, por supuesto, solo es recomendable para exportaciones o importaciones puntuales.

En la interfaz gráfica también se puede acceder al gestor de bases de datos y exportar o importar el backup. Sería recomendable hacerlo periódicamente. Si queremos que sea automático, se puede programar externamente un servicio que, cada cierto tiempo, se conecte de forma remota por XML-RPC:

```python
import requests

# Datos de conexión
odoo_host = 'https://tuservidorodoo.com'  # o http://localhost:8069 si es local
database = 'tu_basededatos'
admin_password = 'tu_contraseña_admin'

# URL para hacer backup
url = f'{odoo_host}/web/database/backup'

# Datos para la solicitud POST
payload = {
	'master_pwd': admin_password,
	'name': database,
	'backup_format': 'zip'  # o 'dump'
}

# Realizar la solicitud POST
response = requests.post(url, data=payload)

# Verificar la respuesta y guardar el archivo si es válida
if response.status_code == 200:
	filename = f"{database}_backup.zip"
	with open(filename, 'wb') as f:
    	f.write(response.content)
	print(f"Backup guardado como {filename}")
else:
	print(f"Error al hacer backup: {response.status_code} - {response.text}")
```

También podemos hacerlo a nivel de la línea de comandos de la base de datos:

```bash
pg_dump db_name
```

La copia de seguridad de la base de datos no incluye los archivos y fotos. Será necesario copiar el directorio `filestore` si realizamos la copia a nivel de base de datos.

A un nivel más bajo, se puede hacer una copia de seguridad del sistema de archivos o incluso de las particiones.

Es importante destacar que dicha copia de seguridad no debe almacenarse en el mismo disco duro que la base de datos original ni en la misma ubicación física.

## Alta disponibilidad

Un sistema empresarial debe estar alojado en un servidor seguro a nivel físico. Esto implica el uso de SAIs y RAIDs o sistemas similares. Si estamos utilizando un VPS en la nube, normalmente no tendremos que preocuparnos de esto. En caso contrario, necesitaremos un CPD, aunque sea sencillo, con seguridad física, sistemas de alimentación ininterrumpida y discos redundantes, además de sistemas de copias de seguridad remotas. El sistema debería poder recuperarse de un fallo sin interrumpir el servicio.

## Usuarios y permisos

Odoo cuenta con un sistema complejo de usuarios, grupos, roles y permisos. Un administrador de Odoo debe gestionar de forma precisa estos permisos. Además, debemos distinguir los diferentes tipos de usuarios que hay que gestionar, de mayor a menor nivel de privilegio:

- Root del sistema operativo: Usuario con control total sobre el sistema operativo; debería ser un administrador de sistemas.
- Administrador de PostgreSQL: Tiene control total sobre los datos de toda la empresa y posiblemente de varias empresas. Si PostgreSQL se utiliza para otros fines además de Odoo, también tiene poder sobre ellos.
- Administrador de bases de datos de Odoo: Su contraseña está en `odoo.conf` y puede crear, borrar y hacer copias de todas las bases de datos. Accede normalmente vía web. Es posible que los programadores no necesiten este nivel de acceso.
- Administrador técnico de una base de datos: Puede administrar módulos y cambiar la interfaz. Los programadores suelen necesitar este nivel de permisos.
- Administrador de la empresa: Puede administrar todo lo relativo al negocio, pero no puede instalar módulos ni programar. Normalmente son los propietarios o responsables de la empresa. No es recomendable que una persona sin experiencia en programación tenga más permisos.
- Usuarios normales: Vendedores, administrativos, etc. Pueden acceder a ciertas partes del backend. Sus permisos dependen del grupo o rol al que pertenezcan.
- Clientes y proveedores: Normalmente tienen acceso a la página web, que puede estar hecha con Odoo. También pueden tener acceso a una API desarrollada por nosotros si queremos automatizar las relaciones comerciales con ellos.

# Creación de una base de datos

!!! Tip

     En general no cal fer aquest pas i és recomanable fer la base de dades per la interfície web.

En el usuario de odoo, creamos una base de datos y le aplicamos el esquema de datos de Odoo:

```bash
createdb --encoding=UTF-8 --template=template0 testdb
odoo -d testdb
```

Esto crea una base de datos con los datos de prueba para empezar a trabajar.

Por defecto, el usuario será admin con contraseña admin.

[Comandos básicos de postgreSQL](https://gist.github.com/Kartones/dd3ff5ec5ea238d4c546)

# Errores documentados

!!! Tip

**Importante** Antes de ejecutar estos comandos, consulta el final del archivo de log, generalmente en /var/log/odoo/odoo-server.log


## Error con el rol Odoo

Si aparece un error similar a:

`OperationalError: FATAL:  no existe el rol «odoo»`  
`OperationalError: FATAL:  role "odoo" does not exist`

Hay que ejecutar el comando:

```bash
su - postgres -c "createuser -s odoo"
```

Esto crea el usuario odoo con permiso de superusuario (-s).

## Error con UTF-8

Muchas veces, al instalar, no se configura el *template0* de la base de datos en utf-8.  
[1](http://www.postgresql.org/docs/9.3/static/manage-ag-templatedbs.html)

Se soluciona borrando y volviendo a crear la base de datos template1 utilizando la codificación UTF8.  
[2](https://gist.github.com/ffmike/877447)

```bash
# su postgres
psql
postgres=# update pg_database set datallowconn = TRUE where datname = 'template0';
postgres=# \c template0
template0=# update pg_database set datistemplate = FALSE where datname = 'template1';
template0=# drop database template1;
template0=# create database template1 with template = template0 encoding = 'UTF8';
template0=# update pg_database set datistemplate = TRUE where datname = 'template1';
template0=# \c template1
template1=# update pg_database set datallowconn = FALSE where datname = 'template0';
```

Puede que no se cree el clúster de postgresql. Primero hay que reconfigurar los locales y después:

```bash
pg_createcluster 9.4 main --start
```

## Recuperar la contraseña del administrador

**Del administrador de una base de datos:** Dentro de la base de datos:

```sql
update res_users set password='test' where login='admin';
```

**Del administrador de Odoo**

Si no puedes administrar o crear nuevas bases de datos, hay que modificar la línea **admin_passwd** de /etc/odoo/odoo.conf o .odoorc, dependiendo de qué archivo de configuración estés usando.

## Problemas en los repositorios oficiales de ubuntu

En el caso del IES, los repositorios oficiales no funcionan bien por alguna interferencia con los de Lliurex. Hay que cambiarlos, por ejemplo, por los de Caliu. Una manera es entrar en **vim** y ejecutar este comando:

```
:%s_http://archive.ubuntu.com/ubuntu_http://ftp.udc.es/ubuntu/_
```

(Usamos _ en lugar de / en la sustitución porque la / ya está en las URL.)

## No conecta con PostgreSQL

Puede ser porque el servicio no está en funcionamiento.

```bash
service postgresql [restart,start,stop,status]
```

En caso de que falle, podemos ver el log:

```bash
cat /var/log/postgresql/(version)...
```

A veces la base de datos queda corrupta. Se puede intentar recuperar con:

```bash
su - postgres -c '/usr/lib/postgresql/12/bin/pg_resetwal -f /var/lib/postgresql/12/main'
```

Si se han perdido datos, no son muy importantes o tenemos copia de seguridad, tal vez hay que eliminar la base de datos de la empresa que no funciona:

```bash
postgres$ psql
psql> drop database <nombre_base_de_datos>;
```

