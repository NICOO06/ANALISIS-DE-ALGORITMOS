
# BUSCADOR JUDICIAL — GRUPO 1

**Proyecto Final Integrador — Análisis de Algoritmos (SIST0175)**  
**Universidad Sergio Arboleda**

Sistema académico de búsqueda y recuperación de pasajes de expedientes judiciales mediante un índice invertido. El proyecto integra seis componentes algorítmicos para ordenar información, construir listas de posteo, sugerir correcciones ortográficas, seleccionar pasajes relevantes, consultar fechas y comparar implementaciones de atención numérica.

## Integrantes

- Nicolás Vargas
- Santiago Mejía
- Bryan Rodríguez

**Docente:** Juan David Rodríguez Martínez  
**Asignatura:** Análisis de Algoritmos (SIST0175)

---

## 1. Descripción del proyecto

El Buscador Judicial permite consultar un corpus sintético de expedientes mediante palabras clave, sugerencias ortográficas y filtros por fechas.

El sistema construye un índice invertido que relaciona cada término con los identificadores de los expedientes donde aparece. Las consultas de varios términos utilizan intersección AND, de modo que los resultados contienen todos los términos solicitados.

Adicionalmente, permite seleccionar pasajes relevantes sin superar un presupuesto de tokens y ejecutar una demostración numérica de atención exacta frente a atención por bloques.

> **Alcance académico:** el corpus contiene expedientes sintéticos. El sistema no procesa expedientes judiciales reales ni constituye una herramienta de asesoría jurídica. El componente C6 es una demostración numérica independiente y no un modelo semántico conectado al buscador.

## 2. Componentes algorítmicos

| Componente | Algoritmo o estructura | Aplicación |
|---|---|---|
| C1 | Merge Sort | Ordenamiento de pares (término, documento). |
| C2 | Arreglo dinámico con duplicación | Almacenamiento de listas de posteo del índice invertido. |
| C3 | Programación dinámica: Levenshtein + Trie | Sugerencias de términos similares y poda por umbral de distancia. |
| C4 | Algoritmo voraz: mochila fraccionaria | Selección de pasajes por relevancia/token bajo un presupuesto. |
| C5 | Árbol B+ en disco simulado | Indexación por fecha, consultas puntuales y por rango, y conteo de E/S. |
| C6 | Atención exacta y atención por bloques | Comparación de equivalencia numérica y uso de memoria. |

### C1 — Merge Sort

Ordena los pares término-documento para construir las listas de posteo.

Recurrencia:

`T(n) = 2T(n/2) + Θ(n)`

Complejidad:

`Θ(n log n)`

Se utiliza un ordenamiento propio y estable.

### C2 — Arreglo dinámico

Almacena los identificadores asociados a cada término. Cuando la capacidad se agota, el arreglo duplica su tamaño.

- Inserción con capacidad disponible: `Θ(1)`.
- Redimensionamiento individual: `Θ(n)`.
- Inserción amortizada: `O(1)`.

Se utiliza el método contable con tres créditos por inserción.

### C3 — Levenshtein y Trie

Calcula la distancia mínima de edición entre cadenas mediante programación dinámica.

- Tiempo: `Θ(mn)`.
- Espacio auxiliar con dos filas: `Θ(min(m,n))`.
- El Trie comparte prefijos y permite podar ramas cuando el mínimo de la fila supera el umbral establecido.

### C4 — Mochila fraccionaria

Selecciona pasajes de acuerdo con su densidad:

`densidad = relevancia / tokens`

Los pasajes se ordenan de mayor a menor densidad y se permite tomar una fracción del último pasaje.

- Ordenamiento: `Θ(P log P)`.
- Recorrido: `Θ(P)`.
- Complejidad total: `Θ(P log P)`.

La optimalidad se justifica mediante una prueba por intercambio. También se incluye un contraejemplo para mostrar que la misma regla no garantiza el óptimo en mochila 0/1.

### C5 — Árbol B+ y disco simulado

Indexa los metadatos de los expedientes por fecha.

Permite:

- Búsqueda puntual.
- Búsqueda por intervalo de fechas.
- Conteo de lecturas y escrituras de bloques.
- Uso de un búfer LRU limitado por `M/B`.

Parámetros de los experimentos:

- Orden del árbol: `f = 8`.
- Tamaño de bloque: `B = 4096 bytes`.
- Memoria simulada: `M = 16384 bytes`.
- Marcos de caché: `M/B = 4`.

Búsqueda puntual:

`Θ(log_f N)`

Búsqueda por rango:

`O(log_f N + R/B_leaf)`

Las E/S se contabilizan mediante la clase `DiscoSimulado`.

### C6 — Atención exacta y por bloques

Se comparan dos implementaciones de:

`Attention(Q,K,V) = softmax(QKᵀ / √d)V`

Ambas realizan:

`Θ(N²d)`

para el caso cuadrado con dimensiones compatibles.

La atención exacta requiere memoria temporal `Θ(N²)`, mientras que la versión por bloques reduce el uso de memoria a `Θ(Nd)` con bloques de tamaño fijo.

La equivalencia se comprueba numéricamente con:

- `rtol = 1e-10`
- `atol = 1e-12`

---

## 3. Estructura del repositorio

```text
BUSCADOR_JUDICIAL_GRUPO1/
│
├── C1_merge_sort/
│   ├── merge_sort.py
│   └── test_merge_sort.py
│
├── C2_arreglo_dinamico/
│   ├── arreglo_dinamico.py
│   ├── indice_invertido.py
│   └── test_arreglo_dinamico.py
│
├── C3_levenshtein/
│   ├── levenshtein.py
│   ├── vocabulario.py
│   └── test_levenshtein.py
│
├── C4_mochila_fraccionaria/
│   ├── mochila_fraccionaria.py
│   ├── test_mochila_fraccionaria.py
│   └── resultados/
│
├── C5_arbol_b_mas/
│   ├── arbol_b_mas.py
│   ├── disco_simulado.py
│   ├── test_arbol_b_mas.py
│   ├── experimentos_b_mas.py
│   └── graficas/
│
├── C6_atencion/
│   ├── atencion.py
│   ├── test_atencion.py
│   ├── experimentos_atencion.py
│   ├── resultados/
│   └── graficas/
│
├── datos/
│   ├── expedientes.json
│   └── generador.py
│
├── experimentos/
│   ├── README.md
│   └── ejecutar_todos.py
│
├── experimentos_graficas/
│
├── pruebas/
│   ├── test_generador.py
│   └── test_integracion.py
│
├── docs/
│   └── INFORME_TECNICO_FINAL_GRUPO1.pdf
│
├── main.py
├── experimentos_optimizacion.py
├── resultados_optimizacion.csv
├── requirements.txt
└── README.md
```

---

## 4. Requisitos

- Python 3.10 o superior.
- NumPy.
- Matplotlib.
- Pytest.

Las dependencias se encuentran en `requirements.txt`.

## 5. Instalación

### Windows — PowerShell

Desde la carpeta raíz del proyecto:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### Linux / macOS

