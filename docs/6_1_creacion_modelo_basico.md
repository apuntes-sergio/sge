---
title: Creación de un modelo básico, vistas, menús y actions
description: Ejemplo guiado de desarrollo de módulo de gestión de proyectos en Odoo
---

# Creación de un Modelo Básico

## Introducción

En esta sección vamos a realizar de forma guiada un ejemplo de desarrollo de un módulo que nos permitirá gestionar la realización de proyectos de desarrollo informático.

Básicamente, los elementos que vamos a tener en cuenta para nuestro módulo son los siguientes:

- **Proyectos**: Cada desarrollo de una aplicación diferente será un proyecto
- **Tareas**: El desarrollo se basa en la realización de tareas
- **Sprints**: Dividimos la fase de desarrollo en sprints temporales que agrupan una serie de tareas a realizar
- **Desarrolladores**: Son las personas que realizan cada una de las tareas del proyecto

## Creación de la Aplicación

Para comenzar, puesto que ya tenemos el sistema configurado, procedemos a la creación de la estructura de la aplicación mediante **scaffold**.

Debemos tener claro el nombre de nuestra aplicación que será **gestion_tareas_sergio** en mi caso.

```bash
docker exec -it odoo_dev_dam odoo scaffold gestion_tareas_sergio /mnt/extra-addons
```

Comprobamos que se ha generado toda la estructura:

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/01_Creación.png){ width="75%" }
  <figcaption>Creación de la estructura del módulo</figcaption>
</figure>

!!! tip "Permisos"
    En caso de tener problemas de permisos, recuerda añadir permisos a todos los elementos:
    
    ```bash
    docker exec -it odoo_dev_dam chmod 777 -R /mnt/extra-addons/gestion_tareas_sergio
    ```

Ahora configuramos el `__manifest__.py` para dar descripción de nuestra aplicación y lo vamos a instalar para comprobar que todo funciona correctamente.

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/02_PrimeraInstalacion.png){ width="70%" }
  <figcaption>Comprobación de que la aplicación está instalada correctamente</figcaption>
</figure>

Estaría bien crear un logotipo para nuestra aplicación y así identificarla rápidamente.

!!! tip "Añadir un Logo a tu Módulo"
    Para que tu módulo tenga un icono personalizado:
    
    1. Crea la carpeta: `static/description/`
    2. Coloca una imagen PNG de 48x48 o 256x256 píxeles llamada exactamente `icon.png`
    3. **Importante**: El logo solo se carga al instalar, no al actualizar
    4. Si cambias el logo: **Desinstala → Reinicia → Reinstala** el módulo

    ```bash
        # Crear estructura dentro del contenedor
        mkdir -p static/description
        
        # Copiar tu imagen desde el exterior del contenedor
        docker cp icon.png /mnt/extra-addons/HolaMundo/static/description/icon.png
        
        # Ajustar permisos dentro del contenedor si es preciso
        chown ubuntu:ubuntu icon.png
    ```
    
!!! info "¿Reiniciar o actualizar?"
    **Debes reiniciar el servidor de Odoo cuando**:
    
    - Modificas **modelos Python** (`models/*.py`), ya que el código se carga al iniciar el servidor
    - Cambias el archivo `__manifest__.py`, porque Odoo lo lee al cargar los módulos
    
    **Puedes actualizar el módulo sin reiniciar el servidor si**:
    
    - Solo modificas **vistas XML** (formularios, listas, etc.)
    - Cambias **archivos de datos estáticos** (`views/*.xml`, `security/*.xml`, etc.)
    - Ajustas **controladores web** (`controllers/*.py`), aunque en algunos casos puede requerir reinicio
    
    En ocasiones, si los cambios realizados en los módulos son muy grandes, la aplicación puede fallar y es mejor **desinstalar** y volver a **instalar** la aplicación, aunque esto conlleve la pérdida de datos.

## Creación de Módulo Básico de Tareas

Comenzaremos creando un módulo básico que contendrá información de las **tareas** en las que se dividirá nuestro desarrollo completo.

### Añadir el Modelo

Lo primero de todo será ir al fichero `models/models.py` y comenzar a modificar el fichero existente para crear el modelo según nuestras necesidades.

