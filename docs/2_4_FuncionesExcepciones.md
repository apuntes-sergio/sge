---
title: Funciones y manejo de excepciones
description: Introducción a Python. Funciones y manejo de excepciones
---

Una función en Python es como una pequeña herramienta que realiza una tarea específica. En lugar de repetir el mismo bloque de código varias veces, podemos escribirlo una sola vez dentro de una función y luego llamarla cada vez que la necesitemos. Esto hace que el código sea más limpio, más fácil de entender y mucho más sencillo de mantener. Por ejemplo, si queremos saludar a varias personas, en lugar de escribir `print("Hola, Sergio")`, `print("Hola, Lucía")`, etc., podemos crear una función que reciba el nombre como entrada y se encargue de mostrar el saludo. Así, cada vez que queramos saludar a alguien, solo tenemos que llamar a esa función con el nombre correspondiente.

Además de aprender a crear funciones, en esta sesión aprovechamos para introducir el manejo de errores mediante excepciones. En muchos programas, pueden surgir situaciones inesperadas: por ejemplo, intentar dividir entre cero, acceder a un dato que no existe o introducir un valor no válido. Las excepciones permiten detectar esos errores y responder de forma controlada, evitando que el programa se detenga bruscamente. Aprender a usar `try`, `except` y `raise` nos ayuda a escribir código más robusto y profesional, preparado para funcionar correctamente incluso cuando algo no sale como se esperaba.

Asi pues, lo contenidos que vamos a abordar en esta sección son: 

- Definición de funciones con `def`.
- Parámetros posicionales, nombrados y con valores por defecto.
- Ámbito de variables: local y global.
- Uso de `return` para devolver resultados.
- Documentación interna con docstrings (`""" """`).
- Manejo de errores con `try`, `except`, `finally`, `raise`.

## Definición básica de una función

Las funciones permiten dividir un programa en bloques lógicos que realizan tareas específicas. Esto mejora la legibilidad, facilita la reutilización del código y permite detectar errores más fácilmente. En proyectos reales, como los desarrollos en Odoo, la modularidad es clave para mantener el código organizado y escalable.

La sintaxis en la definición de una función en Python viene determinada por `def`

!!!example "Ejemplo de definición de una función `saludar` con un parámetro"

    ```python
    def saludar(nombre):
        """Muestra un saludo personalizado."""
        print(f"Hola, {nombre}")
    ```

    Esta función recibe un argumento (`nombre`) y muestra un mensaje. El docstring explica brevemente qué hace la función.

### Parámetros por defecto y nombrados

Los parámetros por defecto permiten definir valores que se usarán si no se especifican en la llamada. Para definir un parámetro por defecto, basta con asignar un valor mediante un `=` en la cabecera de la función

!!!example "Ejemplo de función con parámetro por defecto"

    ```python
    def saludar(nombre="invitado"):
        print(f"Hola, {nombre}")

    saludar("Sergio")  # Hola, Sergio
    saludar()          # Hola, invitado
    ```

También se pueden usar argumentos nombrados para mayor claridad:

```python
saludar(nombre="Lucía")
```

En Python, **no puedes definir un valor por defecto para un parámetro si hay otro parámetro sin valor por defecto después de él**. Es decir, los parámetros con valores por defecto deben ir **al final** de la lista de parámetros.

Por ejemplo, esto **no es válido**:

!!!warning "Incorrecto"

    ```python
    def ejemplo(a=1, b, c=3):  # ❌ Error de sintaxis
        pass
    ```

Python lanzará un `SyntaxError` porque `b` no tiene valor por defecto, pero está entre dos parámetros que sí lo tienen.

En cambio, esto **sí es válido**:

!!!succces "Correcto" 

    ```python
    def ejemplo(a, b=2, c=3):  # ✅ Correcto
        pass
    ```

Si necesitas flexibilidad en el orden, puedes usar **argumentos nombrados** al llamar a la función, o bien reestructurar la definición para que los parámetros sin valor por defecto estén primero. 


