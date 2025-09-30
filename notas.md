# Notas SGE

## organización del curso

- Presentación conceptos
  - Apuntes Opos?? --> revisar. AÑADIR
  - ERP - CRM
  - Industria 4.0
  - SaaS, PaaS, IaaS

- Preparación
  - Docker
  - Git
  - Python
  - PostgresSQL

- Python








- Instalación Odoo
  - Activar módulos habituales
  - Configuración y jugar con algún módulo

- Entorno de desarrollo
  - Preparación 
    - Docker compose 
    - VS Code 
    - Git
  - Primer módulo "Hola Mundo"
  - "scaffold" módulo pruebas
    - Modelo Simple
    - Vista Simple
    - Relaciones
    - Otras complicaciones




TO-DO
-----

- No funciona el HolaMundo_V2
- Añadir al docker compose para poder conectar con la base de datos desde fuera
- Modelos....





## Enlaces a documentación interesante:

### Jose Castillo

- [Documentación en github pages](https://xxjcaxx.github.io/exemples_sge/intro.html)

### Alfredo Oltra:

Profesor de Formación a distancia de CV
Se basa en Odoodock, pero tiene cosas aprovechables.

- Github: [https://github.com/aoltra/odoodock/tree/main](https://github.com/aoltra/odoodock/tree/main)
- Github docs: [https://aoltra.github.io/odoodock/](https://aoltra.github.io/odoodock/)
- Youtube: 
  - Lista odoo: [https://www.youtube.com/watch?v=p-YVRFPVe1I&list=PL7VXpoQP1k26QVayXQ3_zbrChgn83wXyo](https://www.youtube.com/watch?v=p-YVRFPVe1I&list=PL7VXpoQP1k26QVayXQ3_zbrChgn83wXyo)
  - Lista Python: [https://www.youtube.com/watch?v=S2g3xmD9u0k&list=PL7VXpoQP1k25rBWU2jxCpxVZA1enxbye6](https://www.youtube.com/watch?v=S2g3xmD9u0k&list=PL7VXpoQP1k25rBWU2jxCpxVZA1enxbye6)




### Javier Abellan

Tiene una lista un poco desordenada, lo más interesante

- [Cración de un módulo para ampliar el módulo de contactos](https://www.youtube.com/watch?v=haGo8kZdDtg&list=PLvgBbgkfQDMQCR6Cq1OiEBE7aph1sYTuT)


### Formación Odoo
 - ODOO formacion: [Technical Training - Introducing to development](https://www.odoo.com/slides/technical-training-introduction-to-development-318)
 - [Tutorial Server framework 101](https://www.odoo.com/documentation/18.0/es/developer/tutorials/server_framework_101.html)
 - [Referencia](https://www.odoo.com/documentation/18.0/es/developer/reference.html)

### otros

 - [Documentación completa de SGE en un instituto IAX](https://www.iax.es/rea/informatica01/index.html)




### Inma Gijón

Tiene un curso de Odoo desde el principio hasta avanzado. Buena pinta los videos, el github un poco desastre

- [Lista youtube Odoo 16](https://www.youtube.com/watch?v=3zLlW25UwSI&list=PLKBxfVADNf1VBeAnACnbDVVA450s81Jpm)
- [github](https://github.com/igijon)


Pone en marche el servidor e instala los siguientes módulos:

- ventas
- Facturación 
- CRM 
- Inventario

En el fichero de configuración config/odoo.conf, define el fichero de log porque no lo tiene en el contenedor si no en una carpeta en el host por medio del docker-compose.yaml, así que hacemos

```
logfile = /var/log/odoo/odoo-server.log
```

Si hacemos esto, al reiniar el servidor en la carpeta de log, nos mostrará los logs.

En el fichero odoo.conf hay otras cosas interesantes como usuario y contraseña.
Revisando el fichero de configuración, podemos ver por ejemplo las carpetas de los addons



## Alberto Aparicio

### Python

  - https://edube.org/study/pe1
    - Tienen que hacer el curso
    - Deben entregar las capturas de los ejercicios con un print("TU NOMBRE") en la solución.
      - Módulo 2
        - 2.1.1.6 LABORATORIO: La función print()
        - 2.1.1.18 LABORATORIO: La función print()
        - 2.1.1.19 LABORATORIO: Dando formato a la salida
        - 2.2.1.11 LABORATORIO: Literales de Python - Cadenas
        - 2.4.1.7 LABORATORIO: Variables
        - 2.4.1.9 LABORATORIO: Variables, un sencillo convertidor
        - 2.4.1.10 LABORATORIO: Operadores y expresiones
        - 2.6.1.9 LABORATORIO: Entradas y salidas simples
        - 2.6.1.10 LABORATORIO: Operadores y expresiones
        - 2.6.1.11 LABORATORIO: Operadores y expresiones
        - PE1 - Modeulo 2 Quiz
        - FP1 - Examen Módulo 2
      - Módulo 3:
        - 3.1.1.10 LABORATORIO: Operadores de comparación y ejecución condicional
        - 3.1.1.11 LABORATORIO: Fundamentos de la sentencia if-else
        - 3.1.1.12 LABORATORIO: Fundamentos de la sentencia if-elif-else
        - 3.2.1.3 LABORATORIO: Lo esencial del bucle while - Adivina el número secreto
        - 3.2.1.6 LABORATORIO: Fundamentos del bucle for: el conteo
        - 3.2.1.9 LABORATORIO: La sentencia break - Atascado en un bucle
        - 3.2.1.10 LABORATORIO: La sentencia continue - El Feo Devorador de Vocales
        - 3.2.1.11 LABORATORIO: La sentencia continue - El Bonito Devorador de Vocales
        - 3.2.1.14 LABORATORIO: Fundamentos del bucle while
        - 3.2.1.15 LABORATORIO: Hipótesis de Collatz
        - 3.4.1.6 LABORATORIO: Lo básico de las listas
        - 3.4.1.13 LABORATORIO: Lo básico de las listas - Los Beatles
        - 3.6.1.9 LABORATORIO: Operando con listas - conceptos básicos
      - Módulo 4: Funciones
        - 4.3.1.6 LABORATORIO: Un año bisiesto: escribiendo tus propias funciones
        - 4.3.1.7 LABORATORIO: ¿Cuántos días?: escribiendo y utilizando tus propias funciones
        - 4.3.1.8 LABORATORIO: Día del año: escribiendo y utilizando tus propias funciones
        - 4.3.1.9 LABORATORIO: Números primos: ¿Cómo encontrarlos?
        - 4.3.1.10 LAB: Convirtiendo el consumo de combustible




- https://edube.org/study/pe1
- https://edube.org/study/pe2



Para que no funcione el autocompletado en VSCode:

Configuración de proyecto para Visual Studio Code que **desactiva todas las ayudas automáticas**, **impide sugerencias de extensiones**, y **usa un color de tema distintivo** para que los alumnos vean claramente que están en un entorno controlado. Además, puedes combinar esto con una actividad reflexiva sobre cómo el entorno afecta su forma de programar.

---

### 📁 Estructura del proyecto

```
mi_proyecto/
├── .vscode/
│   ├── settings.json
│   └── extensions.json
├── main.py
```

---

### 🧼 `.vscode/settings.json` — sin ayudas, con tema visual claro

```json
{
  "workbench.colorTheme": "Monokai", 
  "editor.quickSuggestions": {
    "other": false,
    "comments": false,
    "strings": false
  },
  "editor.suggestOnTriggerCharacters": false,
  "editor.parameterHints.enabled": false,
  "editor.hover.enabled": false,
  "editor.wordBasedSuggestions": false,
  "editor.inlineSuggest.enabled": false,
  "editor.lightbulb.enabled": false,
  "editor.codeActionsOnSave": {},
  "python.languageServer": "None",
  "python.analysis.autoImportCompletions": false,
  "python.analysis.typeCheckingMode": "off",
  "files.autoSave": "onFocusChange"
  "telemetry.enableTelemetry": false,
  "telemetry.enableCrashReporter": false
}
```

🔸 **Tema Monokai**: sirve como señal visual clara de que están en el entorno “sin ayudas”. Puedes elegir otro tema distintivo si lo prefieres.

---

### 🚫 `.vscode/extensions.json` — sin recomendaciones

```json
{
  "recommendations": [],
  "unwantedRecommendations": [
    "ms-python.python",
    "ms-python.vscode-pylance",
    "ms-vscode.intellicode",
    "ms-toolsai.jupyter"
  ]
}
```

Esto evita que VSCode recomiende extensiones que podrían reactivar ayudas como autocompletado, notebooks o sugerencias inteligentes.
