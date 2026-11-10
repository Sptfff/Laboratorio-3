# Laboratorio de aplicación inteligente adaptativa

## Identificación

- Integrantes: Matias Romero Alvarez
- Curso y sección: Ingeniería Web Avanzada — OII436-1
- URL del repositorio: https://github.com/Sptfff/Laboratorio-3
- Fecha: 2026-11-09

## Objetivo y alcance

Recomendador de recursos de aprendizaje basado en contenido. Representa los
recursos y los intereses de cada usuario con TF-IDF, los compara con similitud del
coseno y actualiza el perfil del usuario activo cuando indica **Me interesa** sobre
un recurso. El núcleo se ejecuta en Python por consola.

Solo se completaron los cuatro bloques de `motor.py` (`crear_perfil`,
`calcular_puntuaciones`, `recomendar` y `actualizar_perfil`). `app.py`, los datos
y las validaciones se mantienen tal como fueron entregados.

La representación usa **solo la columna `etiquetas`**. Títulos y descripciones sirven
para presentar el catálogo. Todos los recursos continúan disponibles después de
una interacción; no se excluye el recurso valorado, de modo que se comparan los
mismos 12 candidatos antes y después de la actualización del perfil.

## Preparación

Versión de Python utilizada: **Python 3.12.10** (Windows 11, 64 bits).
Comando de activación utilizado: **`.venv\Scripts\Activate.ps1`** (PowerShell).

```bash
python -m venv .venv
```

En Windows puede usar `py -m venv .venv`.

Activación en PowerShell (utilizada en este laboratorio):

```powershell
.venv\Scripts\Activate.ps1
```

Si PowerShell bloquea el script por la política de ejecución, ejecutar una vez
`Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` y volver a activar.

Activación en CMD:

```bat
.venv\Scripts\activate.bat
```

Activación en macOS o Linux:

```bash
source .venv/bin/activate
```

Instalación y comprobación (con el entorno activo, el prompt muestra `(.venv)`):

```bash
python -m pip install -r requirements.txt
python -c "import sys; print(sys.executable)"
python app.py --comprobar
```

`sys.executable` debe apuntar a `.venv\Scripts\python.exe`. La comprobación informa
12 recursos, 4 usuarios y una matriz TF-IDF de 12 filas x 17 términos.
`requirements.txt` se generó con `python -m pip freeze > requirements.txt` e incluye
las versiones exactas instaladas (numpy 2.5.4, pandas 3.1.1, scikit-learn 1.9.1).

`.gitignore` excluye `.venv/`, `__pycache__/`, `*.py[cod]`, `.DS_Store` y `node_modules/`.

## Catálogo y datos

`python app.py --comprobar` confirma 12 recursos (R01 a R12) con las columnas
`id`, `titulo`, `descripcion` y `etiquetas`, y un vocabulario de 17 términos:
angular, api, componentes, consultas, datos, formularios, funciones, graficos, http,
interfaces, pandas, python, sql, tablas, validacion, variables, web.

- **Campos que intervienen en el cálculo:** solo `etiquetas` se vectoriza con
  TF-IDF. `id` identifica la fila (y desempata el ranking); `titulo` y `descripcion`
  solo se muestran.
- **Valores vacíos:** `cargar_catalogo` lee todo como texto (`keep_default_na=False`),
  elimina espacios y rechaza el catálogo con un `ValueError` si alguna de las cuatro
  columnas queda vacía. En perfiles, un `intereses` vacío es válido (caso u3) y produce
  un vector nulo.
- **Identificadores repetidos:** `cargar_catalogo` rechaza el archivo si `id` tiene
  duplicados, para que cada interacción afecte a un único recurso.

## Implementación de `motor.py`

| Función | Implementación | Decisión |
|---|---|---|
| `crear_perfil` | `vectorizador.transform([intereses]).toarray()[0]` | Se usa `transform` para proyectar los intereses en el vocabulario ya ajustado al catálogo; no se vuelve a ajustar. Términos desconocidos se ignoran y dan un vector nulo. |
| `calcular_puntuaciones` | `cosine_similarity([perfil], matriz)[0]` | `[perfil]` convierte el vector 1D en matriz de una fila; `[0]` devuelve una puntuación por recurso en el orden del CSV. Con un perfil nulo scikit-learn devuelve ceros. |
| `recomendar` | Filtra puntuaciones `> 0`, ordena por `(-puntuación, id)` y toma `top_k` | No se completan posiciones con ceros. Las puntuaciones se ordenan sin redondear; solo se redondean al mostrarse. |
| `actualizar_perfil` | `normalize([0.4 * perfil + 0.6 * recurso])[0]` | Mezcla el vector actual con el del recurso valorado (alpha = 0.6, valor didáctico) y normaliza a norma L2 = 1. |

