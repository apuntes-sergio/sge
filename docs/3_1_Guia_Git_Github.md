---
title: Guía básica de Git y GitHub
description: Guía básica de Git y GitHub
---

# Guía Completa de Git y GitHub para DAM

## Introducción a Git
Git es un sistema de control de versiones distribuido que permite a los desarrolladores gestionar el historial de cambios en sus proyectos. Es esencial para el trabajo colaborativo y el mantenimiento de código.

### ¿Por qué usar Git?
- Control de versiones.
- Trabajo colaborativo.
- Integridad de datos.
- Flujo de trabajo flexible.

## Instalación de Git
Para instalar Git:
- En Windows: [https://git-scm.com/download/win](https://git-scm.com/download/win)
- En macOS: `brew install git`
- En Linux: `sudo apt install git`

## Configuración inicial
```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tuemail@example.com"
git config --global core.editor "code --wait"
```

## Comandos esenciales de Git
### Inicializar repositorio
```bash
git init
```
Crea un nuevo repositorio local.

### Clonar repositorio
```bash
git clone URL
```
Descarga un repositorio remoto.

### Estado del repositorio
```bash
git status
```
Muestra los archivos modificados y no rastreados.

### Añadir archivos
```bash
git add archivo.txt
```
Agrega archivos al área de preparación.

### Confirmar cambios
```bash
git commit -m "Descripción del cambio"
```
Guarda los cambios en el historial.

### Ver historial
```bash
git log
```
Muestra los commits realizados.

### Crear y cambiar ramas
```bash
git branch nueva-rama
git checkout nueva-rama
```

### Fusionar ramas
```bash
git merge rama-a-fusionar
```

### Eliminar ramas
```bash
git branch -d nombre-rama
```

### Ver diferencias
```bash
git diff
```
Muestra los cambios no confirmados.

### Revertir cambios
```bash
git checkout -- archivo.txt
```
Revierte cambios en un archivo.

### Revertir commit
```bash
git revert ID_commit
```
Crea un nuevo commit que revierte el anterior.

### Resetear a un commit anterior
```bash
git reset --hard ID_commit
```
Vuelve el repositorio al estado de un commit anterior.

## Trabajo con repositorios remotos
### Añadir remoto
```bash
git remote add origin URL
```

### Subir cambios
```bash
git push origin rama
```

### Descargar cambios
```bash
git pull origin rama
```

## GitHub y su integración con Git
GitHub es una plataforma que permite alojar repositorios Git y colaborar en proyectos.

### Crear repositorio en GitHub
1. Inicia sesión.
2. Haz clic en "New".
3. Asigna nombre y visibilidad.
4. Crea el repositorio.

### Conectar repositorio local
```bash
git remote add origin https://github.com/usuario/repositorio.git
git push -u origin master
```

### Colaboración
- **Fork**: Copia de un repositorio.
- **Pull Request**: Solicitud para fusionar cambios.
- **Issues**: Reporte de errores o sugerencias.

## Buenas prácticas
- Commits frecuentes y descriptivos.
- Uso de ramas para nuevas funcionalidades.
- Revisar código antes de fusionar.
- Documentar el proyecto.

## Recursos adicionales
- [Documentación oficial de Git](https://git-scm.com/doc)
- [GitHub Docs](https://docs.github.com)