Recordad que tenemos por una parte `__init__.py` donde se importan los modelos que hay en `./models`, y que aquí tenemos otro `models/__init__.py` que nos indica los ficheros modelo concretos a importar, en este caso `models/models.py`.

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/03_ArquitecturaModelos.png){ width="60%" }
  <figcaption>Arquitectura de importación de modelos</figcaption>
</figure>

En principio, los campos que vamos a necesitar van a ser los siguientes:

- Nombre de tarea, de tipo texto corto (`fields.Char()`)
- Descripción, de tipo texto largo (`fields.Text()`)

Así pues, comencemos modificando el modelo, para lo cual damos un nombre a la clase de la tarea y le asignamos un nombre y una descripción.

El nombre que le vamos a dar a la clase es `tareas_sergio` en mi caso, y después en el `_name` debemos especificar de nuevo este nombre precedido de forma correcta por el nombre de la aplicación.

??? note "models.py"
    ```python
    from odoo import models, fields, api

    class tareas_sergio(models.Model):
        _name = 'gestion_tareas_sergio.tareas_sergio'
        _description = 'Modelo de Tareas para Gestión de Proyectos'

        nombre = fields.Char()
        descripcion = fields.Text()
    ```

Intenta hacer esta parte sin mirar la solución.

Vamos a comenzar poco a poco, de forma que tras cada modificación vamos a actualizar la aplicación para comprobar que todo funciona correctamente. **Si hacemos muchos cambios a la vez sin actualizar ni verificar, nos encontraremos con errores arrastrados que serán muy difíciles de solucionar.**

### Revisión de la Base de Datos

En Odoo, la estructura de las tablas creadas depende del modelo definido en el sistema. Odoo utiliza el **ORM** (Object-Relational Mapping) de Python para mapear clases a tablas en la base de datos PostgreSQL.

Así pues, una vez instalado el módulo, si no hemos tenido ningún problema, podemos comprobar cómo ha trabajado este **ORM** y cómo debe haberse creado el modelo, por tanto debe haber cambios en la base de datos.

Vamos a revisar estos cambios y comprobar que todo cambio en el modelo repercute en un cambio en las tablas.

Para comprobar los cambios realizados en la base de datos, tenemos tres opciones:

- Desde el propio interfaz de Odoo en modo desarrollo
- Desde la línea de comandos usando el interfaz de PostgreSQL
- Desde una herramienta de administración de base de datos con interfaz gráfico como [DBeaver](https://dbeaver.io/)

#### Desde el Interfaz de Odoo

Para revisar los modelos desde el interfaz de Odoo, una vez en modo desarrollo, accedemos a **Ajustes → Técnico** y ahí tenemos una sección dedicada a la **Estructura de la base de datos**:

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/04_ModelosInterfazOdoo.png){ width="70%" }
  <figcaption>Revisión de base de datos desde Odoo</figcaption>
</figure>

Aquí podemos ver un listado de todas las tablas tanto del propio sistema Odoo como de todas las aplicaciones que tenemos instaladas.

En Odoo, el nombre de la tabla que se crea en la base de datos se determina principalmente por el atributo `_name` del modelo:

- Si defines `_name = 'mi.modelo'`, la tabla se llamará `mi_modelo`
- Odoo reemplaza los puntos (`.`) por guiones bajos (`_`) para formar el nombre de la tabla

Si buscamos el modelo que hemos creado, en mi caso `tareas_sergio`, hacemos clic sobre él y podemos ver sus características así como los campos que tiene:

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/05_ModelosInterfazOdoo2.png){ width="60%" }
  <figcaption>Revisión de base de datos desde Odoo - Campos del modelo</figcaption>
</figure>

Como se puede ver en la figura anterior, además de los dos campos que hemos creado para el modelo, Odoo crea automáticamente una serie de campos para llevar control de versiones y añadir sus propios índices.

#### Desde la Línea de Comandos

Como somos apasionados de los sistemas, nos gusta comprobar que todo funciona desde el terminal, por lo que podemos entrar en el interfaz de PostgreSQL y desde ahí revisar tanto la existencia de la tabla asociada al modelo como sus campos.

Para ello, entramos en el terminal del contenedor:

```bash
docker exec -it postgres_dev_dam bash
```

Accedemos al interfaz de comando de PostgreSQL:

```bash
psql -U odoo
```

Una vez en el interfaz, comprobamos las bases de datos existentes y cambiamos a la correspondiente:

```sql
\l
\c nombre_base_datos
```

