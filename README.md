# Laboratorio 3 — Aplicación Inteligente Adaptativa

## 1. Identificación

- **Integrantes:** Javier Bórquez Diaz, Benjamín Peredo Durán y Diego Valenzuela Astudillo
- **Asignatura:** Ingeniería Web Avanzada
- **Curso y sección:** OII436-1
- **URL del repositorio:** https://github.com/JavierBor/Laboratorio3.git
- **Fecha:** 09-11-2026

---

## 2. Objetivo y alcance

El objetivo del laboratorio es desarrollar un sistema inteligente de recomendaciones de recursos de aprendizaje utilizando Python.

La aplicación recomienda contenidos según los intereses de cada usuario y permite actualizar automáticamente su perfil cuando selecciona la opción **"Me interesa"**.

Para realizar las recomendaciones se utilizan las siguientes técnicas:

- **TF-IDF:** transforma los intereses y las etiquetas de los recursos en vectores numéricos.
- **Similitud del coseno:** permite calcular qué tan similares son los recursos respecto al perfil del usuario.
- **Actualización adaptativa:** modifica el perfil según las interacciones realizadas.

El sistema trabaja con un catálogo de 12 recursos y cuatro usuarios de prueba. Toda la implementación se realiza en Python mediante una interfaz de consola.

---

## 3. Preparación del entorno

### Tecnologías utilizadas

- Python 3.11.9
- Pandas 2.2.3
- NumPy
- Scikit-learn
- Git y GitHub

### Crear el entorno virtual

En Windows:

```powershell
py -3.11 -m venv .venv
```

### Activar el entorno virtual

En PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

En CMD:

```bat
.venv\Scripts\activate.bat
```

En macOS o Linux:

```bash
source .venv/bin/activate
```

### Instalar las dependencias

```bash
python -m pip install -r requirements.txt
```

### Comprobar la preparación

```bash
python app.py --comprobar
```

Resultado obtenido:

```text
Comprobación de preparación completada.
Recursos: 12 | Usuarios: 4
Matriz TF-IDF: 12 filas x 17 términos
```

El sistema cargó correctamente los recursos, usuarios y vocabulario.

### Problema encontrado durante la instalación

Inicialmente se utilizó Python 3.14.3 con Pandas 3.1.1, pero Windows bloqueaba la carga de algunos componentes internos de Pandas mediante una directiva de Control de aplicaciones.

Para solucionarlo se instaló Python 3.11.9 y Pandas 2.2.3.

Después del cambio, las dependencias pudieron utilizarse correctamente y el laboratorio se ejecutó sin errores.

---

## 4. Catálogo y perfiles de usuario

Los recursos se encuentran almacenados en:

`datos/recursos.csv`

Cada recurso contiene los siguientes campos:

| Campo | Descripción |
|---|---|
| id | Identificador único del recurso |
| titulo | Nombre del recurso |
| descripcion | Información descriptiva |
| etiquetas | Palabras utilizadas para calcular la similitud |

Para calcular las recomendaciones se utiliza únicamente la columna `etiquetas`. El título y la descripción se utilizan para presentar la información.

Los perfiles iniciales están almacenados en:

`datos/perfiles.json`

Los usuarios disponibles son:

- **u1:** intereses en Python, datos y Pandas.
- **u2:** intereses en Angular, componentes e interfaces.
- **u3:** usuario sin intereses declarados.
- **u4:** usuario con términos fuera del vocabulario.

La función `cargar_catalogo()` comprueba que las columnas obligatorias existan, que sus valores no estén vacíos y que no existan identificadores de recursos duplicados. Si encuentra información inválida, genera un error.

La función `cargar_perfiles()` permite intereses vacíos para comprobar el funcionamiento del sistema en esos casos.

Los perfiles se mantienen independientes y sus modificaciones se conservan en memoria durante la ejecución.

---

## 5. Funciones implementadas

Se completaron las cuatro funciones solicitadas en `motor.py`.

