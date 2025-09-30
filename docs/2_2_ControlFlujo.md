---
title: Estructuras condicionales y control de flujo
description: Introducción a Python. Estructuras condicionales y control de flujo
---

En esta sesión se exploran las herramientas fundamentales que permiten tomar decisiones dentro de un programa. Las estructuras condicionales como `if`, `elif` y `else` permiten ejecutar diferentes bloques de código según se cumplan ciertas condiciones, lo que aporta flexibilidad y dinamismo al comportamiento del programa. Este tipo de lógica es esencial para modelar situaciones reales como validaciones, bifurcaciones o respuestas ante distintos escenarios.

También se aborda el control de flujo, que incluye instrucciones como `break`, `continue` y `pass`, utilizadas para modificar el comportamiento de los bucles o gestionar casos especiales. Estas herramientas permiten controlar con precisión cómo se ejecuta el código, evitando repeticiones innecesarias o errores en estructuras incompletas. Se explicará cómo y cuándo interrumpir o saltar partes del flujo de ejecución para mantener la lógica del programa clara y eficiente.

Por último, se introducen los **bucles** como estructuras que permiten repetir acciones de forma controlada. Se trabajará con `for` y `while`, analizando cómo recorrer colecciones, repetir tareas hasta que se cumpla una condición, y cómo combinar bucles con condicionales para construir programas más expresivos. 

El objetivo es dominar el flujo de ejecución completo: decidir, repetir y controlar:

- Comprender el uso de condicionales en Python.
- Aplicar estructuras de control para tomar decisiones en el código.
- Comparar la lógica condicional en Python con la de Java.
- Utilizar operadores lógicos y de comparación.
- Crear scripts interactivos que respondan a diferentes entradas del usuario.


## Condicionales en Python

El control de flujo permite que un programa tome decisiones y ejecute diferentes bloques de código según ciertas condiciones. En Python, esto se logra principalmente con las estructuras `if`, `elif` y `else`.

### Sintaxis básica

```python
# Definimos una variable con la edad del usuario
edad = 20

# Evaluamos si la edad es mayor o igual a 18
if edad >= 18:
    print("Eres mayor de edad.")
```

Este es el caso más simple: si la condición `edad >= 18` se cumple, se ejecuta el bloque indentado y la condición `if` finaliza con dos puntos (`:`). Si no, el programa continúa sin hacer nada.

###  Estructura completa: `if` - `elif` - `else`

Para más de una condiciones tenemos la estructura `if` - `elif` - `else`. Observar de nuevo la identación y el final de las líneas de los `if`, `elif` y `else`

