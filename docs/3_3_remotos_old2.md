---
title: Repositorios remotos. Github.
description: Guía básica de Git y GitHub. Repositorios remotos, sincronización y colaboración
---

Hasta ahora hemos trabajado localmente. Pero muchas veces querrás compartir tu trabajo o colaborar con otros mediante repositorios remotos (GitHub, GitLab, Bitbucket…).

## Conceptos clave en Git remoto: remoto, tracking, branch remoto

- **Remoto (remote)**: copia del repositorio en un servidor. Generalmente se llama `origin`.
- **Branch remoto**: referencia local que apunta al estado conocido de una rama en el remoto, por ejemplo `origin/main`.
- **Tracking branch**: rama local enlazada con una rama remota, lo que permite usar `git pull` y `git push` sin parámetros adicionales.



### 1. Remoto (`remote`)

En Git, un “remoto” no es más que un alias para un repositorio alojado en otro lugar, normalmente en un servidor como GitHub. El nombre más habitual es `origin`, aunque puedes tener varios (por ejemplo, `upstream` si estás colaborando con otro proyecto). Este alias permite que tu repositorio local sepa dónde enviar y recibir cambios.

- Es una **referencia a un repositorio alojado en un servidor**, como GitHub, GitLab o Bitbucket.
- No es una copia completa, sino un vínculo que permite sincronizar cambios.
- El nombre por defecto es `origin`, pero puedes tener varios remotos (`upstream`, `backup`, etc.).
- Se configura con:

```bash
git remote add origin https://github.com/usuario/repositorio.git
```
Este comando vincula tu proyecto local con el repositorio remoto. A partir de aquí, puedes sincronizar tu trabajo con el servidor.


### 2. Branch remoto (`remote branch`)

Cuando haces `git fetch`, Git no descarga directamente los archivos, sino que actualiza referencias locales que apuntan al estado actual de las ramas en el remoto. Estas referencias se llaman *ramas remotas* y tienen nombres como `origin/main` o `origin/feature-x`.

- No puedes modificar directamente `origin/main`.
- Es una “foto” del estado de la rama en el servidor.
- Se actualiza con `git fetch`, no con `git pull`.

```bash
git branch -r
```

Este comando te muestra todas las ramas remotas conocidas. Son útiles para saber qué hay en el servidor sin tocar tu copia local.


### 3. *Tracking branch*

Aquí es donde Git se vuelve realmente cómodo. Una *tracking branch* es una rama local que está vinculada a una rama remota. Esto significa que puedes usar comandos como `git pull` o `git push` sin tener que especificar a qué rama te refieres: Git ya lo sabe.

Por ejemplo, si creas una rama local basada en `origin/main` con:

```bash
git checkout --track origin/main
```

Git configura automáticamente el seguimiento. Ahora, cuando haces `git push`, Git sabe que debe enviar los cambios a `origin/main`. Y cuando haces `git pull`, sabe de dónde traer las actualizaciones.

Puedes verificar qué ramas están siguiendo a cuáles con:

```bash
git branch -vv
```

Comprender estos vínculos permite trabajar de forma más fluida y evitar errores comunes como “¿por qué no se suben mis cambios?” o “¿por qué me dice que no encuentra la rama?”. En entornos educativos, donde los estudiantes clonan repositorios, hacen prácticas y suben correcciones, tener bien configuradas las *tracking branches* evita confusiones y facilita la evaluación.

<figure markdown="span" align="center">
  ![Image title](./imgs/git/DiagramShowingGit%20.png){ width="85%"  }
  <figcaption>Conceptos básico repositorios remotos.</figcaption>
</figure>

Flechas verdes punteadas que indican las **tracking branches**, es decir, el vínculo entre la rama local y su correspondiente rama remota.


## Conectando un repositorio remoto

Para vincular tu repositorio local con uno remoto (por ejemplo, en GitHub), usa el siguiente comando:

```bash
git remote add origin <url-del-repositorio>
```

