
---

## 🗓️ Planificación: Fundamentos de Python para desarrollo modular (7 sesiones)

### 🔹 **Sesión 1: Introducción a Python y entorno de trabajo**
**Objetivos:**
- Familiarizarse con la sintaxis básica de Python.
- Configurar el entorno de desarrollo (VS Code + WSL o IDLE).
- Ejecutar scripts y trabajar con el intérprete interactivo.

**Contenidos:**
- Tipado dinámico vs. tipado estático (comparación con Java).
- Tipos de datos básicos: `int`, `float`, `str`, `bool`.
- Operadores, expresiones y comentarios.
- Convenciones de estilo (PEP8).

**Actividad práctica:**
- Crear un script con operaciones básicas y mostrar resultados por consola.

---

### 🔹 **Sesión 2: Control de flujo y estructuras condicionales**
**Objetivos:**
- Dominar condicionales y bucles.
- Comprender la indentación como estructura sintáctica.

**Contenidos:**
- `if`, `elif`, `else`.
- Bucles `for`, `while`, `range()`.
- Control de flujo: `break`, `continue`, `pass`.

**Actividad práctica:**
- Crear un menú interactivo por consola.
- Simular un sistema de selección de opciones con validación.

---

### 🔹 **Sesión 3: Estructuras de datos y colecciones**
**Objetivos:**
- Manipular listas, tuplas, diccionarios y conjuntos.
- Aplicar estructuras de datos en contextos similares a modelos de datos.

**Contenidos:**
- Listas y diccionarios: acceso, modificación, métodos comunes.
- Tuplas y sets: usos y diferencias.
- Iteración sobre colecciones.
- Comprensiones de listas (`list comprehensions`).

**Actividad práctica:**
- Crear una estructura que simule registros de usuarios con diccionarios.
- Filtrar y ordenar datos.

---

### 🔹 **Sesión 4: Funciones y modularidad**
**Objetivos:**
- Definir funciones reutilizables.
- Comprender el paso de argumentos y valores de retorno.

**Contenidos:**
- `def`, parámetros, retorno.
- Argumentos por defecto y nombrados.
- Ámbito de variables (`global`, `local`).
- Documentación con docstrings.

**Actividad práctica:**
- Crear funciones para validar datos, calcular totales, y formatear salidas.
- Modularizar el menú de la sesión 2.

---

### 🔹 **Sesión 5: Gestión de paquetes con pip y entornos virtuales**
**Objetivos:**
- Aprender a instalar y gestionar librerías externas.
- Crear entornos virtuales para aislar proyectos.

**Contenidos:**
- Uso de `pip`: instalación, listado, desinstalación.
- Creación de entornos virtuales con `venv`.
- Activación y desactivación del entorno.
- Uso de `requirements.txt`.

**Actividad práctica:**
- Crear un entorno virtual llamado `entorno_dam`.
- Instalar `requests` y `pandas`.
- Exportar e importar dependencias.

---

### 🔹 **Sesión 6: Programación orientada a objetos (POO)**
**Objetivos:**
- Comprender la estructura de clases y objetos en Python.
- Aplicar encapsulamiento y reutilización de código.

**Contenidos:**
- `class`, `__init__`, `self`.
- Atributos y métodos.
- Encapsulamiento básico.
- Comparación con clases Java (constructores, visibilidad).

**Actividad práctica:**
- Crear una clase `Producto` con atributos y métodos.
- Instanciar objetos y simular operaciones como `actualizar_precio()`.

---

### 🔹 **Sesión 7: Proyecto final y repaso**
**Objetivos:**
- Aplicar todos los conceptos en un proyecto modular.
- Consolidar conocimientos previos.

**Contenidos:**
- Revisión de funciones, estructuras de datos y clases.
- Organización en módulos (`modelo.py`, `vista.py`, `controlador.py`).
- Buenas prácticas de estilo y documentación.

**Actividad práctica:**
- Proyecto: sistema de gestión de productos con menú, clases, funciones y exportación de datos.
- Evaluación por rúbrica: funcionalidad, modularidad, estilo, documentación.