!!!example "Ejemplo 1:"

    ```python
    # Evaluamos una nota numérica y mostramos la calificación correspondiente
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

Esta estructura permite evaluar múltiples condiciones de forma ordenada. Python las evalúa de arriba hacia abajo y ejecuta solo el primer bloque que cumpla la condición.

!!!example "Ejemplo 2:"

    ```python
    # Evaluamos la nota de un alumno con entrada por teclado
    nota = float(input("Introduce tu nota (0-10): "))

    if nota >= 9:
        print("¡Excelente! Has sacado un Sobresaliente.")
    elif nota >= 7:
        print("Muy bien, tu nota es Notable.")
    elif nota >= 5:
        print("Has aprobado. ¡Ánimo para mejorar!")
    else:
        print("Lo siento, has suspendido.")
    ```

!!!warning "Importancia de la **indentación**"

    En Python, los bloques de código se definen por **indentación** (espacios), no por llaves `{}` como en Java.  
    Se recomienda usar **4 espacios** por nivel.  
    Una indentación incorrecta provocará errores de sintaxis (`IndentationError`).


## Operadores

Los operadores permiten comparar valores y construir condiciones lógicas. Son esenciales para trabajar con estructuras condicionales.

### Operadores de comparación

En la siguiente tabla tenemos los operadores típicos de Python.ç

| Operador | Significado        | Ejemplo        | Resultado |
|----------|--------------------|----------------|-----------|
| `==`     | Igual a            | `x == 5`       | `True`    |
| `!=`     | Distinto de        | `x != 3`       | `True`    |
| `<`      | Menor que          | `x < 10`       | `True`    |
| `>`      | Mayor que          | `x > 2`        | `True`    |
| `<=`     | Menor o igual que  | `x <= 5`       | `True`    |
| `>=`     | Mayor o igual que  | `x >= 5`       | `True`    |

Estos operadores devuelven valores booleanos (`True` o `False`) que se usan en condiciones. Por ejemplo, `if x != 3:` se ejecutará si `x` es distinto de 3.

### Operadores lógicos

| Operador | Descripción        | Ejemplo                  | Resultado |
|----------|--------------------|--------------------------|-----------|
| `and`    | Ambas condiciones  | `x > 5 and x < 10`       | `True`    |
| `or`     | Una u otra         | `x < 3 or x > 8`         | `True`    |
| `not`    | Negación           | `not x == 5`             | `False`   |

Los operadores lógicos permiten combinar condiciones. Por ejemplo, `if edad >= 18 and pais == "España":` verifica dos requisitos a la vez.

!!!example "Ejemplo entrada de datos"

    ```python
    # Verificamos si un usuario puede acceder a una plataforma
    usuario = input("Introduce tu nombre: ")
    edad = int(input("Introduce tu edad: "))
    pais = input("¿En qué país vives?: ")

    if edad >= 18 and pais.lower() == "españa":
        print(f"{usuario}, puedes acceder al sistema desde España.")
    else:
        print(f"{usuario}, acceso denegado por edad o país.")
    ```

Este ejemplo combina entrada de datos, conversión de tipos (`int()`), condicionales y salida formateada con `f-strings`. Es ideal para introducir lógica básica en Python.


!!!tip "Buenas prácticas"

    - Usa nombres de variables descriptivos.
    - Evita condicionales anidados innecesarios.
    - Comenta el propósito de cada bloque si no es evidente.
    - Usa `elif` en lugar de múltiples `if` independientes.

---

## Estructura `match` en Python (equivalente a `switch`/`case`)

En otros lenguajes como Java, C o JavaScript, existe la estructura `switch`/`case` para evaluar múltiples condiciones sobre una misma variable. Python no tuvo una alternativa directa hasta la versión **3.10**, donde se introdujo la instrucción `match`, también conocida como **pattern matching**.

Esta nueva sintaxis permite escribir código más limpio y legible cuando se necesita comparar un valor contra múltiples opciones.

### Instrucción `match`

La instrucción `match` permite comparar un valor con diferentes patrones. Cada patrón se define con la palabra clave `case`, y se ejecuta el bloque correspondiente si hay coincidencia.

!!!example "Evaluando el día de la semana" 

    ```python
    # Evaluamos el día de la semana usando match-case
    dia = "lunes"

    match dia:
        case "lunes":
            print("Inicio de semana")
        case "viernes":
            print("Último día laboral")
        case "sábado" | "domingo":
            print("Fin de semana")
        case _:
            print("Día no reconocido")
    ```

    Donde:

      - `match dia:` inicia la evaluación del valor de la variable `dia`.
      - Cada `case` representa una posible coincidencia.
      - El símbolo `|` permite agrupar varias opciones en un mismo caso.
      - El patrón `_` actúa como **comodín** o **caso por defecto**, similar a `default` en otros lenguajes.

### Comparación con `if` - `elif` - `else`

Aunque `if` sigue siendo más flexible, `match` es más limpio cuando se evalúa una sola variable contra múltiples valores.

!!!example "Elección de día con `if`-`elif`-`else`"
    ```python
    # Versión equivalente con if-elif-else
    dia = "lunes"

    if dia == "lunes":
        print("Inicio de semana")
    elif dia == "viernes":
        print("Último día laboral")
    elif dia == "sábado" or dia == "domingo":
        print("Fin de semana")
    else:
        print("Día no reconocido")
    ```

**Ventajas de `match`:**  

- Menor repetición de la variable.
- Más legible en estructuras largas.
- Permite patrones más avanzados (como desestructuración).

### Uso con valores numéricos

!!!example "Ejemplo de `match` con números"

    ```python
    # Evaluamos una nota numérica
    nota = 8

    match nota:
        case 10:
            print("Matrícula de honor")
        case 9:
            print("Sobresaliente")
        case 7 | 8:
            print("Notable")
        case 5 | 6:
            print("Aprobado")
        case _:
            print("Suspenso")
    ```

    Este ejemplo muestra cómo agrupar varios valores en un mismo `case`. Es útil cuando varios valores deben producir la misma salida.

### Uso con tuplas y desestructuración

En la siguiente sesión veremos veremos datos más avanzados como duplas que veremos en la siguiente sesión, pero, aun así en este punto enunciaremos que una de las características más potentes de `match` es su capacidad para trabajar con estructuras de datos como tuplas o listas.


!!!examole "Evaluación con tuplas"

    ```python
    # Evaluamos coordenadas en un plano
    coordenada = (0, 0)

    match coordenada:
        case (0, 0):
            print("Origen")
        case (x, 0):
            print(f"Eje X en la posición {x}")
        case (0, y):
            print(f"Eje Y en la posición {y}")
        case (x, y):
            print(f"Punto en ({x}, {y})")
    ```

    - Se puede extraer el contenido de la tupla directamente en el `case`.
    - Si la estructura coincide, se asignan los valores a las variables `x` e `y`.

### Caso por defecto con `_`

El patrón `_` se utiliza para capturar cualquier valor que no haya coincidido con los casos anteriores. Es el típico `default` que podemos ver en cualquier otro lenguaje.

Veamos de nuevo otro ejemplo.

!!!example "Ejemplo de menú típico para el caso de cualquier otro valor"

    ```python
    # Evaluamos una opción de menú
    opcion = "salir"

    match opcion:
        case "iniciar":
            print("Sistema iniciado")
        case "configurar":
            print("Abriendo configuración")
        case _:
            print("Opción no válida")
    ```

    Siempre es recomendable incluir un `case _` para manejar entradas inesperadas o errores del usuario.

También puede ser útil cuando solicitamos unos valores concretos, y el usuario no teclea uno de estos valores

!!!example "Ejemplo completo con entrada del usuario"

    ```python
    # Simulamos un sistema de selección de rol
    rol = input("Introduce tu rol (admin, editor, lector): ")

    match rol.lower():
        case "admin":
            print("Acceso completo al sistema.")
        case "editor":
            print("Acceso limitado a edición de contenidos.")
        case "lector":
            print("Acceso solo lectura.")
        case _:
            print("Rol no reconocido.")
    ```

    en este ejemplo:  

    - Se usa `.lower()` para normalizar la entrada del usuario.
    - Se evalúan diferentes roles con `match`.
    - Se incluye un caso por defecto para manejar errores.

## Estructuras de control de bucles en Python

En Python como en otros lenguajes, los bucles permiten ejecutar un bloque de código varias veces. Son fundamentales para automatizar tareas repetitivas y recorrer estructuras de datos como listas, diccionarios o cadenas.

Repasemos los más usados

### Bucle `for`

El bucle `for` se utiliza para **iterar sobre una secuencia** (como una lista, tupla, diccionario, conjunto o cadena).

**Sintaxis**

```python
for elemento in secuencia:
    # Bloque de código
