# Respuestas

Nombre y código: Samuel Agudelo Sosa - 202459419

## Parte 3: el paso

¿Hasta qué paso el tiempo baja menos que las operaciones, y a partir de cuál baja al mismo ritmo? Relacione ese punto con el tamaño de un `int` y de una línea de caché.

Hasta el **paso 16**, el tiempo baja de forma significativamente menor respecto a la reducción de operaciones (de paso 1 a 16 las operaciones se reducen a 1/16, pero el tiempo solo baja de 15.5 ms a 11.5 ms). A partir del **paso 16 en adelante**, el tiempo empieza a reducirse al mismo ritmo que las operaciones (por ejemplo, al pasar de paso 16 a 64 las operaciones se dividen entre 4 y el tiempo baja proporcionalmente de 11.5 ms a 5.0 ms).

**Relación con el tamaño de `int` y la línea de caché:**
Una línea de caché típica en arquitecturas modernas tiene un tamaño de **64 bytes**. Dado que un tipo de dato `int` ocupa **4 bytes**, dentro de una sola línea de caché se alojan exactamente **16 enteros** ($64 \text{ bytes} / 4 \text{ bytes} = 16$). 

Cuando el tamaño de paso es menor a 16 (por ejemplo, paso 1, 2, 4 u 8), aunque se reducen las operaciones matemáticas, la CPU debe seguir cargando desde la memoria principal a la caché la línea completa de 64 bytes para leer al menos un entero. Dado que el factor limitante es el ancho de banda y la latencia del bus de memoria, traer la línea completa toma casi el mismo tiempo sin importar que solo leamos 1, 2 o 4 datos de ella. Solo cuando el paso supera los 16 elementos, la ejecución empieza a saltarse líneas de caché enteras, reduciendo drásticamente las transferencias de memoria y permitiendo que el tiempo baje proporcionalmente a las operaciones.

---

## Parte 4: speedup, eficiencia y la ley de Amdahl

Con los tiempos de `amdahl.txt`:

| Hilos | Total medido | Speedup medido | Eficiencia | Speedup según Amdahl |
|---:|---:|---:|---:|---:|
| 1 | 1499,4 ms | 1,00 | 1,00 | 1,00 |
| 2 | 990,4 ms | 1,51 | 0,76 | 1,61 |
| 4 | 865,9 ms | 1,73 | 0,43 | 2,33 |
| 8 | 688,5 ms | 2,18 | 0,27 | 2,99 |

Fracción paralelizable `p` (parte paralela sobre el total, con un hilo):
`p` = $1140,6 / 1499,4 =$ **0,7607** (o **76,07%**)

Techo del speedup con esa `p`, `1 / (1 - p)`:
`Techo` = $1 / (1 - 0,7607) =$ **4,18**

¿Dónde se separa la columna medida de la que predice Amdahl, y qué lo explica?

La columna del speedup medido empieza a separarse de la predicción de Amdahl a partir de **2 hilos** (1,51 vs. 1,61), pero la divergencia es acentuada con **4 hilos** (1,73 vs. 2,33) y **8 hilos** (2,18 vs. 2,99).

**Explicación:**
La Ley de Amdahl asume un modelo teórico ideal donde la porción paralelizable se escala perfectamente entre $N$ núcleos sin sobrecosto adicional. En la ejecución real, la brecha se explica por:

1. **Saturación del bus de memoria:** Con 4 u 8 hilos ejecutándose en paralelo, múltiples núcleos compiten simultáneamente por el acceso al ancho de banda de la memoria principal (RAM) y la caché L3 compartida, generando cuellos de botella por contención.
2. **Sobrecarga de gestión de hilos (Overhead):** La creación, distribución del trabajo, sincronización y destrucción de hilos en OpenMP/pthreads introduce un tiempo extra no contabilizado en el modelo teórico de Amdahl.
3. **Efectos de jerarquía de caché:** A medida que aumenta el número de hilos, los núcleos compiten por espacio en los niveles superiores de caché, incrementando la tasa de fallos de caché (*cache misses*).