!!!tip "La palabra clave **`pass`**"

    En Python, la palabra clave `pass` se utiliza como una **instrucción vacía**. Sirve para indicar que no se va a ejecutar ninguna acción en ese lugar del código, pero que la estructura está ahí por motivos de sintaxis o planificación.

    Por ejemplo, si estás definiendo una función pero aún no has decidido qué va a hacer, puedes escribir:

    ```python
    def mi_funcion():
        pass
    ```

    Esto evita que el intérprete lance un error por tener una función sin contenido. Lo mismo ocurre con clases, bucles o condicionales. Es como decir: “aquí irá algo más adelante, pero por ahora déjalo en blanco”.

    También se usa cuando necesitas que el programa siga funcionando sin hacer nada en ciertos casos:

    ```python
    for letra in "Python":
        if letra == "h":
            pass  # No hacemos nada con la letra 'h'
        else:
            print(letra)
    ```

    Es útil para mantener la estructura del código mientras lo desarrollas o cuando quieres ignorar ciertos casos sin romper la lógica del programa.



### Funciones con número de parámetros variables

En Python, a veces necesitamos definir funciones que puedan recibir una cantidad **variable de argumentos**, sin saber cuántos serán exactamente. Esto es útil cuando queremos que la función sea flexible y se adapte a diferentes situaciones sin tener que modificar su definición.

Para ello, Python ofrece dos mecanismos:

- `*args`: permite recibir cualquier número de **argumentos posicionales**.
- `**kwargs`: permite recibir cualquier número de **argumentos nombrados** (clave-valor).

También se pueden combinar ambos para crear funciones muy versátiles.

#### Uso de `*args` (argumentos posicionales variables)

Cuando usamos `*args` en la definición de una función, estamos diciendo que esa función puede recibir **muchos valores sin nombre**, que serán agrupados en una **tupla**. Esto es útil cuando queremos operar con una lista de números, textos u otros elementos sin saber cuántos serán.

Se puede ver más claro en los siguientes ejemplos:

!!!example "Ejemplo 1: Sumar varios números pasados como parámetros"

    ```python
    def sumar_todos(*numeros):
        """Suma todos los números que se pasen como argumentos."""
        return sum(numeros)

    print(sumar_todos(2, 4, 6))       # 12
    print(sumar_todos(1, 3, 5, 7, 9)) # 25
    ```

!!!example "Ejemplo mostramos los parámetros pasados"

    ```python
    def mostrar_nombres(*nombres):
        print("Lista de nombres:")
        for nombre in nombres:
            print("-", nombre)

    mostrar_nombres("Lucía", "Carlos", "Marta")
    mostrar_nombres("Sergio")
    ```

En ambos casos, la función puede recibir uno, tres o veinte argumentos sin cambiar su definición.

#### Uso de `**kwargs` (argumentos nombrados variables)

Cuando usamos `**kwargs`, la función puede recibir muchos **argumentos con nombre**, que serán agrupados en un **diccionario**. Esto es útil cuando queremos que el usuario pueda especificar información sin seguir un orden fijo.

De nuevo comprenderemos mejor su uso con un par de ejemplos. Observar como los parámetros pasados deben tener formato de **diccionario**:

!!!example "Mostrar información de usuario"

    ```python
    def mostrar_info(**datos):
        for clave, valor in datos.items():
            print(f"{clave}: {valor}")

    mostrar_info(nombre="Sergio", edad=25, ciudad="Alberic")
    ```

Observar cómo se han recuperado tanto la clave como el valor de cada tupla del diccionario

Veamos de nuevo otro ejemplo similar

!!!example "Ejemplo de lectura de configuración"

    ```python
    def configurar(**opciones):
        print("Configuración aplicada:")
        for clave, valor in opciones.items():
            print(f"{clave} = {valor}")

    configurar(color="azul", tamaño="mediano", activo=True)
    ```