```

El ejemplo típico en el que iteramos un determinado número de veces, se apoya en el uso del comando `range()`. El siguiente ejemplo puede ser muy representativo

!!!example "Usar `range()` para iterar 5 veces"

    ```python
    for i in range(5):
        print("Número:", i)
    ```

    `range(5)` genera los números del 0 al 4. 

Realmente la función `range()` se define de con dos parámetros, de forma que si solo se invoca con uno de ellos, se interpreta que el primero es `0`. Podemos ver con el ejemplo que el resultado incluye el valor de inicio pero excluye el de fin, por eso usamos `range( 1, 6)` para obtener del `1` al `5`.


!!!example "range() con dos parámetros"

    ```python
    for i in range(1, 6):
        print("Número:", i)
    ```

    Este bucle imprime los números del 1 al 5, uno por línea

pero `range()` también se puede invocar con **3 parámetros**, siendo el tercer parámetro el incremento del rango:


!!!example "`range()` con tres parámetros"

    ```python
    for i in range(1, 11, 2):
        print("Número:", i)
    ```

    Este bucle imprime los números impares del 1 al 10, uno por línea

y para acabar con `range()` veamos el ejemplo típico de la tabla de multiplicar:

!!!example "Tabla de multiplicar con `for` y `range()` "

    ```python
    tabla_del = 5
    for i in range(1, 11):
        print(f"{tabla_del} x {i} = {tabla_del * i}")
    ```
De nuevo vamos a utilizar una estructura que veremos en la siguiente sesión como son las *listas*, así pues, siguiente ejemplo visualiza que podemos iterar por los elementos de una lista.

!!!example  "Ejemplo de interacción sobre una lista"

    ```python
    frutas = ["manzana", "plátano", "cereza"]
    for fruta in frutas:
        print("Me gusta la", fruta)
    ```

    Este bucle recorre cada elemento de la lista `frutas` y lo imprime.

Y ya que estamos, existe la posibilidad de crear una *lista* utilizando la fución `range()`, veamos el siguiente ejemplo: 

!!!example  "Crear una lista con `range()` "

    ```python
    numeros = list(range(1, 6))
        print("Lista:", numeros)
    ```

    Convierte el rango en una lista: `[ 1, 2, 3,4, 5]`.

Se pueden colocar **bucles dentro de otros bucles**. Por supuesto, en Python podemos **anidar** bucle sin problemas. Veamos un ejemplo:

!!!example "Bucle anidado"

    ```python
    for i in range(3):
        for j in range(2):
            print(f"i={i}, j={j}")
    ```

    Este ejemplo muestra cómo se combinan dos bucles `for` para recorrer pares de valores.

!!!question "Ejercicio propuesto con `for`:"

    Crea un programa que pida al usuario un número y muestre su tabla de multiplicar del 1 al 10 usando un bucle `for`."

    ???quote "Solucion"

        ```python
        numero = int(input("Introduce un número: "))
        for i in range(1, 11):
            print(f"{numero} x {i} = {numero * i}")
        ```
    > **Variante**: Consigue que la tabla multiplicar la imprima al revés, del 10 al 1

### Bucle `while`

El bucle `while` ejecuta su bloque de código **mientras una condición sea verdadera**.

**Sintaxis**

```python
while condición:
    # Bloque de código