## Ejecución

Modo interactivo:

```bash
python app.py
```

Demostración reproducible:

```bash
python app.py --demo
```

Genera `evidencias/reporte_demo.json` (perfiles, historiales, recomendaciones y
comprobaciones) y `evidencias/puntuaciones_antes_despues.csv` (los 12 recursos con
puntuación antes, después y diferencia).

Salida observada de `python app.py --demo`:

```text
T1: Cumple | Dos perfiles iniciales sobre el mismo catálogo
T2: Cumple | u1 indica Me interesa sobre R04
T3: Cumple | La interacción de u1 no modifica a u2
T4: Cumple | Perfil vacío y términos fuera del vocabulario
```

## Pruebas funcionales y evidencias

| Caso | Entrada o acción | Resultado esperado | Resultado observado y evidencia | Estado |
|---|---|---|---|---|
| T1 | Recomendar para u1 (`python datos pandas`) y u2 (`angular componentes interfaces`) sin interacciones | R02 primero para u1; R07 primero para u2; orden descendente, máximo 3 y puntuaciones positivas. | u1: R02 (1.0000), R12 (0.4965), R03 (0.4326). u2: R07 (1.0000), R08 (0.5677), R09 (0.3087). Tres resultados positivos y descendentes en ambos. | Cumple |
| T2 | u1 indica Me interesa sobre R04 y se recalcula | Cambia el perfil; R04 pasa al primer lugar; aumenta la puntuación de R05; se conservan los 12 candidatos. | El perfil incorpora `api` y `web`. Nuevo ranking: R04 (0.8752), R02 (0.6880), R09 (0.4958). R05 sube de 0.0000 a 0.4378. El CSV contiene los mismos 12 recursos. Historial de u1: `#1 Me interesa R04`. | Cumple |
| T3 | Consultar u2 después de la interacción de u1 | El perfil, el historial y las puntuaciones de u2 se mantienen. | Perfil de u2 idéntico antes y después (componentes 0.657512, interfaces 0.564679, angular 0.498813), historial vacío y mismo ranking R07, R08, R09. | Cumple |
| T4 | Recomendar para u3 (intereses vacíos) y u4 (`zzxyz123`) | Cero puntuaciones positivas, lista vacía y mensaje informativo para ambos. | Ambos producen un vector nulo, lista `[]` y el mensaje «No hay coincidencias positivas. Declare intereses presentes en el catálogo.» La aplicación no falla. | Cumple |

**Interpretación:**

- **T1:** el mismo catálogo da rankings distintos porque cada perfil activa términos
  diferentes del vocabulario. En el top 3 inicial no hay empates; el desempate por
  `id` se verificó con el ranking completo de u1 tras la valoración, donde R05 y R06
  (ambos 0.4378) y R10 y R11 (ambos 0.1026) quedan ordenados por `id` ascendente.
- **T2:** el cambio proviene de la actualización del vector, no de una lista fija:
  recursos que no fueron valorados (R05, R06, R09) pasan de 0 a valores positivos.
- **T3:** cada usuario tiene su propio vector e historial en memoria; actualizar u1
  no modifica a u2.
- **T4:** sin términos reconocidos, todas las similitudes son 0 y `recomendar` no
  rellena posiciones con recursos sin coincidencia.

### Comparación de la adaptación

**Historial de u1:** antes, vacío; después, `[{numero: 1, accion: "Me interesa", recurso: "R04"}]`.

**Perfil de u1 (pesos no nulos):**

| Término | Antes | Después |
|---|---|---|
| pandas | 0.753247 | 0.376477 |
| datos | 0.465091 | 0.232455 |
| python | 0.465091 | 0.637116 |
| api | — | 0.446269 |
| web | — | 0.446269 |

**Recomendaciones de u1:**

| Posición | Antes | Después |
|---|---|---|
| 1 | R02 Análisis de datos con Pandas (1.0000) | R04 API web con Python (0.8752) |
| 2 | R12 Conectar Python con datos SQL (0.4965) | R02 Análisis de datos con Pandas (0.6880) |
| 3 | R03 Gráficos con Python (0.4326) | R09 Consumir una API desde Angular (0.4958) |

**Puntuaciones de los 12 candidatos** (`evidencias/puntuaciones_antes_despues.csv`, redondeadas a 4 decimales):