En estos ejemplos, los argumentos se pasan como pares clave-valor, y la función los procesa como un diccionario.

#### Combinación de parámetros fijos, `*args` y `**kwargs`

También se pueden combinar parámetros normales con `*args` y `**kwargs` en una misma función. El orden debe ser:

1. Parámetros normales
2. `*args`
3. `**kwargs`

Esto permite que la función tenga una parte fija, una parte variable de datos, y una parte flexible de configuración.

De nuevo veamos un par de ejemplos:

!!!example "Procesar datos con nombre y valores"

    ```python
    def procesar(nombre, *valores, **opciones):
        print("Nombre:", nombre)
        print("Valores:", valores)
        print("Opciones:", opciones)

    procesar("Lucía", 10, 20, activo=True, nivel="avanzado")
    ```
Observar en este ejemplo como el primer parámetro de la invocación de la función pertenece al parámetro fijo y el resto de parámetros simple se convierten en una lista para los parámetros posicionales y los parámetros pasados como tuplas clave-valor se convierten en los parámetros finales en modo diccionario.

Veamos otro ejemplo similar

!!!example "Enviar mensaje personalizado"

    ```python
    def enviar_mensaje(destinatario, *lineas, **firma):
        print(f"Para: {destinatario}")
        for linea in lineas:
            print(linea)
        if "nombre" in firma:
            print(f"\nAtentamente,\n{firma['nombre']}")

    enviar_mensaje("Carlos", "Hola,", "¿Cómo estás?", nombre="Sergio")
    ```

Estas funciones permiten construir estructuras muy flexibles, ideales para proyectos reales donde los datos pueden variar según el contexto.


#### 🧩 Ejemplo y ejercicio de uso

En el siguiente ejemplo se muestra cómo definir una función que acepta una cantidad variable de argumentos posicionales (`*args`) y nombrados (`**kwargs`). La función simula el registro de un pedido, donde se pueden añadir productos sin límite y configurar opciones adicionales como envío o método de pago.

Este ejemplo permite aplicar de forma conjunta los conceptos de funciones, `*args`, `**kwargs`, bucles, y acceso a diccionarios.

```python
def registrar_pedido(cliente, *productos, **configuracion):
    """Registra un pedido con productos y opciones adicionales."""
    print(f"Pedido de: {cliente}")
    
    if productos:
        print("Productos solicitados:")
        for producto in productos:
            print("-", producto)
    else:
        print("No se han añadido productos.")

    if configuracion:
        print("Configuración del pedido:")
        for clave, valor in configuracion.items():
            print(f"{clave}: {valor}")
    else:
        print("Sin configuración adicional.")

# Ejemplo de uso
registrar_pedido("Lucía", "Teclado", "Ratón", envio="urgente", pago="tarjeta")
registrar_pedido("Carlos")
```

Este ejemplo muestra cómo se pueden combinar argumentos fijos, múltiples valores y opciones flexibles en una sola función.

!!!question "Ejercicio básico: Registro de usuario con parámetros variables"

    Crea una función llamada `registrar_usuario` que reciba:

    - Un argumento obligatorio: `nombre`
    - Una cantidad variable de intereses (`*intereses`)
    - Una cantidad variable de datos personales (`**datos`)

    La función debe:

    1. Mostrar el nombre del usuario.
    2. Mostrar los intereses si se han indicado.
    3. Mostrar los datos personales como pares clave-valor si se han indicado.

    Prueba la función con los siguientes casos:

    - `registrar_usuario("Sergio", "Python", "IA", ciudad="Valencia", edad=25)`
    - `registrar_usuario("Lucía")`

    > Pistas  
    > - Usa `*intereses` para recoger temas de interés.  
    > - Usa `**datos` para recoger información adicional.  
    > - Usa bucles para mostrar los contenidos.

    ???quote "Solución"

        ```python
        def registrar_usuario(nombre, *intereses, **datos):
            print(f"Usuario registrado: {nombre}")
            
            if intereses:
                print("Intereses:")
                for interes in intereses:
                    print("-", interes)
            else:
                print("No se han indicado intereses.")

            if datos:
                print("Datos personales:")
                for clave, valor in datos.items():
                    print(f"{clave}: {valor}")
            else:
                print("No se han indicado datos personales.")

        # Pruebas
        registrar_usuario("Sergio", "Python", "IA", ciudad="Valencia", edad=25)
        registrar_usuario("Lucía")
        ```