Buscamos la tabla con una de las siguientes opciones:

```sql
\dt *sergio*
```

o con SQL:

```sql
SELECT * FROM pg_tables WHERE tablename LIKE '%sergio%';
```

En el primer caso, el `*` (asterisco) sirve como comodín.

Finalmente, listamos los campos de la tabla:

```sql
\d gestion_tareas_sergio_tareas_sergio
```

o con SQL:

```sql
SELECT column_name, data_type 
FROM information_schema.columns 
WHERE table_name = 'gestion_tareas_sergio_tareas_sergio';
```

Para salir del interfaz de PostgreSQL:

```sql
\q
```

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/06_ModelosInterfazOdoo3.png){ width="90%" }
  <figcaption>Revisión de base de datos - Campos del modelo desde interfaz PostgreSQL</figcaption>
</figure>

#### Desde DBeaver

También podemos utilizar esta aplicación de gestión de base de datos para acceder y comprobar las tablas y campos de nuestro modelo.

Para ello, tras instalar la aplicación, creamos una nueva conexión a la base de datos. Como hemos expuesto el puerto del contenedor en la definición del `docker-compose.yml`, podemos acceder como si se tratara de una base de datos en local.

Creamos y configuramos una conexión:

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/07_dbeaver_Config.png){ width="75%" }
  <figcaption>Configuración de conexión en DBeaver</figcaption>
</figure>

Si todo funciona bien, podemos ver también la tabla y su estructura:

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/08_dbeaver_Tabla.png){ width="80%" }
  <figcaption>Tabla y su estructura en DBeaver</figcaption>
</figure>

## Vistas, Menú y Actions

Ahora que ya tenemos el modelo definido y lo hemos revisado en la base de datos, es momento de continuar trabajando con nuestro módulo y hacer que sea visible desde el menú de Odoo.

Para ello, tal y como hemos visto en el ejemplo de "Hola Mundo V3", tenemos que:

1. Definir las vistas que queremos publicar
2. Mostrar una opción en el menú para poder ver estas vistas
3. Activar las acciones para dar funcionalidad al menú

### Configurando Vistas

De momento, este modelo no es accesible desde ningún sitio: no tenemos ninguna vista. Por defecto, Odoo crea unas vistas básicas si no haces nada, pero vamos a ver dónde están y cómo personalizarlas.

Las vistas están en el directorio `views/`, en un documento XML.

En este fichero comentado tenemos algunos bloques:

- Uno de vista
- Uno de action (que veremos más adelante)
- Uno de server action
- Uno de menús

Vamos a ir viéndolos poco a poco, porque se trata de ir definiendo cada uno de esos bloques.

Realmente, un bloque `<record>` implica que va a ser un registro en la base de datos. Este registro necesita saber dónde se va a guardar, en el primer caso en el modelo `ir.ui.view` (las vistas también son modelos en Odoo).

Así pues comenzamos descomentando el primer bloque y ponemos un `id` único, en mi caso, por ejemplo `gestion_tareas_sergio.list`, que será la vista en modo **lista** (anteriormente Tree) de las tareas.

Establecemos un nombre (no es lo más importante, pero hay que establecerlo), por ejemplo `Gestion Tareas Sergio list`.

Después establecemos el **modelo** que debe coincidir con el nombre del modelo que hemos establecido al definirlo, en mi caso `gestion_tareas_sergio.tareas_sergio`. Esto es fundamental.

A continuación, añadimos el listado de campos que mostrará la vista. En este caso, solo tenemos dos campos creados: `nombre` y `descripcion`. De nuevo son los mismos que hemos establecido en el modelo.

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/09_DefiniendoVistas.png){ width="80%" }
  <figcaption>Definiendo la vista de un modelo</figcaption>
</figure>

Este XML indica que es una vista de tipo **list** y que mostrará los campos especificados del modelo. Estos campos deben existir en el fichero Python, como hemos visto antes.

??? note "Código de la vista"
    ```xml
    <!-- Vista de Lista -->
    <record model="ir.ui.view" id="gestion_tareas_sergio.list">
      <field name="name">Gestion Tareas Sergio list</field>
      <field name="model">gestion_tareas_sergio.tareas_sergio</field>
      <field name="arch" type="xml">
        <list>
          <field name="nombre"/>
          <field name="descripcion"/>
        </list>
      </field>
    </record>
    ```