| Función | Descripción |
|---|---|
| `crear_perfil()` | Convierte los intereses de un usuario en un vector TF-IDF utilizando el vocabulario del catálogo. |
| `calcular_puntuaciones()` | Calcula la similitud del coseno entre el perfil y los 12 recursos. |
| `recomendar()` | Selecciona hasta tres recursos con puntuación positiva, ordenados de mayor a menor y con desempate por ID ascendente. |
| `actualizar_perfil()` | Actualiza y normaliza el perfil cuando el usuario selecciona "Me interesa". |

### Actualización del perfil

Para actualizar el perfil se utiliza la siguiente fórmula:

```python
mezcla = 0.4 * perfil + 0.6 * vector_recurso
perfil_actualizado = normalize([mezcla])[0]
```

El 40 % corresponde al vector anterior y el 60 % al vector del recurso seleccionado.

Esta combinación permite incorporar nuevos intereses sin eliminar completamente los anteriores.

El resultado se normaliza para mantener una representación adecuada del perfil.

---

## 6. Ejecución de la aplicación

### Modo interactivo

```bash
python app.py
```

Este modo permite seleccionar un usuario e interactuar mediante las siguientes opciones:

| Opción | Función |
|---|---|
| r | Consultar recomendaciones |
| c | Consultar el catálogo completo |
| m | Seleccionar "Me interesa" |
| u | Cambiar de usuario |
| s | Salir de la aplicación |

Los cambios realizados en un usuario se conservan durante la ejecución, sin modificar los perfiles de los demás usuarios.

Al reiniciar la aplicación se recuperan los perfiles iniciales.

### Modo demostración

```bash
python app.py --demo
```

Este comando ejecuta automáticamente las pruebas T1, T2, T3 y T4 y genera los archivos de evidencia.

---

## 7. Pruebas funcionales

Se ejecutaron las cuatro pruebas solicitadas utilizando `python app.py --demo` y se verificó el funcionamiento de la aplicación mediante la consola interactiva.

| Caso | Entrada o acción | Resultado esperado | Resultado observado | Estado |
|---|---|---|---|---|
| T1 | Solicitar recomendaciones para u1 y u2. | Obtener hasta tres recomendaciones según los intereses de cada usuario. | u1 obtuvo R02, R12 y R03. u2 obtuvo R07, R08 y R09. | Cumple |
| T2 | u1 selecciona "Me interesa" en R04. | Actualizar el perfil y recalcular las puntuaciones, manteniendo los 12 recursos. | El perfil incorporó `api` y `web`. R04 pasó al primer lugar y R05 aumentó su puntuación. | Cumple |
| T3 | Consultar u2 después de modificar u1. | Mantener el perfil, historial y puntuaciones de u2 sin cambios. | u2 conservó sus intereses, puntuaciones y recomendaciones originales. | Cumple |
| T4 | Solicitar recomendaciones para u3 y u4. | Obtener listas vacías y mensajes informativos. | Ambos usuarios recibieron listas vacías sin provocar errores. | Cumple |

### Interpretación de las pruebas

**T1:** El sistema utiliza los intereses de cada usuario para calcular recomendaciones diferentes, incluso cuando ambos consultan el mismo catálogo.

**T2:** La interacción de u1 con R04 modificó automáticamente su perfil. Esto produjo cambios en las puntuaciones de distintos recursos, demostrando que existe adaptación.

**T3:** Los perfiles son independientes. La modificación realizada sobre u1 no afectó las recomendaciones ni el historial de u2.

**T4:** Los perfiles vacíos o con términos desconocidos producen puntuaciones cero y listas vacías, evitando recomendar recursos sin coincidencias.

---

## 8. Evidencias de recomendaciones iniciales

### Usuario u1

Intereses iniciales: `python datos pandas`.

| Posición | Recurso | Puntuación | Explicación |
|---|---|---|---|
| 1 | R02 — Análisis de datos con Pandas | 1.0000 | Coincide con datos, pandas y python. |
| 2 | R12 — Conectar Python con datos SQL | 0.4965 | Coincide con datos y python. |
| 3 | R03 — Gráficos con Python | 0.4326 | Coincide con datos y python. |

### Usuario u2

Intereses iniciales: `angular componentes interfaces`.