### Retorno de valores

Una función puede devolver un resultado con `return`, lo que permite usar ese valor en otras partes del programa.

!!!example "Ejemplo de función con devolución de resultado"

    ```python
    def sumar(a, b):
        return a + b

    resultado = sumar(3, 5)
    print("Resultado:", resultado)
    ```

## Ámbito de variables

Tal y como sucede en otros lenguajes, debemos diferenciar entre variables **locales** y **globales**. Las variables definidas **dentro de una función** son **locales** y no afectan al resto del programa. Las variables **globales** pueden ser accedidas **desde cualquier parte**, pero deben usarse con **cuidado** para evitar **conflictos**.

### Variables locales

Una variable local es aquella que se define **dentro de una función** y solo existe mientras esa función se está ejecutando. No puede ser accedida desde fuera.

!!!warning "Error en uso de variable local" 

    ```python
    def saludar():
        mensaje = "Hola desde dentro de la función"
        print(mensaje)

    saludar()
    print(mensaje)  # ❌ Esto da error: 'mensaje' no está definida fuera de la función
    ```

En este ejemplo, `mensaje` solo existe dentro de `saludar()`. Al intentar usarla fuera, Python no la reconoce.

### Variables globales

Una variable global se define **fuera de cualquier función** y puede ser utilizada dentro de funciones, pero solo para leer su valor. Si se intenta modificarla directamente dentro de una función, Python la tratará como una nueva variable local, a menos que se indique lo contrario.

!!!success "Uso *correcto* de variable global"

    ```python
    contador = 0  # Variable global

    def mostrar_contador():
        print("Contador global:", contador)

    mostrar_contador()  # ✅ Se puede leer sin problema
    ```

Aquí, `contador` es global y puede ser leído desde dentro de la función sin necesidad de hacer nada especial.

### Variables con el mismo nombre: local vs. global

Si dentro de una función se define una variable con el mismo nombre que una global, la versión local **oculta** la global dentro de ese ámbito.

!!!example "Ejemplo de ámbito de uso de variables local y global"

    ```python
    mensaje = "Hola desde fuera"

    def cambiar_mensaje():
        mensaje = "Hola desde dentro"
        print("Dentro de la función:", mensaje)

    cambiar_mensaje()
    print("Fuera de la función:", mensaje)
    ```

En este caso, aunque ambas variables se llaman `mensaje`, son distintas. La función usa su propia versión local y no modifica la global.

### Modificar una variable global desde una función

Si realmente se quiere **modificar** una variable global desde dentro de una función, hay que usar la palabra clave `global`.

!!!example "Ejemplo de modificación de variable global desde función"

    ```python
    contador = 0

    def incrementar():
        global contador
        contador += 1

    incrementar()
    print("Contador después de la función:", contador)  # 1
    ```

Aquí, al declarar `global contador`, le decimos a Python que queremos usar la variable global y no crear una nueva local. Esto permite modificar su valor desde dentro de la función.

---

## Manejo de excepciones en Python

En Python, una excepción es un tipo de error que ocurre mientras el programa se está ejecutando. A diferencia de los errores de sintaxis, que impiden que el programa arranque, las excepciones aparecen cuando algo inesperado sucede durante la ejecución: por ejemplo, dividir entre cero, acceder a una posición inexistente en una lista o convertir una cadena que no es un número con `int()`.

