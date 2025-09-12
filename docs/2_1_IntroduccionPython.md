---
title: Introducción a Python
description: Introducción del Python
---

Python es un lenguaje de programación de alto nivel, interpretado y de propósito general. Es conocido por su sintaxis clara y legible, lo que lo convierte en una excelente opción para principiantes y profesionales.

## Instalación de Python

### Windows
1. Visita https://www.python.org/downloads/
2. Descarga el instalador para Windows.
3. Ejecuta el instalador y asegúrate de marcar la opción "Add Python to PATH".

### macOS
Puedes instalar Python usando Homebrew:
```bash
brew install python
```

### Linux
En la mayoría de distribuciones, puedes instalar Python con:
```bash
sudo apt update
sudo apt install python3
```

## Verificación de la instalación
Para verificar que Python está correctamente instalado, abre una terminal y ejecuta:
```bash
python --version
```
o
```bash
python3 --version
```

## Primeros pasos en Python

Puedes iniciar el intérprete interactivo escribiendo `python` o `python3` en la terminal. También puedes guardar tus programas en archivos `.py` y ejecutarlos.

## Tipos de datos básicos

### Números
```python
entero = 10
flotante = 3.14
complejo = 2 + 3j
```

### Cadenas de texto
```python
cadena = "Hola, mundo"
texto_multilinea = '''Esto es
una cadena
multilínea'''
```

### Booleanos
```python
verdadero = True
falso = False
```

### Listas
```python
lista = [1, 2, 3, "cuatro", True]
```

### Tuplas
```python
tupla = (1, 2, 3)
```

### Conjuntos
```python
conjunto = {1, 2, 3, 3}
```

### Diccionarios
```python
diccionario = {"nombre": "Ana", "edad": 30}
```

## Estructuras de control

### Condicionales
```python
if edad >= 18:
    print("Eres mayor de edad")
elif edad > 13:
    print("Eres adolescente")
else:
    print("Eres niño")
```

### Bucles
```python
for i in range(5):
    print(i)

while condicion:
    hacer_algo()
```

## Funciones
```python
def saludar(nombre):
    print(f"Hola, {nombre}!")
```