| Posición | Recurso | Puntuación | Explicación |
|---|---|---|---|
| 1 | R07 — Componentes e interfaces en Angular | 1.0000 | Coincide con angular, componentes e interfaces. |
| 2 | R08 — Formularios e interfaces en Angular | 0.5677 | Coincide con angular e interfaces. |
| 3 | R09 — Consumir una API desde Angular | 0.3087 | Coincide con angular. |

Los rankings demuestran que cada usuario recibe recomendaciones relacionadas con sus intereses.

Las puntuaciones representan similitudes entre vectores, no probabilidades de que el recurso guste.

---

## 9. Comparación de la adaptación

### Interacción realizada

- **Usuario:** u1
- **Acción:** Me interesa
- **Recurso seleccionado:** R04 — API web con Python
- **Etiquetas del recurso:** python web api

### Perfil antes de la interacción

```python
{
    'pandas': 0.753247,
    'datos': 0.465091,
    'python': 0.465091
}
```

### Perfil después de la interacción

```python
{
    'python': 0.637116,
    'api': 0.446269,
    'web': 0.446269,
    'pandas': 0.376477,
    'datos': 0.232455
}
```

Se observa que el perfil incorporó los términos `api` y `web`, mientras que los intereses anteriores se conservaron con pesos actualizados.

### Historial de la interacción

Antes de la valoración:

```python
[]
```

Después de la valoración:

```python
[
    {
        'numero': 1,
        'accion': 'Me interesa',
        'recurso': 'R04'
    }
]
```

### Comparación de las puntuaciones de los 12 recursos

| Recurso | Antes | Después | Diferencia |
|---|---:|---:|---:|
| R01 | 0.1861 | 0.2549 | +0.0688 |
| R02 | 1.0000 | 0.6880 | -0.3120 |
| R03 | 0.4326 | 0.4044 | -0.0282 |
| R04 | 0.2510 | 0.8752 | +0.6241 |
| R05 | 0.0000 | 0.4378 | +0.4378 |
| R06 | 0.0000 | 0.4378 | +0.4378 |
| R07 | 0.0000 | 0.0000 | 0.0000 |
| R08 | 0.0000 | 0.0000 | 0.0000 |
| R09 | 0.0000 | 0.4958 | +0.4958 |
| R10 | 0.2053 | 0.1026 | -0.1027 |
| R11 | 0.2053 | 0.1026 | -0.1027 |
| R12 | 0.4965 | 0.4642 | -0.0324 |

### Interpretación del cambio

El recurso R04 aumentó su similitud de 0.2510 a 0.8752 porque el usuario lo seleccionó mediante la interacción "Me interesa".

Un resultado importante es el del recurso **R05 — Principios de una API HTTP**, que aumentó su puntuación de 0.0000 a 0.4378 sin haber sido valorado directamente.

Esto ocurrió porque R05 comparte los términos `web` y `api` con R04, características que fueron incorporadas al perfil del usuario.

También se observa que R09 aumentó de 0.0000 a 0.4958, pasando a formar parte de las tres principales recomendaciones.

Por otro lado, R02 disminuyó su similitud de 1.0000 a 0.6880, debido a que el perfil ahora también representa intereses relacionados con API web.

**Conclusión:** el sistema demuestra un comportamiento adaptativo porque una interacción modifica automáticamente el perfil del usuario y produce cambios en las puntuaciones de distintos recursos, manteniendo los mismos 12 candidatos.

---

## 10. Independencia de los usuarios

Después de actualizar el perfil de u1, se verificó que u2 mantuviera su estado original.

Perfil de u2 antes y después:

```python
{
    'componentes': 0.657512,
    'interfaces': 0.564679,
    'angular': 0.498813
}
```

Sus recomendaciones permanecieron iguales:

| Recurso | Antes | Después |
|---|---:|---:|
| R07 | 1.0000 | 1.0000 |
| R08 | 0.5677 | 0.5677 |
| R09 | 0.3087 | 0.3087 |

Su historial también permaneció vacío.

Esto confirma que el sistema conserva perfiles independientes para cada usuario.

---