Cuando ocurre una excepción y no se gestiona, el programa se detiene y muestra un mensaje de error. Para evitar esto, Python permite capturar y manejar excepciones usando las instrucciones `try`, `except`, `finally` y `raise`. Esto permite que el programa continúe funcionando de forma controlada, incluso si algo falla.

El manejo de excepciones es especialmente útil cuando se trabaja con entrada de datos, operaciones matemáticas, acceso a archivos o cualquier situación en la que el comportamiento del usuario o del entorno pueda provocar errores. Saber identificar y tratar estos errores es fundamental para escribir programas robustos y profesionales.

### Uso de `try` `except`

Veamos un ejemplo sencillo de captura de excepciones.

!!!example "Captura de excepción"

    ```python
    try:
        x = int("abc")
    except:
        print("Algo salió mal")  # Captura cualquier excepción
    ```

En este ejemplo, se captura el error aunque no se especifique que es un . Sin embargo, si se quiere saber qué tipo de error ocurrió, se puede capturar el objeto de la excepción:

!!!example "Captura de excepción obteniendo el mensaje de la excepción" 

    ```python
    try:
        x = int("abc")
    except Exception as e:
        print("Error:", e)
    ```

Esto permite mostrar el mensaje del error sin detener el programa.

Pero también podemos esperar capturar un excepción en concreto, por ejemplo la típica de división por cero.

!!!example "Ejemplo de captura de excepción de división entre cero"

    ```python
    def dividir(a, b):
        try:
            return a / b
        except ZeroDivisionError:
            print("Error: No se puede dividir entre cero.")
            return None
    ```

Y podemos complicar mucho más la gestón de excepciones. En el siguiente ejemplo se muestra cómo capturar dos tipos distintos de errores: uno por división entre cero (`ZeroDivisionError`) y otro por conversión incorrecta de tipo (`ValueError`). Si ocurre cualquier otro tipo de error, se muestra un mensaje genérico indicando el tipo de excepción.

!!!example "Ejemplo de captura de varios tipos de excepciones"

    ```python
    def procesar_datos():
        try:
            numero = int(input("Introduce un número entero: "))
            resultado = 100 / numero
            print("Resultado:", resultado)
        except ZeroDivisionError:
            print("Error: No se puede dividir entre cero.")
        except ValueError:
            print("Error: Debes introducir un número válido.")
        except Exception as e:
            print("Ha ocurrido una excepción no prevista:", type(e).__name__)

    procesar_datos()
    ```

Este ejemplo permite comprobar cómo se pueden capturar errores específicos y también cómo se puede mostrar el tipo de error si no se ha previsto. El uso de `type(e).__name__` permite mostrar el nombre de la excepción que se ha producido.

Son muchos los tipos de excepciones a captura, sin embargo, para simplificar a continuación se muestra una tabla con algunas de las excepciones más habituales que pueden encontrarse al programar en Python:

| Excepción           | Cuándo ocurre                                                                 |
|---------------------|--------------------------------------------------------------------------------|
| `ZeroDivisionError` | Al dividir un número entre cero                                                |
| `ValueError`        | Al pasar un valor incorrecto a una función (por ejemplo, `int("abc")`)         |
| `TypeError`         | Al operar con tipos incompatibles (por ejemplo, `5 + "hola"`)                  |
| `IndexError`        | Al acceder a una posición inexistente en una lista                             |
| `KeyError`          | Al acceder a una clave inexistente en un diccionario                           |
| `FileNotFoundError` | Al intentar abrir un archivo que no existe                                     |
| `ImportError`       | Al intentar importar un módulo que no se encuentra                             |
| `AttributeError`    | Al intentar acceder a un atributo que no existe en un objeto                   |
| `NameError`         | Al usar una variable que no ha sido definida                                   |
| `RuntimeError`      | Error genérico que ocurre durante la ejecución                                 |
| `Exception`         | Clase base de todas las excepciones; se puede usar para capturar cualquier tipo|


### Uso de `finally`