| id | Etiquetas | Antes | Después | Diferencia |
|---|---|---|---|---|
| R01 | python variables funciones | 0.1861 | 0.2549 | +0.0688 |
| R02 | python datos pandas | 1.0000 | 0.6880 | -0.3120 |
| R03 | python datos graficos | 0.4326 | 0.4044 | -0.0282 |
| R04 | python web api | 0.2510 | 0.8752 | +0.6241 |
| **R05** | **web api http** | **0.0000** | **0.4378** | **+0.4378** |
| R06 | web api validacion | 0.0000 | 0.4378 | +0.4378 |
| R07 | angular componentes interfaces | 0.0000 | 0.0000 | 0.0000 |
| R08 | angular formularios interfaces | 0.0000 | 0.0000 | 0.0000 |
| **R09** | **angular web api** | **0.0000** | **0.4958** | **+0.4958** |
| R10 | sql datos consultas | 0.2053 | 0.1026 | -0.1027 |
| R11 | sql datos tablas | 0.2053 | 0.1026 | -0.1027 |
| R12 | python sql datos | 0.4965 | 0.4642 | -0.0324 |

- **Perfil de u1 antes de valorar R04:** `pandas 0.753247, datos 0.465091, python 0.465091`.
- **Perfil de u1 después de valorar R04:** `python 0.637116, api 0.446269, web 0.446269, pandas 0.376477, datos 0.232455`.
- **Puntuación de R05 antes y después:** 0.0000 → 0.4378 (R09: 0.0000 → 0.4958).
- **Por qué ese cambio es evidencia de adaptación:** R05 (`web api http`) no fue
  valorado y sus etiquetas no comparten ningún término con los intereses iniciales de
  u1, por eso su similitud inicial era exactamente 0. Al indicar Me interesa en R04
  (`python web api`), `actualizar_perfil` mezcló el 60 % del vector de R04 en el perfil,
  que incorporó los términos `api` y `web` con peso 0.446269. R05 comparte esos dos
  términos, por lo que su coseno pasa a 0.4378. Lo mismo ocurre con R09 (`angular web
  api`), que además entra al top 3 desplazando a R12 y R03. A la inversa, los recursos
  que dependen de `datos` y `pandas` (R02, R10, R11) bajan porque esos pesos se redujeron
  a la mitad tras la mezcla y la normalización. Los 12 candidatos son los mismos; solo
  cambió el vector del perfil.
- **Problemas detectados y acciones realizadas:**
  - Al ejecutar `python app.py --demo` sin activar el entorno se obtuvo
    `ModuleNotFoundError: No module named 'sklearn'`, porque se usó el intérprete global.
    Se resolvió activando `.venv` y verificando `sys.executable`.
  - La similitud de R02 para u1 aparece como `1.0000000000000002` en las evidencias.
    Es un error de redondeo de punto flotante (vectores idénticos), no un valor mayor
    que 1; al mostrarse con cuatro decimales es 1.0000.
  - No se detectaron casos que no cumplan.

## Preguntas evaluadas

1. **¿Qué información describe el contenido de los recursos y qué información constituye el perfil del usuario?**

   El contenido de cada recurso está descrito por la columna `etiquetas` de
   `datos/recursos.csv` (por ejemplo R04: `python web api`); `construir_indice` la
   convierte en la matriz TF-IDF (12 x 17). `titulo` y `descripcion` solo se muestran.
   El perfil del usuario parte del campo `intereses` de `datos/perfiles.json` (u1:
   `python datos pandas`), que `crear_perfil` transforma en un vector en el mismo
   espacio. Durante la ejecución, el perfil se compone de los intereses iniciales, el
   vector actual (que se modifica con cada Me interesa) y el historial de interacciones,
   mantenidos en memoria por `app.py` para cada usuario por separado.

2. **¿Por qué se utiliza `fit_transform` sobre el catálogo y `transform` sobre los intereses? ¿Qué problema causaría ajustar un vectorizador diferente por perfil?**

   `fit_transform` aprende el vocabulario y los pesos IDF a partir del catálogo y
   representa los recursos en ese espacio. `transform` reutiliza exactamente ese
   vocabulario e IDF para representar los intereses, de modo que el término `python`
   ocupa la misma columna en el perfil y en los recursos y la similitud del coseno
   compara dimensiones equivalentes. Si se ajustara un vectorizador por perfil, cada
   perfil tendría su propio vocabulario (u1 tendría solo 3 columnas) y pesos IDF
   calculados sobre un único texto: los vectores tendrían dimensiones y significados
   distintos a los de la matriz, por lo que la comparación sería incorrecta o
   directamente imposible (error de dimensiones).