## 11. Usuarios sin intereses reconocidos

Se probaron los siguientes casos:

**Usuario u3:** no tiene intereses declarados.

**Usuario u4:** tiene el término desconocido `zzxyz123`.

Ambos recibieron el siguiente mensaje:

```text
No hay coincidencias positivas. Declare intereses presentes en el catálogo.
```

En ambos casos se obtuvo una lista vacía de recomendaciones, sin generar errores ni asignar puntuaciones positivas a recursos sin coincidencias.

---

## 12. Preguntas evaluadas

### Pregunta 1

**¿Qué información describe el contenido de los recursos y qué información constituye el perfil del usuario? Identifique ambas en los archivos del proyecto.**

**Respuesta:**

El contenido de los recursos se encuentra en `datos/recursos.csv`, donde cada recurso tiene un identificador, título, descripción y etiquetas. Para calcular la similitud se utiliza únicamente la columna de etiquetas.

El perfil del usuario se obtiene de `datos/perfiles.json`, que contiene su nombre e intereses iniciales. Estos intereses se convierten en un vector TF-IDF y se mantienen como parte del estado del usuario, junto con su historial de interacciones.

### Pregunta 2

**¿Por qué se utiliza fit_transform sobre el catálogo y transform sobre los intereses del usuario? ¿Qué problema causaría ajustar un vectorizador diferente para cada perfil?**

**Respuesta:**

Se utiliza fit_transform sobre el catálogo para que el modelo "aprenda" el vocabulario completo y calcule los pesos bases, construyendo el espacio vectorial de referencia. Luego, se usa transform sobre los intereses del usuario para proyectar sobre los intereses del usuario para proyectar sus términos exactamente en ese mismo espacio vectorial ya aprendido. Usar un vectorizador diferente haría que los vectores resultantes tuvieran tendrían dimensiones y columnas distintas, haciendo matemáticamente imoposibel compararlos con la matriz del catálogo mediante la similitud del coseno.

### Pregunta 3

**¿Por qué el mismo catálogo puede producir recomendaciones distintas para u1 y u2? ¿Una similitud de 0,80 significa que existe un 80 % de probabilidad de que el recurso guste? Justifique.**

**Respuesta:**

El mismo catalogo produce recomendaciones distintas porque los usuarios parten con intereses diferentes, por ejemplo, u1 "python datos panda" y u2 busca "angular componentes interfaces", lo que posiciona sus vectores en direcciones distintas dentro del espacio vectorial. 

No, no significa un 80% de probabilidad de que el recurso guste. La similitud del coseno simplemente mide que tan alineadas están las direcciones de dos vectores basándose en los términos que comparten. Es una medida geométrica de coincidencia de características, no una probabilidad estadística ni una medida de comprensión semántica.

### Pregunta 4

**¿En qué se diferencia modificar manualmente los intereses iniciales de la adaptación implementada? Identifique el evento que provoca el cambio, el estado modificado y su efecto observable.**

**Respuesta:**

Modificar manualmente los intereses cambia estáticamente el punto de partida del usuario. En cambio, la adaptación implementada es dinámica y ocurre en tiempo de ejecución.

El evento: Es la interacción del usuario al marcar [Me interesa] en un recurso.
El estado modificado: El vector actual del perfil del usuario y su historial
El efecto observable: Las puntuaciones de similitud de todos los recursos se recalculan en la misma sesión, alterando el ranking y subiendo la prioridad de recursos similares al valorado, incluso sin una participación directa.

Por ejemplo, cuando u1 seleccionó R04, su perfil incorporó los términos `api` y `web`, haciendo que R04 pasara al primer lugar y aumentando también las puntuaciones de R05 y R09.

### Pregunta 5

**Relacione su implementación con las etapas del ciclo MAPE-K estudiado en clase: Monitorizar, Analizar, Planificar, Ejecutar y Conocimiento. Indique una función, operación o dato de su programa para cada etapa.**

**Respuesta:**

El sistema implementado puede relacionarse con las etapas del ciclo MAPE-K de la siguiente manera:

** Monitorizar: ** La recepción del evento de retroalimentación
** Analizar: ** La función calcular_puntuaciones, que compara el perfil actual contra el catálogo utilizando la similitud del coseno.
** Planificar y Ejecutar: ** La función actualizar_perfil, que calcula la mezcla ponderada de los vectores y normaliza el resultado para crear un estado nuevo.
** Conocimiento: ** La matriz del catálogo generada por TF-IDF y el estado en memoria de cada perfil de usuario.

De esta manera, el sistema recibe información de las interacciones, analiza los intereses, selecciona recomendaciones y actualiza su estado.

### Pregunta 6

**Indique una limitación del mecanismo implementado y proponga una mejora concreta. Explique qué dato adicional o qué cambio del algoritmo exigiría esa mejora.**

**Respuesta:**

Una limitación importante del mecanismo implementado es que TF-IDF se basa en coincidencias exactas de vocabulario y no comprende de la semántica. Si el usuario tiene interés en "Inteligencia artificial" y el recurso está etiquetado como "machine learning", la similitud será cero porque las palabras no coinciden con exactitud.

Una mejora concreta sería reemplazar TF-IDF por modelos de Word Embeddings (como Wotd2Vec, BERT o embeddings de OpenAI). Esto exigiría un cambio algorítmico, utilizar un modelo preentrenado que convierta textos en vectores densos, permitiendo que el sistema conecte conceptos sinónimos o semánticamente relacionados aunque no compartan la misma palabra exacta.

---

## 13. Archivos de evidencia

La ejecución del modo demostración generó los siguientes archivos:

- `evidencias/reporte_demo.json`: contiene los perfiles, historiales, recomendaciones y resultados de las cuatro pruebas.
- `evidencias/puntuaciones_antes_despues.csv`: registra las puntuaciones de los 12 recursos antes y después de la valoración de R04, incluyendo las diferencias.

Los archivos contienen los resultados reales obtenidos durante la ejecución.

---

## 14. Archivos de entrega

El repositorio contiene los siguientes archivos:

```text
laboratorio3/
├── app.py
├── motor.py
├── datos/
│   ├── recursos.csv
│   └── perfiles.json
├── evidencias/
│   ├── puntuaciones_antes_despues.csv
│   └── reporte_demo.json
├── requirements.txt
├── .gitignore
└── README.md
```

No se incluyen el entorno virtual `.venv`, los archivos de caché de Python ni dependencias instaladas.

---

## 15. Reproducción del laboratorio

Para ejecutar el proyecto desde una copia nueva del repositorio:

1. Clonar el repositorio de GitHub.
2. Abrir la carpeta del laboratorio en Visual Studio Code.
3. Crear un entorno virtual con Python 3.11.
4. Activar el entorno virtual.
5. Instalar las dependencias desde `requirements.txt`.
6. Ejecutar la comprobación inicial.
7. Ejecutar la demostración o el modo interactivo.

Comandos principales en PowerShell:

```powershell
py -3.11 -m venv .venv

.\.venv\Scripts\Activate.ps1

python -m pip install -r requirements.txt

python app.py --comprobar

python app.py --demo

python app.py
```

Las pruebas T1, T2, T3 y T4 se completaron correctamente en el entorno utilizado durante el desarrollo.

---

## 16. Conclusión

El laboratorio permitió implementar un sistema de recomendaciones adaptativo utilizando Python, TF-IDF y similitud del coseno.

Se logró representar los intereses de los usuarios, calcular recomendaciones según sus perfiles y actualizar automáticamente sus preferencias mediante la interacción "Me interesa".

Las pruebas realizadas demostraron que las recomendaciones cambian de acuerdo con las interacciones del usuario, que los perfiles se mantienen independientes y que el sistema maneja correctamente los casos sin intereses reconocidos.

De esta manera, se cumplieron los objetivos funcionales establecidos para el laboratorio.z
### Reproducción desde una copia nueva

Compruebe que otra persona puede crear el entorno, instalar las dependencias y
ejecutar los comandos anteriores utilizando únicamente los archivos entregados.
Registre aquí cualquier instrucción adicional necesaria: [Completar].