También se puede usar `finally` para ejecutar código siempre, ocurra o no una excepción:

!!!example "Ejemplo de uso de `finally`"

    ```python
    try:
        print("Intentando operación...")
    finally:
        print("Fin del bloque")
    ```

Por supuesto, este ejemplo puede ser mejorado con el uso de tantos `except` como queramos

### Lanzar excepciones con `raise`

Por último, veamos también un ejemplo de `raise` que nos permite lanzar una excepción manualmente:

!!!example "Uso de `raise`"

    ```python
    def validar_edad(edad):
        if edad < 0:
            raise ValueError("La edad no puede ser negativa")
    ```

En ocasiones, para la gestión de errores de nuestro aplicativo nos puede resultar conveniente el uso de esta estructura.

## 🧩 Ejemplo y ejercicio de uso

En el siguiente ejemplo se muestra cómo definir una función que realiza una operación matemática, valida la entrada del usuario, y gestiona distintos tipos de errores mediante excepciones específicas. Además, se incluye un bloque genérico para capturar cualquier otro tipo de error no previsto.

Este ejemplo permite aplicar de forma conjunta los conceptos de funciones, parámetros, retorno de valores, docstrings, y manejo de excepciones.

```python
def calcular_division():
    """Solicita dos números al usuario y devuelve el resultado de dividirlos."""
    try:
        a = int(input("Introduce el numerador: "))
        b = int(input("Introduce el denominador: "))
        resultado = a / b
        print(f"Resultado: {resultado}")
    except ZeroDivisionError:
        print("Error: No se puede dividir entre cero.")
    except ValueError:
        print("Error: Debes introducir números enteros válidos.")
    except Exception as e:
        print("Ha ocurrido un error inesperado:", type(e).__name__)

calcular_division()
```

Este ejemplo cubre tres tipos de errores:
- División entre cero (`ZeroDivisionError`)
- Conversión fallida de texto a número (`ValueError`)
- Cualquier otro error no previsto (`Exception`)

---

!!!question "Ejercicio básico: Validación de precio con función y excepciones"

    Crea una función llamada `calcular_precio_final` que reciba dos argumentos:
    
    - `precio_base`: un número que representa el precio sin impuestos.
    - `iva`: porcentaje de IVA aplicado (por defecto 21).
    
    La función debe:
    
    1. Calcular el precio final aplicando el IVA.
    2. Lanzar una excepción `ValueError` si el precio base es negativo.
    3. Lanzar una excepción `TypeError` si alguno de los valores no es numérico.
    4. Devolver el resultado con un mensaje formateado.
    
    Prueba la función con los siguientes valores:
    
    - `calcular_precio_final(100)`  
    - `calcular_precio_final(-50)`  
    - `calcular_precio_final("cien")`

    > Pistas  
    > - Usa `try`, `except` y `raise` dentro de la función.  
    > - Usa `isinstance()` para comprobar si los valores son numéricos.  
    > - Usa `return` para devolver el mensaje final.

    ???quote "Solución"

        ```python
        def calcular_precio_final(precio_base, iva=21):
            """Calcula el precio final con IVA, validando los datos de entrada."""
            try:
                if not isinstance(precio_base, (int, float)) or not isinstance(iva, (int, float)):
                    raise TypeError("Los valores deben ser numéricos.")
                if precio_base < 0:
                    raise ValueError("El precio base no puede ser negativo.")
                precio_final = precio_base * (1 + iva / 100)
                return f"Precio final: {round(precio_final, 2)} €"
            except ValueError as ve:
                return f"Error de valor: {ve}"
            except TypeError as te:
                return f"Error de tipo: {te}"
            except Exception as e:
                return f"Error inesperado: {type(e).__name__}"

        print(calcular_precio_final(100))     # Precio final: 121.0 €
        print(calcular_precio_final(-50))     # Error de valor
        print(calcular_precio_final("cien"))  # Error de tipo
        ```