```

!!!example "Contador simple con `while`"

    ```python
    contador = 0
    while contador < 5:
        print("Contador:", contador)
        contador += 1
    ```

    Este bucle se repite mientras `contador` sea menor que 5.

En generar podemos utilizar todos los operadores indicados anteriormente con el `if` así que tenemos ejemplos con todo tipo de validaciones:


!!!example "Validación de entrada"

    ```python
    respuesta = ""
    while respuesta != "sí":
        respuesta = input("¿Quieres continuar? (sí/no): ")
    ```

    Este bucle sigue preguntando hasta que el usuario responde "sí".

!!!question "Ejercicio sencillo con `while`"

    Repite de nuevo el bucle de la tabla de multiplicar, pero ahora con  `while`.

    ???quote "Solución"

        ```python
        numero = int(input("Introduce un número: "))
        i = 1  # Inicializamos el contador
        while i <= 10:
            print(f"{numero} x {i} = {numero * i}")
            i += 1  # Incrementamos el contador
        ```



###  Control de bucles

Al igual que ocurre con otros lenguajes, tenemos dos comandos que permite alterar el control de los bucles, en este caso `break` para interrumpir la ejecución del bucle, ya que provoca que el bucle finalice al pasar por esta función y `continue` que provoca el salto del bucle a la siguiente iteracion, quedando las instrucciones siguiente a `continue` sin ejecución en la iteración correspondiente. 

Veamos dos ejemplos sencillos

**`break`**

!!!example "`break`: salir del bucle"

    ```python
    for letra in "Python":
        if letra == "h":
            break
        print(letra)
    ```

    El bucle se detiene cuando encuentra la letra "h".

y **`continue`**

!!!example "`continue`: saltar a la siguiente iteración"

    ```python
    for numero in range(5):
        if numero == 2:
            continue
        print("Número:", numero)
    ```

    Cuando `numero` es 2, se omite esa iteración.



!!!tip "Buenas prácticas con los bucles"

    - Evita bucles infinitos: asegúrate de que la condición del `while` se pueda cumplir.
    - Usa nombres de variables descriptivos.
    - Si puedes usar `for` en lugar de `while`, hazlo: suele ser más legible.
    - No abuses de `break` y `continue`: pueden dificultar la comprensión del flujo.



!!!question "Ejercicio propuesto tablas de multiplicar"

    Amplia el programa anterior de las tablas de multiplicar para que te pregunte de qué numero quiere hacer la tabla de multiplicar, genere la tabla de multiplicar y termine cuando le pidas la tabla del 0"

    ???quote "Solucion"

        ```python
        while True:
            numero = int(input("Introduce un número (0 para salir): "))
            
            if numero == 0:
                print("¡Hasta luego!")
                break  # Sale del bucle si el número es 0
            
            print(f"\nTabla de multiplicar del {numero}:")
            for i in range(1, 11):
                print(f"{numero} x {i} = {numero * i}")
            print()  # Línea en blanco para separar tablas
        ```

    > **Variante**: si pides las tablas del 5 al 7, solo haga hasta el 5 y no hasta el 10



## 🧩 Ejemplo y ejercicio de uso

```python
# Mostrar si los números del 1 al 10 son pares o impares
for numero in range(1, 11):
    if numero % 2 == 0:
        print(f"{numero} es par")
    else:
        print(f"{numero} es impar")
```

Este ejemplo combina un bucle `for` con una estructura `if-else` para analizar cada número y mostrar su tipo.

!!!question "Ejercicio: Contador de positivos y negativos"

    Escribe un programa que:

    1. Pida al usuario 5 números (uno por uno).
    2. Use un bucle para recoger los datos.
    3. Use condiciones para contar cuántos son positivos y cuántos negativos.
    4. Al final, muestre los totales.

    ???quote "Solución"

        ```python
        positivos = 0
        negativos = 0

        for i in range(5):
            numero = int(input("Introduce un número: "))
            if numero > 0:
                positivos += 1
            elif numero < 0:
                negativos += 1

        print(f"Números positivos: {positivos}")
        print(f"Números negativos: {negativos}")
        ```