---

## 📘 Recursos complementarios
- Comparativas Java ↔ Python para cada sesión.
- Cuaderno de ejercicios por sesión.
- Plantilla de proyecto final.
- Rúbrica de evaluación.


---
---
---

# Unidad 1: Introducción a Python y entorno de trabajo

## 🎯 Objetivos

- Familiarizarse con la sintaxis básica de Python.
- Comprender las diferencias clave entre Python y Java.
- Configurar el entorno de desarrollo para trabajar con Python.
- Ejecutar scripts y trabajar con el intérprete interactivo.
- Manipular cadenas de texto y realizar entradas/salidas por consola.
- Conocer los tipos numéricos y operadores básicos.
- Adoptar buenas prácticas de estilo y documentación.


## ¿Qué es Python?

Python es un lenguaje de programación interpretado, de alto nivel y con tipado dinámico. Su sintaxis clara y concisa lo convierte en una excelente opción para desarrolladores que buscan rapidez en el desarrollo y legibilidad del código.

### Características principales

- **Interpretado**: No requiere compilación previa.
- **Tipado dinámico**: No es necesario declarar el tipo de variable.
- **Multiparadigma**: Soporta programación imperativa, orientada a objetos y funcional.
- **Extensible**: Gran ecosistema de librerías y módulos.
- **Popularidad**: Usado en desarrollo web, ciencia de datos, automatización, IA, etc.

## Comparativa Python vs. Java

| Característica         | Java                          | Python                        |
|------------------------|-------------------------------|-------------------------------|
| Tipado                 | Estático                      | Dinámico                      |
| Sintaxis               | Verbosa                       | Concisa                       |
| Compilación            | Compilado a bytecode          | Interpretado                  |
| Declaración de variables | Obligatoria                  | Implícita                     |
| Manejo de excepciones  | Obligatorio en muchos casos   | Opcional                      |
| Orientación a objetos  | Obligatoria                   | Opcional                      |

### Ejemplo comparativo

**Java**
```java
int x = 5;
System.out.println(x);
```

**Python**
```python
x = 5
print(x)
```

---

## Instalación y entorno de trabajo

### Opción 1: Python nativo