3. **¿Por qué el mismo catálogo produce recomendaciones distintas para u1 y u2? ¿Una similitud de 0,80 significa un 80 % de probabilidad de que el recurso guste?**

   Porque el ranking depende del vector de perfil: u1 tiene peso en `python`, `datos` y
   `pandas`, y u2 en `angular`, `componentes` e `interfaces`. El coseno es alto para los
   recursos que comparten esos términos (R02 para u1, R07 para u2) y 0 para los que no
   comparten ninguno. No: 0,80 indica que los vectores TF-IDF apuntan en direcciones
   parecidas, es decir, que comparten términos con pesos similares. No es una
   probabilidad calibrada, no se aprendió de datos de preferencias reales y no mide
   comprensión semántica (por ejemplo, `graficos` y `visualizacion` serían términos
   sin relación para el modelo).

4. **¿En qué se diferencia modificar manualmente los intereses iniciales de la adaptación implementada?**

   Modificar manualmente los intereses cambia el dato de partida (`perfiles.json`) y
   requiere una acción explícita del usuario o del desarrollador; el sistema no
   aprende nada. En la adaptación implementada, el **evento** es la acción
   «Me interesa» sobre un recurso (u1 sobre R04), registrada en el historial de u1. El
   **estado modificado** es el vector actual del perfil de u1 (no sus intereses
   iniciales ni el perfil de otros usuarios), que `actualizar_perfil` recalcula
   automáticamente como `normalize(0.4 * perfil + 0.6 * R04)`. El **efecto observable**
   es el recálculo inmediato de las puntuaciones en la misma ejecución: aparecen los
   términos `api` y `web` en el perfil, R05 pasa de 0 a 0.4378, R09 de 0 a 0.4958 y el
   ranking cambia a R04, R02, R09.

5. **Relación con el ciclo MAPE-K.**

   | Etapa | Elemento del programa |
   |---|---|
   | Monitorizar | `app.py` captura la acción «Me interesa» con el `recurso_id`; `registrar_interes` la agrega al historial del usuario activo. |
   | Analizar | `obtener_indice` valida el identificador (un id desconocido no modifica el estado) y obtiene el vector del recurso desde la matriz TF-IDF. |
   | Planificar | La regla de `actualizar_perfil`: combinar `0.4 * perfil + 0.6 * recurso` y normalizar, es decir, decidir el nuevo perfil. |
   | Ejecutar | Se reemplaza el vector actual del usuario y se recalculan `calcular_puntuaciones` y `recomendar`, mostrando el nuevo ranking. |
   | Conocimiento | El vectorizador y la matriz TF-IDF del catálogo, los perfiles (intereses iniciales, vector actual) y los historiales en memoria. |

6. **Limitación y mejora propuesta.**

   Limitación: el perfil solo aprende de señales positivas y cada interacción pesa
   0.6, de modo que un único clic desplaza fuertemente el perfil (R02 cae de 1.0 a
   0.688 tras una sola valoración) y no hay forma de expresar desinterés. Además, el
   estado se pierde al reiniciar. Mejora: incorporar retroalimentación negativa
   («No me interesa») con una regla tipo Rocchio,
   `perfil' = a * perfil + b * media(positivos) - c * media(negativos)`, recortando
   valores negativos a 0 y normalizando, y usar un `alpha` decreciente según el número
   de interacciones para estabilizar el perfil. Esto exige un dato adicional (el tipo
   de interacción positiva/negativa y su fecha en el historial) y persistir perfiles e
   historiales (por ejemplo en un archivo JSON o una base de datos) para que la
   adaptación se conserve entre sesiones.

## Archivos de la entrega

`app.py`, `motor.py`, `datos/recursos.csv`, `datos/perfiles.json`, `requirements.txt`,
`.gitignore`, este README y los dos archivos de `evidencias/`. No se incluyen `.venv`,
`__pycache__` ni dependencias instaladas.

### Reproducción desde una copia nueva

```bash
git clone https://github.com/Sptfff/Laboratorio-3.git
cd Laboratorio-3
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python app.py --comprobar
python app.py --demo
```

Instrucción adicional: activar el entorno antes de ejecutar `app.py`; con el
intérprete global falla la importación de `sklearn`. `requirements.txt` fija versiones
para Python 3.12; con otra versión de Python podría requerirse instalar
`numpy pandas scikit-learn` sin versión fija.
