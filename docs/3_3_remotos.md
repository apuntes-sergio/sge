---
title: Repositorios remotos
description: Guía básica de Git y GitHub. Repositorios remotos, sincronización y colaboración
---

Hasta ahora hemos trabajado localmente. Pero muchas veces querrás compartir tu trabajo o colaborar con otros mediante repositorios remotos (GitHub, GitLab, Bitbucket…).

## Conceptos: remoto, tracking, branch remoto

- **Remoto (remote)**: copia del repositorio en un servidor. Generalmente se llama `origin`.
- **Branch remoto**: referencia local que apunta al estado conocido de una rama en el remoto, por ejemplo `origin/main`.
- **Tracking branch**: rama local enlazada con una rama remota, lo que permite usar `git pull` y `git push` sin parámetros adicionales.

---

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



## Conectando un remoto

```bash
git remote add origin <url-del-repositorio>
```

Asocia el nombre `origin` con la URL del repositorio remoto.

Comprobar:

```bash
git remote -v
```


## Ver ramas remotas conocidas

- `git branch -r` → ramas remotas.
- `git branch -a` → locales + remotas.

---

## Crear una rama local desde una remota

```bash
git fetch origin
git switch -c nueva_rama origin/rama_remota
```

---

## Manejando diferencias entre local y remoto

- **Adelantada**: tienes commits que el remoto no tiene → `git push`.
- **Atrás**: hay cambios remotos que aún no tienes → `git pull`.
- **Divergente**: ambos tienen commits distintos → posible conflicto.

---

## Eliminar una rama remota

```bash
git push origin --delete nombre_rama
git remote prune origin
```

---

## Ejemplo paso a paso

1. Crea una rama: `git switch -c feature-nueva`
2. Haz commits.
3. Conecta el remoto: `git remote add origin https://github.com/usuario/mi-repo.git`
4. Sube: `git push -u origin feature-nueva`
5. Crea un *pull request* en GitHub.
6. Si alguien actualiza `main`: `git pull`

Perfecto, Sergio. Aquí tienes una documentación clara, modular y lista para incluir en tus apuntes o como guía institucional para estudiantes y docentes. Está pensada para un uso básico y efectivo de GitHub con Git desde línea de comandos.



## Uso de GitHub con Git

**GitHub** es una plataforma en línea que permite alojar repositorios Git y colaborar en proyectos de software. Funciona como un “repositorio remoto” donde puedes subir tu código, compartirlo con otros, gestionar versiones, reportar errores y trabajar en equipo. GitHub se basa en Git, el sistema de control de versiones distribuido que permite llevar un historial completo de los cambios realizados en un proyecto.

### Requisitos previos

Antes de comenzar a trabajar con GitHub desde tu equipo local, asegúrate de tener lo siguiente:

- Git instalado y funcionando correctamente (`git --version`)
- Una cuenta activa en [GitHub](https://github.com)
- Un repositorio local creado con `git init` o clonado desde GitHub

### Autenticación con GitHub

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

Si quieres empezar a trabajar con un repositorio ya existente en GitHub, puedes clonarlo con:

```bash
git clone https://github.com/usuario/repositorio.git
```

Esto crea una copia local completa del repositorio, incluyendo su historial de versiones.
### Conectar un repositorio local con GitHub

Una vez creado tu repositorio en GitHub, puedes vincularlo con tu repositorio local usando:

```bash
git remote add origin https://github.com/usuario/repositorio.git
```

Esto crea una referencia llamada `origin` que apunta al repositorio remoto. Para confirmar que la conexión se ha establecido correctamente:

```bash
git remote -v
```

Este comando muestra las URLs configuradas para subir (`push`) y descargar (`fetch`) cambios.

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


### Actividad práctica

Version 1: 

1. Crear un nuevo repositorio en GitHub (sin README ni .gitignore)  
2. En tu equipo local
   ```bash
   git remote add origin https://github.com/usuario/mi-proyecto.git
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
   git push -u origin main
   ```
5. Verificar que el archivo aparece en GitHub



!!!tip "Buenas prácticas con remotos"

    - Haz `git fetch` o `git pull` antes de trabajar.
    - No hagas *push* directo a `main`.
    - Usa ramas por funcionalidad.
    - Limpia ramas remotas obsoletas.