```bash
python -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

---

## 6. Ejecución del buscador

Para iniciar la aplicación:

```bash
python main.py
```

El menú principal permite:

1. Buscar expedientes por palabras mediante intersección AND.
2. Buscar expedientes por rango de fechas utilizando el árbol B+.
3. Seleccionar pasajes relevantes mediante mochila fraccionaria.
4. Ejecutar la demostración numérica de atención de C6.
5. Consultar estadísticas generales.
0. Salir.

El corpus predeterminado se encuentra en:

`datos/expedientes.json`

El generador de datos está en:

`datos/generador.py`

---

## 7. Pruebas automatizadas

Para ejecutar la suite completa:

```bash
python -m pytest -v
```

Para ejecutar únicamente las pruebas de integración:

```bash
python -m pytest pruebas/test_integracion.py -v
```

### Resultado de validación

La revisión de esta versión se realizó con Python 3.13.5 y obtuvo:

**126 pruebas aprobadas.**

| Archivo de pruebas | Casos aprobados |
|---|---:|
| C1 — Merge Sort | 7 |
| C2 — Arreglo dinámico | 14 |
| C3 — Levenshtein | 21 |
| C4 — Mochila fraccionaria | 20 |
| C5 — Árbol B+ | 25 |
| C6 — Atención | 17 |
| Generador de datos | 7 |
| Integración | 15 |
| **Total** | **126** |

---

## 8. Experimentos reproducibles

Los experimentos utilizan `time.perf_counter()`, preparan los datos antes de medir y reportan medianas de cinco repeticiones cuando aplica.

### C1, C2 y C3

```bash
python experimentos_optimizacion.py
```

Genera:

- `resultados_optimizacion.csv`
- Gráficas de tiempo.
- Gráficas de memoria.
- Resultados del análisis amortizado.

### C4 — Mochila fraccionaria

```bash
python -m C4_mochila_fraccionaria.experimentos_mochila
```

Genera:

- CSV de tiempos.
- Gráfica log-log.
- Razón normalizada `T(P)/(P log₂P)`.

### C5 — Árbol B+

```bash
python -m C5_arbol_b_mas.experimentos_b_mas
```

Genera:

- CSV con lecturas y escrituras predichas.
- Conteo real de E/S.
- Resultados de consultas puntuales y por rango.
- Gráficas de operaciones de disco.

### C6 — Atención

```bash
python -m C6_atencion.experimentos_atencion
```

Genera:

- CSV de resultados.
- Gráfica de tiempos.
- Gráfica de memoria auxiliar.
- Comparación de equivalencia numérica.

También se encuentra una guía general en `experimentos/README.md`.

---

## 9. Resultados y consideraciones experimentales

Los resultados obtenidos muestran las tendencias esperadas:

- C1 presenta comportamiento consistente con `Θ(n log n)`.
- C2 mantiene un costo amortizado constante por inserción y saldo contable no negativo.
- C3 reduce considerablemente el consumo de memoria mediante dos filas.
- C4 presenta comportamiento consistente con el ordenamiento `Θ(P log P)`.
- C5 permite comparar las cotas de E/S con las operaciones contadas en el disco simulado.
- C6 conserva equivalencia numérica y reduce la memoria auxiliar frente a la atención exacta.

**Limitación de C5:** los experimentos realizados abarcan tamaños de 1.000 a 16.000 registros. La guía del curso propone como escala orientativa entre 100.000 y 1.000.000 de registros para memoria secundaria. Esta diferencia se documenta en el informe técnico y debe tenerse en cuenta al interpretar los resultados.

Los tiempos pueden variar dependiendo del equipo, sistema operativo, versión de Python y bibliotecas instaladas.

---

## 10. Informe técnico

El informe técnico contiene:

- Descripción del problema.
- Arquitectura de los seis componentes.
- Análisis teórico y pseudocódigo.
- Complejidades y demostraciones.
- Validación empírica.
- Gráficas y tablas de resultados.
- Comparación de E/S de C5.
- Discusión y conclusiones.
- Reparto de actividades.
- Anexos con capturas y evidencias de ejecución.

Ubicación:

`docs/INFORME_TECNICO_FINAL_GRUPO1.pdf`

---

## 11. Uso de herramientas de inteligencia artificial

Se utilizó ChatGPT como herramienta de apoyo para la revisión de requisitos, análisis de complejidad, depuración, diseño de pruebas, revisión de resultados experimentales y preparación de documentación técnica.

La responsabilidad académica sobre el código, los resultados, la comprensión de los algoritmos y la sustentación corresponde a los integrantes del equipo.

---

## 12. Referencias

Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2022). *Introduction to algorithms* (4th ed.). MIT Press.

Kleinberg, J., & Tardos, É. (2006). *Algorithm design*. Pearson.

Knuth, D. E. (1998). *The art of computer programming: Volume 3, sorting and searching* (2nd ed.). Addison-Wesley.

Rodríguez Martínez, J. D. (2026). *Proyecto Final Integrador – Análisis de Algoritmos (SIST0175)*. Universidad Sergio Arboleda.

OpenAI. (2026). *ChatGPT* [Modelo de lenguaje]. https://chatgpt.com/