!!! info "Revisando la vista definida"
    Una vez definida la vista, ahora deberíamos poder encontrarla si consultamos desde la aplicación el listado de vistas. Si actualizamos el módulo, la podremos encontrar. En este caso, no hemos tenido que reiniciar el servicio.
    
    <figure markdown="span" align="center">
      ![Image title](./imgs/ejemploT/10_ListadoDeVistas.png){ width="80%" }
      <figcaption>Listado de vistas disponibles</figcaption>
    </figure>

### Actions

La vista aún es inaccesible porque no hay ningún menú que permita acceder a ella. Para relacionarlo y ver una lista de tareas, vamos a activar el action correspondiente.

Un **action** permite abrir, cuando se relacione con una opción de menú, la vista correspondiente. Será un registro del modelo de acciones de ventana (`ir.actions.act_window`).

Continuamos pues descomentando el siguiente bloque de código y asignando un `id` único, por ejemplo `gestion_tareas_sergio.action_window`. Le ponemos un nombre, indicamos el modelo sobre el que actúa y el orden en el que queremos que se muestren las vistas (primero la **list** que acabamos de crear y después la **form**, que Odoo generará automáticamente si no la hemos definido).

??? note "Código del action"
    ```xml
    <record model="ir.actions.act_window" id="gestion_tareas_sergio.action_window">
      <field name="name">Gestión de Tareas</field>
      <field name="res_model">gestion_tareas_sergio.tareas_sergio</field>
      <field name="view_mode">list,form</field>
    </record>
    ```

Una acción relaciona un menú o botón con una acción que se desencadena en el cliente y se convierte en una petición al servidor. En este caso, el action se transforma en una petición que demanda al servidor las vistas **list** y **form** del modelo de tareas, y el servidor devolverá la vista **list** creada y una vista **form** generada automáticamente.

### Menús

Para que este action funcione, el cliente debe tener un menú.

Así que seguimos descomentando el bloque de código de menú y vamos a crear una opción de menú principal, por ejemplo `Gestión Tareas Sergio`, y de ahí colgará un submenú llamado `Gestión`, y finalmente otro llamado `Tareas`. El action del menú `Tareas` será el que hemos definido antes, para que al pulsar se cargue la vista de tareas.

La `action` de este menú será el `id` del `ir.actions.act_window` que hemos definido anteriormente.

Por una parte tenemos el código:

??? note "Bloques de menú"
    ```xml
    <!-- Top menu item -->
    <menuitem name="Gestión Tareas Sergio" id="gestion_tareas_sergio.menu_root"/>

    <!-- menu categories -->
    <menuitem name="Gestión" id="gestion_tareas_sergio.gestion" 
              parent="gestion_tareas_sergio.menu_root"/>

    <!-- actions -->
    <menuitem name="Tareas" id="gestion_tareas_sergio.gestion_tareas" 
              parent="gestion_tareas_sergio.gestion"
              action="gestion_tareas_sergio.action_window"/>
    ```

Y por otra tenemos el esquema de cómo debe estar todo conectado:

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/11_ManuOrganizacion.png){ width="80%" }
  <figcaption>Estructura de la configuración del menú</figcaption>
</figure>

Ahora ya tenemos definido el menú y antes de seguir sería aconsejable volver a actualizar el módulo para asegurarnos de que no tenemos ningún error y comprobar si sale o no el menú.

Si revisamos, sigue sin salir nuestra aplicación en el menú tras haberlo definido. Esto es debido a un tema de permisos que abordamos en el siguiente punto.

## Permisos

En Odoo, para que un usuario pueda **ver un menú** y **acceder a listas o formularios** de un modelo, se deben cumplir ciertos requisitos de **permisos y configuraciones**.

Se debe definir un archivo XML o CSV con los permisos para el modelo. Esto se hace en un archivo como `security/ir.model.access.csv`.

Cada línea define:

- **Modelo**
- **Grupo**
- **Permisos**: leer (`perm_read`), escribir (`perm_write`), crear (`perm_create`), borrar (`perm_unlink`)