- Descargar desde [python.org](https://www.python.org)
- Instala IDLE (editor básico incluido)
- Verifica instalación:
  ```bash
  python --version
  ```

### Opción 2: WSL + VS Code (recomendado)

- Instala WSL con Ubuntu desde Microsoft Store.
- Instala Python dentro de WSL:
  ```bash
  sudo apt update
  sudo apt install python3 python3-pip
  ```
- Instala VS Code y la extensión "Remote - WSL".

---

## 🔢 Tipos numéricos y operadores

### 📌 Tipos numéricos

```python
entero = 10          # int
decimal = 3.14       # float
complejo = 2 + 3j    # complex
```

### ➕ Operadores aritméticos

| Operador | Descripción         | Ejemplo       | Resultado |
|----------|---------------------|---------------|-----------|
| `+`      | Suma                | `5 + 3`       | `8`       |
| `-`      | Resta               | `5 - 2`       | `3`       |
| `*`      | Multiplicación      | `4 * 2`       | `8`       |
| `/`      | División flotante   | `5 / 2`       | `2.5`     |
| `//`     | División entera     | `5 // 2`      | `2`       |
| `%`      | Módulo (resto)      | `5 % 2`       | `1`       |
| `**`     | Potencia            | `2 ** 3`      | `8`       |

---

## 🧵 Cadenas de texto (`str`)

### ✍️ Definición y operaciones

```python
nombre = "Sergio"
saludo = 'Hola'
```

```python
texto = "Python DAM"
print(len(texto))            # Longitud
print(texto.upper())         # Mayúsculas
print(texto.lower())         # Minúsculas
print(texto.replace("DAM", "2025"))  # Reemplazo
print(texto[0:6])            # Slicing
```

### 🔗 Concatenación

```python
nombre = "Sergio"
mensaje = "Hola " + nombre
print( mensaje)
print( "Esta también es otra cadena concatenada con comas, ", nombre, "¿se entiende?")
```

### 🎯 f-strings (recomendado)

```python
edad = 35
print(f"Hola {nombre}, tienes {edad} años.")
```

---

## 📤 Entrada y salida por consola

### 📥 `input()`

```python
nombre = input("Introduce tu nombre: ")
print(f"Bienvenido, {nombre}")
```

### 🔄 Conversión de tipos

```python
edad = int(input("Introduce tu edad: "))
print(f"Tu edad es {edad}")
```

---

## 🧼 Buenas prácticas y estilo

### 📐 Convenciones PEP8

- Usa 4 espacios por nivel de indentación.
- Nombres de variables en minúsculas y con guiones bajos: `nombre_usuario`
- Comentarios claros y útiles.
- Usa docstrings para documentar funciones y clases.

### 💬 Comentarios

```python
# Esto es un comentario de línea

"""
Esto es un comentario
de varias líneas
"""
```

---

## Actividades

### 🧪 Actividad 1: Instalación y entorno

**Objetivo:** Configurar el entorno de trabajo en WSL o IDLE.

**Pasos:**
- Instalar Python y verificar versión.
- Crear un archivo `hola.py` que imprima un saludo.
- Ejecutar el archivo desde terminal.

**Entrega:** Captura de pantalla del terminal con la ejecución del script.

---

### 🧪 Actividad 2: Primer script con entrada y salida

**Objetivo:** Crear un script que solicite el nombre del usuario y su edad, y muestre un mensaje personalizado.

**Requisitos:**
- Uso de `input()` y conversión de tipos.
- Uso de `print()` y f-strings.
- Manipulación básica de cadenas.

**Ejemplo de salida:**
```
Introduce tu nombre: Sergio
Introduce tu edad: 35
Hola Sergio, tienes 35 años.
```

---

### 🧪 Actividad 3: Comparativa Java ↔ Python

**Objetivo:** Traducir tres fragmentos de código Java a Python.

**Fragmentos sugeridos:**
1. Declaración de variables
2. Condicional `if`
3. Entrada/salida por consola

**Entrega:** Documento con ambos códigos y una breve reflexión sobre las diferencias.



## ✅ Cierre de la unidad

Esta unidad sienta las bases para trabajar con Python de forma profesional. A partir de aquí, se abordarán estructuras de control, colecciones, funciones y clases, que serán esenciales para el desarrollo modular en entornos reales.


---

Incluye apuntes extensos, ejemplos claros y tres actividades prácticas al final.

---

# Unidad 2: Estructuras condicionales y control de flujo

## 🎯 Objetivos

- Comprender el uso de condicionales en Python.
- Aplicar estructuras de control para tomar decisiones en el código.
- Comparar la lógica condicional en Python con la de Java.
- Utilizar operadores lógicos y de comparación.
- Crear scripts interactivos que respondan a diferentes entradas del usuario.

---

## 🧠 ¿Qué es el control de flujo?

El control de flujo permite que un programa tome decisiones y ejecute diferentes bloques de código según ciertas condiciones. En Python, esto se logra principalmente con las estructuras `if`, `elif` y `else`.

---

## 🔍 Condicionales en Python

### ✅ Sintaxis básica

```python
edad = 20

if edad >= 18:
    print("Eres mayor de edad.")
```

### 🔄 `if` - `elif` - `else`

```python
nota = 7

if nota >= 9:
    print("Sobresaliente")
elif nota >= 7:
    print("Notable")
elif nota >= 5:
    print("Aprobado")
else:
    print("Suspenso")
```

### ⚠️ Importancia de la indentación

En Python, los bloques de código se definen por **indentación** (espacios), no por llaves `{}` como en Java. Se recomienda usar **4 espacios** por nivel.

---

## 🔗 Operadores de comparación

| Operador | Significado        | Ejemplo        | Resultado |
|----------|--------------------|----------------|-----------|
| `==`     | Igual a            | `x == 5`       | `True`    |
| `!=`     | Distinto de        | `x != 3`       | `True`    |
| `<`      | Menor que          | `x < 10`       | `True`    |
| `>`      | Mayor que          | `x > 2`        | `True`    |
| `<=`     | Menor o igual que  | `x <= 5`       | `True`    |
| `>=`     | Mayor o igual que  | `x >= 5`       | `True`    |

---

## 🔗 Operadores lógicos

| Operador | Descripción        | Ejemplo                  | Resultado |
|----------|--------------------|--------------------------|-----------|
| `and`    | Ambas condiciones  | `x > 5 and x < 10`       | `True`    |
| `or`     | Una u otra         | `x < 3 or x > 8`         | `True`    |
| `not`    | Negación           | `not x == 5`             | `False`   |

---

## 🧪 Ejemplo completo

```python
usuario = input("Introduce tu nombre: ")
edad = int(input("Introduce tu edad: "))

if edad >= 18:
    print(f"{usuario}, puedes acceder al sistema.")
else:
    print(f"{usuario}, acceso denegado por edad.")
```

---

## 🧼 Buenas prácticas

- Usa nombres de variables descriptivos.
- Evita condicionales anidados innecesarios.
- Comenta el propósito de cada bloque si no es evidente.
- Usa `elif` en lugar de múltiples `if` independientes.

---

## 📝 Actividades

### 🧪 Actividad 1: Clasificador de notas

**Objetivo:** Crear un script que solicite una nota numérica y muestre la calificación textual.

**Requisitos:**
- Uso de `input()` y conversión a `int`.
- Condicionales `if`, `elif`, `else`.

**Ejemplo de salida:**
```
Introduce tu nota: 8
Tu calificación es: Notable
```

---

### 🧪 Actividad 2: Control de acceso

**Objetivo:** Crear un script que solicite nombre y edad, y determine si el usuario puede acceder a un sistema.

**Requisitos:**
- Entrada de datos con `input()`.
- Condicional simple con `if` y `else`.
- Uso de f-strings.

**Ejemplo de salida:**
```
Introduce tu nombre: Sergio
Introduce tu edad: 17
Sergio, acceso denegado por edad.
```

---

### 🧪 Actividad 3: Comparador de números

**Objetivo:** Solicitar dos números al usuario y mostrar cuál es mayor, menor o si son iguales.

**Requisitos:**
- Conversión de tipos.
- Condicionales múltiples.
- Uso de operadores de comparación.

**Ejemplo de salida:**
```
Introduce el primer número: 10
Introduce el segundo número: 7
El número mayor es: 10
```

---

## ✅ Cierre de la unidad

Las estructuras condicionales son esenciales para controlar el comportamiento de una aplicación. En el desarrollo de módulos, estas decisiones permiten validar datos, controlar accesos y definir reglas de negocio. En la próxima unidad, abordaremos las **estructuras de datos** que permiten almacenar y manipular información de forma eficiente.





---
---
---


# Unidad 2 bis: Estructuras condicionales, bucles y control de flujo

## Objetivos

- Dominar condicionales `if`, `elif`, `else` para tomar decisiones.
- Comprender y aplicar bucles `for` y `while`.
- Utilizar operadores lógicos y de comparación.
- Controlar el flujo de ejecución con `break`, `continue` y `pass`.
- Comparar las estructuras de control en Python y Java.
- Crear scripts interactivos que respondan a diferentes entradas del usuario.

---

## Condicionales en Python

Las estructuras condicionales permiten ejecutar bloques de código según se cumplan ciertas condiciones.

### Sintaxis básica

```python
edad = 20

if edad >= 18:
    print("Eres mayor de edad.")
```

### if - elif - else

```python
nota = 7

if nota >= 9:
    print("Sobresaliente")
elif nota >= 7:
    print("Notable")
elif nota >= 5:
    print("Aprobado")
else:
    print("Suspenso")
```

### Importancia de la indentación

En Python, los bloques se definen por **espacios** (indentación), no por llaves `{}` como en Java. Se recomienda usar **4 espacios** por nivel.

---

## Operadores de comparación

| Operador | Significado        | Ejemplo        | Resultado |
|----------|--------------------|----------------|-----------|
| `==`     | Igual a            | `x == 5`       | `True`    |
| `!=`     | Distinto de        | `x != 3`       | `True`    |
| `<`      | Menor que          | `x < 10`       | `True`    |
| `>`      | Mayor que          | `x > 2`        | `True`    |
| `<=`     | Menor o igual que  | `x <= 5`       | `True`    |
| `>=`     | Mayor o igual que  | `x >= 5`       | `True`    |

---

## Operadores lógicos

| Operador | Descripción        | Ejemplo                  | Resultado |
|----------|--------------------|--------------------------|-----------|
| `and`    | Ambas condiciones  | `x > 5 and x < 10`       | `True`    |
| `or`     | Una u otra         | `x < 3 or x > 8`         | `True`    |
| `not`    | Negación           | `not x == 5`             | `False`   |

---

## Bucles en Python

Los bucles permiten repetir bloques de código mientras se cumpla una condición o sobre una colección de elementos.

### Bucle while

```python
contador = 0

while contador < 5:
    print(f"Contador: {contador}")
    contador += 1
```

### Bucle for con range()

```python
for i in range(5):
    print(f"Iteración {i}")
```

### Iterar sobre listas

```python
frutas = ["manzana", "pera", "kiwi"]

for fruta in frutas:
    print(fruta)
```

---

## Control de flujo

### break

Interrumpe el bucle actual.

```python
for i in range(10):
    if i == 5:
        break
    print(i)
```

### continue

Salta a la siguiente iteración.

```python
for i in range(5):
    if i == 2:
        continue
    print(i)
```

### pass

No hace nada; se usa como marcador de posición.

```python
for i in range(3):
    pass  # Código pendiente
```

---

## Comparativa con Java

| Concepto        | Java                          | Python                        |
|-----------------|-------------------------------|-------------------------------|
| Condicional     | `if`, `else if`, `else`       | `if`, `elif`, `else`          |
| Bucle for       | `for (int i = 0; ...)`        | `for i in range()`            |
| Bucle while     | `while (condición)`           | `while condición:`            |
| Control flujo   | `break`, `continue`           | `break`, `continue`, `pass`   |

---

## Actividades

### Actividad 1: Clasificador de notas

**Objetivo:** Crear un script que solicite una nota numérica y muestre la calificación textual.

**Requisitos:**
- Uso de `input()` y conversión a `int`.
- Condicionales `if`, `elif`, `else`.

**Ejemplo de salida:**
```
Introduce tu nota: 8
Tu calificación es: Notable
```

---

### Actividad 2: Contador con while

**Objetivo:** Crear un contador que muestre los números del 1 al 10 usando un bucle `while`.

**Requisitos:**
- Uso de `while`.
- Incremento de variable.
- Uso de `print()`.

**Ejemplo de salida:**
```
1
2
3
...
10
```

---

### Actividad 3: Menú interactivo con for

**Objetivo:** Crear un menú que muestre una lista de opciones y permita al usuario seleccionar una.

**Requisitos:**
- Uso de listas.
- Bucle `for` para mostrar opciones.
- Condicional para ejecutar la opción seleccionada.

**Ejemplo de salida:**
```
Opciones disponibles:
1. Ver perfil
2. Editar perfil
3. Salir

Selecciona una opción: 2
Has elegido: Editar perfil
```

---

## Cierre de la unidad

Las estructuras condicionales y los bucles son esenciales para controlar el comportamiento de una aplicación. En el desarrollo de módulos, estas decisiones permiten validar datos, controlar accesos y definir reglas de negocio. En la próxima unidad, abordaremos las **estructuras de datos** que permiten almacenar y manipular información de forma eficiente.


---
---
---





# Unidad 3: Estructuras de datos básicas

## Objetivos

- Comprender el uso de listas, diccionarios y tuplas en Python.
- Manipular colecciones de datos de forma eficiente.
- Aplicar métodos comunes para acceder, modificar y recorrer estructuras.
- Comparar las estructuras de datos de Python con las de Java.
- Preparar la base para representar modelos de datos en aplicaciones reales.

---

## Listas

Las listas son colecciones ordenadas y mutables. Se definen con corchetes `[]`.

### Definición

```python
frutas = ["manzana", "plátano", "naranja"]
```

### Acceso por índice

```python
print(frutas[0])  # manzana
```

### Modificación

```python
frutas[1] = "kiwi"
```

### Métodos comunes

```python
frutas.append("pera")       # Añadir al final
frutas.insert(1, "uva")     # Insertar en posición
frutas.remove("naranja")    # Eliminar por valor
frutas.pop()                # Eliminar último
frutas.sort()               # Ordenar
```

---

## Diccionarios

Los diccionarios son colecciones no ordenadas de pares clave-valor. Se definen con llaves `{}`.

### Definición

```python
persona = {
    "nombre": "Sergio",
    "edad": 35,
    "ciudad": "Valencia"
}
```

### Acceso y modificación

```python
print(persona["nombre"])        # Sergio
persona["edad"] = 36            # Modificar valor
persona["email"] = "sergio@example.com"  # Añadir nueva clave
```

### Métodos útiles

```python
persona.keys()      # Claves
persona.values()    # Valores
persona.items()     # Pares clave-valor
```

### Acceso seguro

```python
print(persona.get("telefono", "No disponible"))
```

---

## Tuplas

Las tuplas son colecciones ordenadas e inmutables. Se definen con paréntesis `()`.

### Definición

```python
coordenadas = (39.47, -0.38)
```

### Acceso

```python
print(coordenadas[0])  # 39.47
```

### Conversión

```python
lista = list(coordenadas)
tupla = tuple(lista)
```

---

## Comparativa con Java

| Concepto        | Java                          | Python                        |
|-----------------|-------------------------------|-------------------------------|
| Lista mutable   | `ArrayList<String>`           | `list`                        |
| Diccionario     | `HashMap<String, Object>`     | `dict`                        |
| Tupla (no existe) | No equivalente directo       | `tuple`                       |

---

## Actividades

### Actividad 1: Lista de tareas

**Objetivo:** Crear una lista de tareas pendientes y permitir al usuario añadir, eliminar y mostrar tareas.

**Requisitos:**
- Uso de listas.
- Métodos `append()`, `remove()`, `print()`.

**Ejemplo de salida:**
```
Tareas actuales: ['Estudiar Python', 'Preparar examen']
Añade una tarea: Comprar café
Tareas actualizadas: ['Estudiar Python', 'Preparar examen', 'Comprar café']
```

---

### Actividad 2: Diccionario de usuario

**Objetivo:** Crear un diccionario que almacene información de un usuario y mostrarla formateada.

**Requisitos:**
- Uso de claves y valores.
- Acceso con `get()` y f-strings.

**Ejemplo de salida:**
```
Nombre: Sergio
Edad: 35
Ciudad: Valencia
Email: No disponible
```

---

### Actividad 3: Coordenadas y conversión

**Objetivo:** Definir una tupla con coordenadas, convertirla en lista, modificarla y volver a tupla.

**Requisitos:**
- Uso de `tuple()` y `list()`.
- Acceso por índice.

**Ejemplo de salida:**
```
Coordenadas originales: (39.47, -0.38)
Coordenadas modificadas: (39.47, -0.40)
```

---

## Cierre de la unidad

Las estructuras de datos básicas permiten representar información de forma flexible y eficiente. En el desarrollo de módulos, estas estructuras se utilizan para modelar registros, almacenar configuraciones y gestionar colecciones de objetos. En la próxima unidad, abordaremos las funciones, que permiten encapsular lógica y reutilizar código de forma estructurada.



---
---
---

# Unidad 4: Funciones en Python

## Objetivos

- Comprender la definición y uso de funciones en Python.
- Aplicar el paso de argumentos y el retorno de valores.
- Utilizar funciones con parámetros por defecto y nombrados.
- Entender el ámbito de las variables (`local` y `global`).
- Documentar funciones correctamente con docstrings.
- Modularizar el código para mejorar su legibilidad y mantenimiento.

---

## ¿Qué es una función?

Una función es un bloque de código que realiza una tarea específica y puede ser reutilizado. Permite dividir el programa en partes más pequeñas y organizadas.

---

## Definición de funciones

### Sintaxis básica

```python
def saludar():
    print("Hola, bienvenido al curso de Python.")
```

### Llamada a la función

```python
saludar()
```

---

## Parámetros y argumentos

### Función con parámetros

```python
def saludar_usuario(nombre):
    print(f"Hola {nombre}, ¿cómo estás?")
```

```python
saludar_usuario("Sergio")
```

### Retorno de valores

```python
def sumar(a, b):
    return a + b

resultado = sumar(3, 5)
print(resultado)  # 8
```

---

## Parámetros por defecto

```python
def saludar(nombre="usuario"):
    print(f"Hola {nombre}")
```

```python
saludar()            # Hola usuario
saludar("Sergio")    # Hola Sergio
```

---

## Argumentos nombrados

```python
def mostrar_datos(nombre, edad):
    print(f"Nombre: {nombre}, Edad: {edad}")

mostrar_datos(edad=35, nombre="Sergio")
```

---

## Ámbito de las variables

### Local

```python
def ejemplo():
    x = 10  # variable local
    print(x)
```

### Global

```python
x = 5

def modificar():
    global x
    x = 10

modificar()
print(x)  # 10
```

---

## Documentación con docstrings

```python
def saludar(nombre):
    """
    Esta función muestra un saludo personalizado.
    Parámetros:
        nombre (str): nombre del usuario
    """
    print(f"Hola {nombre}")
```

---

## Modularización

Las funciones permiten dividir el código en módulos reutilizables. Esto mejora la legibilidad, facilita el mantenimiento y permite trabajar en equipo.

Ejemplo de estructura modular:

```python
# archivo: utilidades.py
def convertir_euros_a_dolares(euros):
    return euros * 1.08
```

```python
# archivo principal
from utilidades import convertir_euros_a_dolares

print(convertir_euros_a_dolares(100))
```

---

## Actividades

### Actividad 1: Calculadora básica

**Objetivo:** Crear funciones para sumar, restar, multiplicar y dividir dos números.

**Requisitos:**
- Definir 4 funciones.
- Solicitar datos al usuario.
- Mostrar resultados con `print()`.

**Ejemplo de salida:**
```
Introduce el primer número: 10
Introduce el segundo número: 5
Suma: 15
Resta: 5
Multiplicación: 50
División: 2.0
```

---

### Actividad 2: Conversor de moneda

**Objetivo:** Crear una función que convierta euros a dólares, con un parámetro por defecto para el tipo de cambio.

**Requisitos:**
- Uso de parámetros por defecto.
- Retorno de valor.
- f-string para mostrar el resultado.

**Ejemplo de salida:**
```
Introduce cantidad en euros: 100
Con 100 euros obtienes 108.0 dólares.
```

---

### Actividad 3: Generador de saludo personalizado

**Objetivo:** Crear una función que reciba nombre y hora del día, y devuelva un saludo adecuado.

**Requisitos:**
- Uso de argumentos nombrados.
- Condicionales dentro de la función.
- Retorno de cadena.

**Ejemplo de salida:**
```
Introduce tu nombre: Sergio
Introduce la hora (0-23): 9
Buenos días, Sergio.
```

---

## Cierre de la unidad

Las funciones son esenciales para estructurar el código de forma clara y reutilizable. En el desarrollo de módulos, permiten encapsular lógica de negocio, validar datos y definir comportamientos específicos. En la próxima unidad, abordaremos la **gestión de paquetes con pip y entornos virtuales**, que será clave para trabajar con dependencias externas en proyectos reales.



---
---
---




# Unidad 5: Gestión de paquetes con pip y entornos virtuales

## Objetivos

- Comprender el propósito de `pip` como gestor de paquetes en Python.
- Instalar, listar y eliminar paquetes externos.
- Crear y activar entornos virtuales para aislar proyectos.
- Usar `requirements.txt` para compartir dependencias.
- Preparar el entorno para trabajar con librerías específicas en proyectos reales.

---

## ¿Qué es pip?

`pip` es el gestor de paquetes oficial de Python. Permite instalar librerías externas desde el repositorio [PyPI](https://pypi.org), como `requests`, `flask`, `pandas`, entre muchas otras.

### Comandos básicos

```bash
pip install nombre_paquete
pip list
pip uninstall nombre_paquete
```

### Ejemplo

```bash
pip install requests
```

---

## ¿Qué es un entorno virtual?

Un entorno virtual es una carpeta que contiene una instalación aislada de Python y sus paquetes. Permite trabajar en varios proyectos sin que sus dependencias interfieran entre sí.

### Creación del entorno

```bash
python -m venv nombre_entorno
```

Esto crea una carpeta con el entorno virtual.

### Activación

- En Windows (CMD o PowerShell):

```bash
nombre_entorno\Scripts\activate
```

- En WSL/Linux/macOS:

```bash
source nombre_entorno/bin/activate
```

### Desactivación

```bash
deactivate
```

---

## Uso de requirements.txt

Este archivo contiene la lista de paquetes necesarios para un proyecto.

### Exportar dependencias

```bash
pip freeze > requirements.txt
```

### Instalar desde archivo

```bash
pip install -r requirements.txt
```

---

## Buenas prácticas

- Crear un entorno virtual por proyecto.
- No instalar paquetes globalmente si no es necesario.
- Usar `requirements.txt` para compartir el entorno con otros desarrolladores.
- Activar el entorno antes de ejecutar el proyecto.

---

## Actividades

### Actividad 1: Crear y activar un entorno virtual

**Objetivo:** Crear un entorno virtual llamado `entorno_dam` y activarlo correctamente.

**Pasos:**
- Crear el entorno con `python -m venv entorno_dam`.
- Activarlo desde terminal.
- Verificar que el entorno está activo.

**Entrega:** Captura de pantalla del terminal con el entorno activado.

---

### Actividad 2: Instalar paquetes y exportar dependencias

**Objetivo:** Instalar los paquetes `requests` y `pandas`, y generar el archivo `requirements.txt`.

**Pasos:**
- Activar el entorno.
- Instalar los paquetes con `pip install`.
- Exportar dependencias con `pip freeze`.

**Entrega:** Archivo `requirements.txt` y captura de pantalla del comando `pip list`.

---

### Actividad 3: Script con librería externa

**Objetivo:** Crear un script que use la librería `requests` para hacer una petición HTTP a una API pública.

**Requisitos:**
- Crear el script `consulta_api.py`.
- Usar `requests.get()` para obtener datos.
- Mostrar parte del contenido recibido.

**Ejemplo de código:**

```python
import requests

response = requests.get("https://pokeapi.co/api/v2/pokemon/pikachu")
data = response.json()
print(f"Nombre: {data['name']}")
print(f"Altura: {data['height']}")
```

**Entrega:** Código funcional y captura de pantalla con la salida del script.

---

## Cierre de la unidad

La gestión de paquetes y entornos virtuales es fundamental para trabajar en proyectos Python de forma profesional. Permite mantener el entorno limpio, reproducible y preparado para integrar librerías externas. En la próxima unidad, abordaremos la **programación orientada a objetos**, que será clave para estructurar el comportamiento de los módulos.



---
---
---



# FALTAN Unidad 6 y 7