Esto crea una referencia llamada `origin` que apunta a la URL del repositorio remoto. Es una convención común usar `origin` como nombre por defecto, aunque puedes usar cualquier otro identificador si lo necesitas.

Para verificar que el remoto se ha configurado correctamente:

```bash
git remote -v
```
Este comando muestra las URLs asociadas al remoto, tanto para `fetch` (descargar) como para `push` (subir).

Para comprobar tanto ramas locales como remotas

```bash
git branch -a
```



### Obtener cambios del remoto

Una vez conectado el remoto, puedes sincronizar tu repositorio local con los cambios que haya en el servidor:

- **`git fetch origin`**  
  Descarga los cambios del remoto pero **no los fusiona** con tu rama actual. Es útil para revisar antes de aplicar.

- **`git pull`**  
  Realiza un `fetch` seguido de un `merge` (o `rebase`, si está configurado). Es el comando más común para actualizar tu rama local con los últimos cambios del remoto.


### Enviar cambios al remoto

Para subir tus commits locales al repositorio remoto:

```bash
git push origin nombre_rama
```

Esto envía los cambios de la rama especificada (`nombre_rama`) al remoto `origin`.

Si es la primera vez que subes esa rama y no existe en el remoto, debes establecer una relación de rastreo:

```bash
git push -u origin nombre_rama
```

El parámetro `-u` configura la rama local para que rastree automáticamente la rama remota, lo que permite usar simplemente `git push` o `git pull` en el futuro sin especificar el nombre del remoto ni de la rama.




### Crear una rama local desde una remota

```bash
git fetch origin
git switch -c nueva_rama origin/rama_remota
```


### Eliminar una rama remota

```bash
git push origin --delete nombre_rama
git remote prune origin
```

---

### Ejemplo paso a paso

1. Crea una rama: `git switch -c feature-nueva`
2. Haz commits.
3. Conecta el remoto: `git remote add origin https://github.com/usuario/mi-repo.git`
4. Sube: `git push -u origin feature-nueva`
5. Crea un *pull request* en GitHub.
6. Si alguien actualiza `main`: `git pull`


## Uso de **GitHub** con Git

**GitHub** es una plataforma en línea que permite alojar repositorios Git y colaborar en proyectos de software. Funciona como un “repositorio remoto” donde puedes subir tu código, compartirlo con otros, gestionar versiones, reportar errores y trabajar en equipo. GitHub se basa en Git, el sistema de control de versiones distribuido que permite llevar un historial completo de los cambios realizados en un proyecto.

### Requisitos previos

Antes de comenzar a trabajar con GitHub desde tu equipo local, asegúrate de tener lo siguiente:

- Git instalado y funcionando correctamente (`git --version`)
- Una cuenta activa en [GitHub](https://github.com)
- Un repositorio local creado con `git init` o clonado desde GitHub


A continuación crearemos un repositorio en [Github](https://github.com/) y veremos cómo podemos usarlo para almacenar el repositorio Git que tenemos en nuestra máquina local.

Para ello veremos 

- Cómo crear el repositorio en [https://github.com/](https://github.com).
- Cómo preparar la autentificación para conectar el repositorio remoto con el local
- Cómo clonar o conectar los repositorios local y remoto.

<figure markdown="span" align="center">
  ![Image title](./imgs/git/git-local-remote.png){ width="85%"  }
  <figcaption>Trabajando con GitHub.</figcaption>
</figure>




!!!tip "Tutoriales recomendados para aprender a usar GitHub"

    - [Documentación oficial de GitHub (en español)](https://docs.github.com/es/repositories/creating-and-managing-repositories/cloning-a-repository) — Precisa y actualizada.
    - [Video: Crear y clonar repositorio con GitHub Desktop](https://www.youtube.com/watch?v=x03HP-z6brc) — Ideal para principiantes.
    - [Chuck's Academy: Creación y clonación de repositorios](https://www.chucksacademy.com/es/topic/git-github/creacion-y-clonacion-de-repositorios) — Explica tanto Git local como GitHub remoto.
    - [FreeCodeCamp en español](https://www.freecodecamp.org/espanol/news/como-crear-y-sincronizar-repositorios-con-git-y-github/) — Incluye ejemplos y vídeos subtitulados.



### Crear el repositorio en GitHub

Para crear un nuevo, por supuesto debemos tener una cuenta en [https://github.com/](https://github.com) y seguir los siguientes pasos:


1. Accede a [github.com](https://github.com) e inicia sesión.
2. Haz clic en el botón **"New"** o ve a [https://github.com/new](https://github.com/new).
3. Rellena los campos:
   - **Repository name**: nombre del proyecto.
   - **Description** (opcional).
   - Elige **Public** o **Private**.
   - Marca “Add a README file” si quieres incluirlo.
   - Opcional: añade `.gitignore` y licencia.
4. Haz clic en **"Create repository"**.

### Autenticación en GitHub

Desde 2021, GitHub **ya no permite autenticación por contraseña** al usar Git desde la terminal. 

Debes autenticarte mediante uno de estos métodos:

#### 1. **Token de acceso personal (PAT)**
- Se genera desde tu cuenta de GitHub en [Settings → Developer settings → Personal access tokens](https://github.com/settings/tokens)
- Al hacer `git push` o `git clone`, se te pedirá usuario y contraseña: usa tu usuario de GitHub y el token como contraseña.

!!!tip "Cómo generar un token clásico en GitHub"

    1. Accede a la configuración de tokens
         - Ve a:  [https://github.com/settings/tokens](https://github.com/settings/tokens)
         - Haz clic en **"Generate new token (classic)"**
    2. Configura el token
         - **Nombre**: elige uno descriptivo (por ejemplo, “Token para Git en SGE”)
         - **Expiración**: puedes elegir 30 días, 90 días o sin expiración
         - **Permisos mínimos necesarios**:
             - `repo` → acceso completo a tus repositorios (lectura, escritura, clonación)
             - (Opcional) `read:org` si trabajas con repositorios en organizaciones
    3. Generar y guardar el token
        - Haz clic en **"Generate token"**. GitHub te mostrará el token **una sola vez**.  
        - **Cópialo y guárdalo** en un lugar seguro (gestor de contraseñas o archivo `.token`).
    4. Usar el token al clonar por HTTPS
        
        Cuando hagas:

        ```bash
        git clone https://github.com/usuario/repositorio.git
        ```

        Git te pedirá:

        ```
        Username for 'https://github.com': tu_usuario
        Password for 'https://tu_usuario@github.com': pega_el_token
        ```

        - **Usuario**: tu nombre de usuario en GitHub (no tu correo)
        - **Contraseña**: el token generado

        Puedes guardar el token para no tener que escribirlo cada vez:

        ```bash
        git config --global credential.helper store
        ```


#### 2. **Clave SSH**
- Genera una clave SSH con:
  ```bash
  ssh-keygen -t ed25519 -C "tuemail@example.com"
  ```
- Añade la clave pública (`~/.ssh/id_ed25519.pub`) en GitHub → *Settings → SSH and GPG keys*
- Usa la URL SSH del repositorio (por ejemplo: `git@github.com:usuario/repositorio.git`) en lugar de la URL HTTPS.

Ambos métodos son seguros y válidos. El uso de SSH es más cómodo si trabajas frecuentemente desde el mismo equipo.


### Clonar un repositorio de GitHub

Esto crea una copia local completa del repositorio, incluyendo su historial de versiones.

Los pasos para clonar un repositorio GitHub en local usando la línea de comandos es la siguiente.

1. Copia la URL del repositorio (botón verde “Code” → HTTPS).
2. Abre la terminal y navega a la carpeta donde quieres clonar:
   ```bash
   cd ~/Documentos/proyectos
   ```
3. Ejecuta el comando:
   ```bash
   git clone https://github.com/usuario/nombre-del-repositorio.git
   ```
4. Accede a la carpeta clonada:
   ```bash
   cd nombre-del-repositorio
   ```

También es posible usar herramientas como [GitHub Desktop](https://desktop.github.com/download/), pero no vamos a usar este tipo de herramientas en nuestro curso.

### Conectar un repositorio local con GitHub

También podemos hacer el paso contrario, crear un repositorio local en el que ya tenemos un trabajo hecho y después  vincularlo con tu repositorio local usando:

```bash
git remote add origin https://github.com/usuario/repositorio.git
```

Esto crea una referencia llamada `origin` que apunta al repositorio remoto. Para confirmar que la conexión se ha establecido correctamente:

```bash
git remote -v
```

Este comando muestra las URLs configuradas para subir (`push`) y descargar (`fetch`) cambios.

## Operaciones básicas para gestión de repositorios remotos GitHub

<figure markdown="span" align="center">
  ![Image title](./imgs/git/git_remoto.webp){ width="85%"  }
  <figcaption>Trabajando con GitHub.</figcaption>
</figure>

### Subir cambios al repositorio remoto

Cuando hayas hecho commits en tu repositorio local, puedes subirlos a GitHub con:

```bash
git push origin nombre_rama
```

Si es la primera vez que subes esa rama, añade el parámetro `-u` para establecer una relación de seguimiento:

```bash
git push -u origin nombre_rama
```

Esto permite que en el futuro puedas usar simplemente `git push` sin especificar el remoto ni la rama.


### Obtener cambios del remoto

Para mantener tu repositorio local actualizado con los cambios que se hagan en GitHub:

- **`git fetch origin`**  
  Descarga los cambios del remoto, pero **no los aplica** automáticamente. Es útil para revisar antes de fusionar.

- **`git pull`**  
  Descarga y **fusiona** los cambios del remoto con tu rama actual. Es el comando más común para sincronizar tu trabajo.



## Resolución de conflictos con el repositorio remoto

Cuando trabajamos con GitHub, el conflicto no siempre aparece al hacer un `merge` entre ramas locales. Lo más habitual es que aparezca al **sincronizar** con el remoto: alguien (un compañero, o tú desde otro equipo o desde la web de GitHub) ha subido commits a la misma rama que tú has modificado en local.

!!!note "Repaso previo"

    La resolución del conflicto en sí (marcadores `<<<<<<<`, `=======`, `>>>>>>>`, `git add`, `git commit`) es **exactamente la misma** que vimos en el apartado [Resolución de conflictos en un merge](3_2_ramas.md#resolucion-de-conflictos-en-un-merge). Aquí nos centramos en lo que cambia: cómo se llega al conflicto y cómo se sincroniza después con GitHub.

### ¿Por qué se rechaza mi `git push`?

Git solo permite subir commits al remoto si la rama remota puede **avanzar en línea recta** (*fast-forward*) hasta tu último commit. Si en GitHub hay commits que tú no tienes en local, las historias han **divergido** y Git rechaza el envío para no sobrescribir el trabajo de otros.

```
      B ← origin/main (commit que ya está en GitHub)
     /
A ——┤
     \
      C ← main (tu commit local, aún sin subir)
```

La solución siempre es la misma: **primero traer los cambios del remoto, integrarlos con los tuyos (resolviendo conflictos si los hay) y después subir**.

### Escenario de ejemplo

Partimos del repositorio `mi-proyecto` de la actividad práctica, sincronizado con GitHub, con el fichero `README.md` que contiene `# Mi Proyecto`.

**1. Simulamos el cambio "de otra persona" en GitHub.** En la web del repositorio abre `README.md`, pulsa el icono del lápiz (*Edit this file*), cambia la primera línea a `# Mi Proyecto - versión GitHub` y haz *Commit changes* sobre `main`.

**2. Hacemos un cambio distinto sobre la misma línea, en local**, sin haber traído antes lo de GitHub:

```bash
echo "# Mi Proyecto - versión local" > README.md
git add README.md
git commit -m "README: título versión local"
```

**3. Intentamos subirlo:**

```bash
git push origin main
```

Git lo rechaza:

```
To https://github.com/usuario/mi-proyecto.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://github.com/usuario/mi-proyecto.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. ...
```

!!!warning "Nunca resuelvas esto con `git push --force`"

    `git push --force` sobrescribe la rama remota con tu versión y **borra los commits de los demás**. En una rama compartida no es una solución, es el problema. Si alguna vez es imprescindible (por ejemplo, tras un `rebase` en una rama personal), usa `git push --force-with-lease`, que al menos se niega a sobrescribir si el remoto ha cambiado desde tu último `fetch`.

### Pasos para resolver un conflicto con el remoto (merge)

#### 1. Traer los cambios del remoto sin integrarlos

```bash
git fetch origin
```

Esto actualiza `origin/main` en tu repositorio, pero **no toca** tu rama ni tus ficheros.

#### 2. Comparar qué ha pasado

```bash
git log --oneline --graph --all
```

```
* 3b4c5d6 (origin/main) README: título versión GitHub
| * 7e8f9a0 (HEAD -> main) README: título versión local
|/
* 1a2b3c4 Primer commit
```

Para ver qué cambios concretos trae el remoto:

```bash
git diff main origin/main
```

El `fetch` previo es lo que te permite **ver el conflicto venir** antes de provocarlo. Con `git pull` haces los pasos 1 y 3 de golpe, sin oportunidad de revisar.

#### 3. Integrar los cambios del remoto en tu rama

```bash
git merge origin/main
```

```
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

!!!note "`git merge origin/main` frente a `git pull`"

    `git pull` equivale a `git fetch` + `git merge origin/main` (o `git rebase`, según la configuración). Si en lugar de `git merge origin/main` usas `git pull`, el resultado es el mismo, pero en los marcadores de conflicto puede aparecer el identificador del commit en lugar de `origin/main`. Además, en versiones recientes de Git, si las historias han divergido y no hay configurado cómo reconciliarlas, `git pull` te pedirá que lo indiques (`git pull --no-rebase` para *merge*, `git pull --rebase` para *rebase*).

#### 4. Interpretar los marcadores

```
<<<<<<< HEAD
# Mi Proyecto - versión local
=======
# Mi Proyecto - versión GitHub
>>>>>>> origin/main
```

| Parte | Significado |
|---|---|
| Entre `<<<<<<< HEAD` y `=======` | Tu versión **local** (rama actual). |
| Entre `=======` y `>>>>>>> origin/main` | La versión que viene del **remoto** (GitHub). |

#### 5. Resolver, marcar y confirmar

Edita el fichero, deja el contenido final y **elimina los marcadores**:

```
# Mi Proyecto - versión local y GitHub
```

Después:

```bash
git add README.md
git commit -m "Merge de origin/main: resuelto conflicto en README.md"
```

#### 6. Subir el resultado a GitHub

```bash
git push origin main
```

Ahora el push sí se acepta, porque tu rama contiene todo lo que había en el remoto más tu commit de merge.

```
      B ← origin/main (antes del push)
     /   \
A ——┤     M ← main (HEAD) → tras el push, origin/main también apunta a M
     \   /
      C
```

#### 7. Verificar

```bash
git status
```

```
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

Comprueba también en GitHub (pestaña *Commits*) que aparecen tu commit, el del compañero y el commit de merge.

### Alternativa: integrar con `rebase`

En lugar de crear un commit de merge, podemos **recolocar** nuestros commits locales encima de lo que hay en el remoto:

```bash
git pull --rebase origin main
```

Si hay conflicto, Git se detiene en el commit conflictivo:

```
CONFLICT (content): Merge conflict in README.md
error: could not apply 7e8f9a0... README: título versión local
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
```

Los pasos son: editar el fichero y quitar marcadores → `git add README.md` → `git rebase --continue` (**sin** `git commit`) → `git push origin main`.

!!!warning "En un `rebase` los lados se invierten"

    Durante un `rebase`, Git va "reaplicando" tus commits sobre el remoto. Por eso, en los marcadores, `HEAD` (parte de arriba) es la versión **del remoto** y la parte de abajo es **tu commit local**. Es el efecto contrario al de un `merge`.

Resultado: historia lineal, sin commit de merge, pero con tu commit **reescrito** (nuevo identificador):

```
A — B — C' ← main, origin/main
```

| | `merge` | `rebase` |
|---|---|---|
| Historia resultante | Con commit de merge (refleja que hubo trabajo paralelo) | Lineal |
| Commits locales | Se conservan tal cual | Se reescriben (nuevo hash) |
| Cancelar el proceso | `git merge --abort` | `git rebase --abort` |
| Terminar tras resolver | `git add` + `git commit` | `git add` + `git rebase --continue` |
| Riesgo | Bajo | **No usar sobre commits que ya has subido y otros han descargado** |

!!!tip "Recomendación para empezar"

    Mientras estés aprendiendo, usa `fetch` + `merge`: es más fácil de entender y de deshacer (`git merge --abort`). Pasa a `rebase` cuando domines el flujo y solo con commits locales que aún no hayas subido.

### Conflictos en un *Pull Request*

Si trabajas con ramas por funcionalidad (como recomiendan las buenas prácticas) y abres un *Pull Request* hacia `main`, GitHub puede mostrar el aviso **"This branch has conflicts that must be resolved"**. Significa que `main` ha cambiado desde que creaste tu rama y hay solapamiento.

La forma recomendada de resolverlo es **en local**, en tu rama de funcionalidad:

```bash
git switch feature-nueva
git fetch origin
git merge origin/main        # traemos main a nuestra rama
# resolver conflictos: editar, git add, git commit
git push                     # el Pull Request se actualiza solo
```

El *Pull Request* se actualiza automáticamente con el nuevo commit y el aviso desaparece. GitHub también ofrece un botón **Resolve conflicts** con un editor web, útil para conflictos sencillos, pero en local tienes tu editor, puedes ejecutar el código y comprobar que sigue funcionando antes de subir.

### Resumen del proceso

1. `git push` rechazado (*fetch first* / *non-fast-forward*) → el remoto tiene commits que no tienes.
2. `git fetch origin` → traer sin integrar.
3. `git log --oneline --graph --all` → ver la divergencia.
4. `git merge origin/main` → integrar (o `git pull --rebase`).
5. Si hay conflicto: editar, quitar marcadores, `git add`, y `git commit` (merge) o `git rebase --continue` (rebase).
6. `git push` → subir el resultado.
7. Si algo sale mal: `git merge --abort` o `git rebase --abort`.

!!!note "Cómo reducir los conflictos con el remoto"

    - Haz `git pull` (o `fetch`) **antes de empezar a trabajar** y antes de cada `push`.
    - Sube cambios pequeños y con frecuencia.
    - Trabaja en ramas distintas y evita que dos personas editen a la vez los mismos ficheros.
    - Nunca uses `git push --force` en ramas compartidas.


## El archivo `.gitignore` en sistemas de control de versiones con Git

El archivo `.gitignore` constituye una herramienta esencial en la gestión de proyectos versionados con Git. Su función principal es indicar al sistema qué archivos o directorios deben ser excluidos del seguimiento, evitando que se incluyan en los commits y, por tanto, en el historial del repositorio. Esta exclusión resulta especialmente útil para preservar la limpieza del repositorio, reducir el ruido técnico y evitar la incorporación de archivos irrelevantes, temporales o específicos del entorno local de desarrollo.

En contextos educativos o institucionales, donde los proyectos pueden ser compartidos entre múltiples usuarios y entornos, el uso adecuado de `.gitignore` contribuye a mantener la coherencia del repositorio y a prevenir conflictos derivados de configuraciones locales o archivos generados automáticamente.


### Funcionamiento y sintaxis

El archivo `.gitignore` se ubica habitualmente en la raíz del repositorio, aunque Git permite definir archivos `.gitignore` adicionales en subdirectorios para aplicar reglas específicas a determinadas rutas. Su contenido consiste en una lista de patrones que describen los nombres de archivos o carpetas que deben ser ignorados por Git.

Ejemplos de patrones comunes incluyen:

- Archivos temporales o de registro: `*.log`, `*.tmp`
- Directorios de configuración de editores: `.vscode/`, `.idea/`
- Archivos compilados o generados: `*.class`, `*.pyc`, `dist/`, `build/`

Los patrones pueden incluir comodines (`*`, `?`) y rutas relativas. Los comentarios se indican con el símbolo `#`.

!!!example "Ejemplo de fichero `.gitignore`"

    Fichero `.gitignore` en la raíz del proyecto git

    ```
    # Ignorar archivos temporales
    *.log
    *.tmp

    # Ignorar carpetas de configuración local
    .vscode/
    .idea/

    # Ignorar archivos compilados
    *.class
    *.pyc
    ```

    Git lee este archivo cada vez que evalúa qué cambios hay en tu proyecto. Si un archivo coincide con algún patrón de .gitignore, simplemente lo ignora.

### Consideraciones prácticas

El uso de `.gitignore` debe guiarse por criterios técnicos que favorezcan la reproducibilidad del proyecto y la claridad del repositorio. Se recomienda:

- Excluir archivos que no aportan valor al código fuente ni a la documentación compartida.
- Evitar la exclusión de archivos necesarios para la ejecución o comprensión del proyecto por parte de otros usuarios.
- Utilizar plantillas específicas por lenguaje o entorno, disponibles en repositorios como [github.com/github/gitignore](https://github.com/github/gitignore), para facilitar la configuración inicial.
- Revisar cuidadosamente los patrones amplios que podrían excluir archivos relevantes por error.


### Eliminación de archivos ya versionados

En caso de que un archivo haya sido añadido al repositorio antes de ser incluido en `.gitignore`, Git continuará rastreándolo. Para que deje de hacerlo, es necesario eliminarlo del índice sin borrarlo del sistema de archivos:

```bash
git rm --cached nombre-del-archivo.ext
```

Una vez ejecutado este comando, Git respetará la regla definida en `.gitignore` y omitirá dicho archivo en futuros commits.

  El uso correcto de `.gitignore` no solo mejora la calidad técnica del repositorio, sino que también facilita la colaboración entre estudiantes y docentes, al evitar la propagación de configuraciones locales, archivos innecesarios o elementos que dificulten la evaluación y el mantenimiento del proyecto. 


## Actividad práctica

Version 1: 

1. Crear un nuevo repositorio en GitHub (sin README ni .gitignore)  
2. En tu equipo local
   ```bash
   mkdir mi-proyecto
   cd mi-proyecto
   git clone  https://github.com/usuario/mi-proyecto.git
   ```


Versión 2: 

1. Crear un nuevo repositorio en GitHub (sin README ni .gitignore)
2. En tu equipo local:
   ```bash
   mkdir mi-proyecto
   cd mi-proyecto
   git init
   ```
3. Crear un archivo `README.md` y hacer el primer commit:
   ```bash
   echo "# Mi Proyecto" > README.md
   git add README.md
   git commit -m "Primer commit"
   ```
4. Conectar con GitHub:
   ```bash
   git remote add origin https://github.com/usuario/mi-proyecto.git
   git pull --rebase origin main    # esto solo si hay algo en el repositorio
   git push -u origin main
   ```
5. Verificar que el archivo aparece en GitHub



!!!tip "Buenas prácticas con remotos"

    - Haz `git fetch` o `git pull` antes de trabajar.
    - No hagas *push* directo a `main`.
    - Usa ramas por funcionalidad.
    - Limpia ramas remotas obsoletas.