Ejemplo:

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_gestion_tareas_sergio_tareas_sergio,acceso_tareas_sergio,model_gestion_tareas_sergio_tareas_sergio,base.group_user,1,1,1,1
```

Donde cada línea representa un conjunto de permisos para un grupo sobre un modelo. Los campos son:

- **id**: Identificador único del registro (puede ser cualquier nombre único)
- **name**: Nombre legible del permiso, puede ser cualquiera
- **model_id:id**: Hace referencia al modelo al que se aplican los permisos. La forma estándar de referenciarlo es `model_<nombre_del_modelo>`, donde `<nombre_del_modelo>` es el valor de `_name` del modelo, sustituyendo los puntos (`.`) por guiones bajos (`_`). Puede parecer poco intuitivo, pero es el formato que exige Odoo
- **group_id:id**: Grupo de usuarios al que se le asignan los permisos (puede ser `base.group_user`, `base.group_system`, etc.). Si se deja vacío, aplica a todos los usuarios
- **perm_read**: 1 si puede leer, 0 si no
- **perm_write**: 1 si puede modificar, 0 si no
- **perm_create**: 1 si puede crear, 0 si no
- **perm_unlink**: 1 si puede borrar, 0 si no

De esta forma, para que un menú sea visible:

- El usuario debe tener acceso al **grupo** asignado al menú. Para simplificar, de momento van a ser todos los usuarios: `base.group_user`
- El menú debe estar vinculado a una **acción**, aspecto que en nuestro caso ya hemos hecho
- Se debe tener al menos una vista de tipo `list` o `form` para que la acción funcione correctamente

### Configurar el Manifest

En el fichero `__manifest__.py` hay que habilitar estos permisos para que se carguen. Esta línea suele venir comentada por defecto y hay que descomentarla:

```python
'data': [
    'security/ir.model.access.csv',
    'views/views.xml',
],
```

Ahora ya deberíamos verlo todo: reiniciamos el servidor Odoo, actualizamos el módulo y ya aparece la opción de menú principal. Si la cargamos, tenemos `Gestión` y `Tareas`. Al pulsar en `Tareas` aparece la vista con los campos definidos. Si pulsamos en "Nuevo", podemos crear tareas (esto es una vista formulario generada por defecto).

### Verificación Rápida

Para comprobar si todo está bien configurado:

1. Activa el **modo desarrollador**
2. Ve a **Ajustes → Técnico → Seguridad → Reglas de acceso** y **Reglas de registro**
3. Asegúrate de que el usuario pertenece al grupo correcto
4. Verifica que el menú tiene una acción válida y que el modelo tiene permisos definidos

O directamente, podemos ver que ya aparece en el menú de aplicación todos los elementos que hemos definido:

<figure markdown="span" align="center">
  ![Image title](./imgs/ejemploT/12_ManuVista.png){ width="80%" }
  <figcaption>Vista de menú y aplicación funcionando</figcaption>
</figure>

En esta imagen estamos viendo la vista tipo listado con los dos campos, y además se ha accedido al menú y se pueden ver los 3 niveles de menú que hemos definido: **Gestión Tareas Sergio → Gestión → Tareas**

De esta manera, hemos visto cómo crear un módulo básico con funcionalidad básica: un modelo, vistas, permisos, opciones de menú y cómo relacionarlo todo.

---


## 🧩 Tu Turno: Gestor de Restaurante

Ahora que has visto cómo crear un módulo completo de gestión de tareas, es tu turno de aplicar estos mismos conceptos en un contexto diferente. Vamos a crear un **Gestor de Restaurante** que seguirá la misma estructura técnica pero adaptada al mundo de la gastronomía.

### Objetivos y Contexto

Vas a desarrollar un sistema para gestionar un restaurante que permita gestionar platos del menú, organizarlos en menús del día/semanales, controlar ingredientes y gestionar categorías.

En esta primera sesión crearás el módulo básico con el modelo **Platos**, con dos campos sencillos (nombre y descripción), siguiendo los mismos pasos que acabamos de ver con el modelo de Tareas.

**Especificaciones técnicas**:

- **Módulo**: `gestion_restaurante_tunombre`
- **Modelo**: Plato (`platos_tunombre`)
- **Campos**: `nombre` (Char) y `descripcion` (Text)
- **Menú**: Gestión Restaurante → Menú → Platos

### Pasos a Realizar

1. **Crear el módulo con scaffold**
    ```bash
    docker exec -it odoo_dev_dam odoo scaffold gestion_restaurante_tunombre /mnt/extra-addons
    ```

2. **Ajustar permisos**
    ```bash
    docker exec -it odoo_dev_dam chmod 777 -R /mnt/extra-addons/gestion_restaurante_tunombre
    ```

3. **Configurar el `__manifest__.py`**
    - Actualiza el nombre a algo como "Gestión de Restaurante"
    - Actualiza la descripción
    - Asegúrate de que la línea de seguridad está descomentada

4. **Crear el modelo `Plato` en `models/models.py`**
    - Nombre de la clase: `platos_tunombre`
    - `_name`: `'gestion_restaurante_tunombre.platos_tunombre'`
    - `_description`: "Modelo de Platos para Gestión de Restaurante"
    - Campos: `nombre` y `descripcion`

5. **Verificar el modelo en la base de datos**
    - Desde Odoo: Ajustes → Técnico → Modelos
    - Busca tu modelo y verifica los campos
    - Opcional: Comprueba desde PostgreSQL o DBeaver

6. **Configurar la vista en `views/views.xml`**
    - Descomenta y modifica el bloque de vista tipo `list`
    - Asegúrate de que el `id` es único (ej: `gestion_restaurante_tunombre.list`)
    - Indica correctamente el `model`
    - Añade los campos `nombre` y `descripcion`

7. **Configurar el action**
    - Descomenta el bloque `ir.actions.act_window`
    - Ponle un nombre descriptivo (ej: "Gestión de Platos")
    - Indica el modelo correcto
    - View mode: `list,form`

8. **Configurar los menús**
    - Menú principal: "Gestión Restaurante"
    - Submenú: "Menú"
    - Opción: "Platos" (vinculada al action)

9. **Configurar permisos en `security/ir.model.access.csv`**
    ```csv
    id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
    access_gestion_restaurante_tunombre_platos,acceso_platos,model_gestion_restaurante_tunombre_platos_tunombre,base.group_user,1,1,1,1
    ```

10. **Instalar y probar**
    - Reinicia el servidor Odoo
    - Activa modo desarrollador
    - Actualiza lista de aplicaciones
    - Instala tu módulo
    - Verifica que aparece el menú
    - Crea algunos platos de prueba

### Verificaciones

Antes de dar por terminado el ejercicio, comprueba que:

- El módulo se instala sin errores
- Aparece el menú "Gestión Restaurante" en la barra superior
- Al entrar en "Menú → Platos" aparece la vista de lista vacía
- Puedes crear nuevos platos con el botón "Nuevo"
- El formulario muestra los campos `nombre` y `descripcion`
- Puedes guardar platos y aparecen en la lista

### Resultado Esperado

Al finalizar esta sesión deberías tener:

- Un módulo Odoo instalado y funcionando
- Un modelo `Plato` con dos campos básicos
- Una vista de lista mostrando los platos
- Una vista de formulario (generada automáticamente) para crear/editar platos
- Un menú de navegación de tres niveles
- Permisos configurados correctamente

!!!example "Datos de Prueba Sugeridos"

    Para probar tu módulo, crea al menos estos 3 platos:

    1. **Ensalada César**
        - Descripción: "Lechuga romana, pollo a la parrilla, queso parmesano y salsa césar"

    2. **Pizza Margarita**
        - Descripción: "Tomate, mozzarella, albahaca fresca y aceite de oliva"

    3. **Tiramisú**
        - Descripción: "Postre italiano con café, mascarpone y cacao"

!!!tips "Consejos"

    - Haz commits frecuentes en Git mientras desarrollas
    - Si algo no funciona, revisa los logs: `docker logs odoo_dev_dam -f`
    - No olvides reiniciar el servidor después de modificar archivos Python
    - Puedes actualizar el módulo sin reiniciar si solo cambias XML
    - Guarda capturas de pantalla de tu progreso

!!!question "Preguntas para Reflexionar"

    Una vez terminado el ejercicio, reflexiona sobre:

    1. ¿Qué diferencias técnicas hay entre tu módulo de Restaurante y el de Gestión de Tareas?
    2. ¿Por qué es importante la nomenclatura en `_name` del modelo?
    3. ¿Qué pasaría si no configuramos los permisos en `ir.model.access.csv`?
    4. ¿Cómo se relaciona el `id` del action con el atributo `action` del menú?

En la siguiente sesión añadiremos más campos al modelo de Platos (precio, tiempo de preparación, disponibilidad, etc.) y empezaremos a crear relaciones con otros modelos como Menús e Ingredientes.