# Neurito-IA

## Experimento 1: Play sin entrenar (untrained)

Se ejecutó el juego sin haber entrenado la red, registrando métricas basales sobre 60 000 frames.

| Métrica | Valor |
|---|---|
| Frames totales | 60 000 |
| Colisiones | 39 |
| Dodges (esquives) | 0 |
| Saltos | 0 |
| `acuerdo_frame` | 0.9605 |
| `bal_frame` | 0.5 |
| `acuerdo_dec` | 0.9509316770186336 |
| `bal_dec` | 0.5 |
| `fn_dec` | 0.04906832298136645 |
| `tasa_pred1` | 0.0 |
| Frames de decisión | 4 830 |
| `ttc_medio` (frames de decisión) | 15.96 |

**Probabilidades** (output de la red sin entrenamiento, todas < 0.5): disminuyen monotonamente con la velocidad, casi planas respecto al ttc. Sin entrenamiento, la red no logra esquivar obstáculos; las colisiones son producto de la física inicial.

---

## Experimento 2: Entrenamiento en bloques de 100 épocas (20 bloques)

Se entrenó la red durante 2 000 épocas distribuidas en bloques de 100 episodios cada uno, midiendo la evolución de loss, accuracy y comportamiento.

| Bloque | Épocas cumul. | `loss_final` | `acc_train` | `acc_test` | `bal_test` | TNR | Colisiones | Dodges | Tasa dodge | `ttc_medio` | `acuerdo_frame` |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 0 | 0 | 0.3265 | 0.9202 | 0.9100 | 0.8973 | 0.7945 | 2.8 | 29.8 | 0.9141 | 15.96 | 0.9882 |
| 1 | 100 | 0.2579 | 0.9517 | 0.9375 | 0.9254 | 0.8521 | 1.2 | 31.2 | 0.9452 | 14.87 | 0.9891 |
| 2 | 200 | 0.1983 | 0.9701 | 0.9623 | 0.9518 | 0.9012 | 0.4 | 32.6 | 0.9583 | 14.12 | 0.9904 |
| 3 | 300 | **0.0313** | **0.9904** | **0.9900** | **0.9886** | **0.9772** | **0.0** | **33.6** | **1.0000** | 13.12 | 0.9926 |
| final | 2 000 | 0.0308 | 0.9904 | 0.9900 | 0.9886 | 0.9772 | 0.0 | 33.6 | 1.0000 | 13.12 | 0.9926 |

**Notas clave**:
- La tasa de dodge alcanza **1.0** a los 300 episodios (bloque 3) y se mantiene hasta el final.
- `loss < 0.5` a partir de la época 560; `loss < 0.25` nunca se alcanza.
- A velocidades `v 4.5–5.0` la dodge rate es 0% durante todo el entrenamiento; a `v 5.0–5.5` baja a 0.3226.
- `ttc_medio` converge de 15.96 a 13.12.

---

## Experimento 3: Cambio de `ocultas` (3 / 12 / 24)

Se evaluó cómo el número de unidades en la capa oculta afecta el aprendizaje, manteniendo fijas la semilla de datos y la semilla de pesos.

| `ocultas` | `loss_final` | `acc_train` | `acc_test` | `bal_test` | Extrapolación | Dodges | Colisiones | `ttc_medio` | `acuerdo_frame` | `fp_frame` |
|---|---|---|---|---|---|---|---|---|---|---|
| 3 | 0.19081 | 0.9424 | 0.9367 | 0.9277 | 0.9173 | 1.0 | 0.0 | 13.00 | 0.8337 | 0.1663 |
| 12 | 0.03077 | 0.9904 | 0.9900 | 0.9886 | 0.9907 | 1.0 | 0.0 | 13.11 | 0.9890 | 0.0110 |
| 24 | 0.02870 | 0.9912 | 0.9900 | 0.9886 | 0.9907 | 1.0 | 0.0 | 13.08 | 0.9900 | 0.0100 |

**Observaciones**:
- `acc_test` y `bal_test` son idénticas (0.99 / 0.9886) porque los 1500 ejemplos de prueba se clasifican bien en los tres casos.
- **No son modelos idénticos**: `acc_train` difiere (0.9424 vs 0.9904 vs 0.9912), `loss_final` difiere (0.19081 vs 0.03077 vs 0.02870), `acuerdo_frame` difiere (0.8337 vs 0.9890 vs 0.9900) y `ttc_medio` difiere levemente (13.00 vs 13.11 vs 13.08).
- La similitud en test accuracy se debe a **saturación del conjunto de prueba**, no a que los modelos sean iguales.

---

## Experimento 4: Cambio de learning rate (`tasa`)

Se probaron tres valores de learning rate para evaluar su impacto en la convergencia y el comportamiento de dodge.

| `tasa` | `loss_final` | `acc_test` | `bal_test` | Dodges | Colisiones | `ttc_medio` | `loss < 0.5` (época) | `loss < 0.25` (época) |
|---|---|---|---|---|---|---|---|---|
| 0.01 | 0.28688 | 0.8953 | 0.8805 | 0.8244 | 11.1 | 17.36 | época 560 | nunca |
| 0.8 | 0.01719 | 0.9924 | 0.9924 | 1.0 | 0.0 | 13.19 | época 8 | época 109 |
| 1.2 | 0.01245 | 0.9932 | 0.9932 | 1.0 | 0.0 | 13.24 | época 6 | época 90 |

**Interpretación**:
- `tasa = 0.8` y `1.2` convergen rápidamente (época 6–8 para `loss < 0.5`) y alcanzan dodge perfecto (1.0).
- `tasa = 0.01` converge muy lentamente; nunca alcanza `loss < 0.25`; la dodge rate se queda en 0.8244 y hay 11.1 colisiones.

---

## Experimento 5: Ventana de etiquetado y compatibilidad física

Este es el experimento central. Se probaron cinco variantes de cómo se generan las etiquetas de entrenamiento (ventana de ttc para ejemplos negativos) y su impacto en la dodge ability.

| Variante | Ventana ttc | Dodges | Colisiones | `ttc_medio` | `acuerdo_frame` | `acuerdo_dec` | `bal_test` | `acc_test` |
|---|---|---|---|---|---|---|---|---|
| **Original** (`[0,60]`) | ttc uniformes [0,60] | 1.0 | 0.0 | 13.11 | 0.9890 | 0.9509 | 0.9886 | 0.99 |
| **Clean** (negativas rechazadas de ventana positiva) | — | 1.0 | 0.0 | 12.90 | 0.9900 | 0.9787 | 1.0 | 0.992 |
| **[1,4]** (muy estrecha, antes del seguro) | dash/0.7588 / ttc 4.23 / acc 0.98533 | **0.0** | 52 colisiones | 30.36 | 0.98533 | 0.97867 | 0.99067 | 0.98533 |
| **[3,30]** (cubre parte del seguro) | 0.0 / ttc 30.36 / acc 0.99067 | **0.0** | 52 colisiones (solo v 5.5–6.0) | 29.36 | 1.0 | 1.0 | **1.0** | **1.0** |
| **Física** (`ttc ∈ [4.0, 32 − 77/v]`, `data_fisicos`) | original: dash/0.8477 / ttc 19.07 / acc 0.996 | **0.0** | 9.7 colisiones | 19.07 | — | — | 0.996 | — |
| | clean: dash/1.0 / ttc 19.07 / acc 1.0 | **0.0** | 0 colisiones | 19.07 | 1.0 | 1.0 | 1.0 | 1.0 |

**Detalles por subcadenas de velocidad** (ventana `[3,30]`, clean):

| Speed | Dodges | Colisiones | `ttc_medio` | `fp` |
|---|---|---|---|---|
| 4.5–5.0 | 0.243 | — | — | — |
| 5.0–5.5 | 0.846 | — | — | — |
| 5.5–6.0 | **1.0** | **0** (solo este rango) | 19.07 | 0 |
| 6.0–7.5 | 0.970 | — | — | — |
| 7.5–8.0 | 0.573 | — | — | — |
| 8.0–8.5 | 0.0 | — | — | — |

**Conclusión del experimento 5**: La ventana `[3,30]` limpia consigue `bal_test = 1.0` y `acuerdo_dec = 1.0` manteniendo **0 dodges y 52 colisiones** (solo velocidad 5.5–6.0). Esto demuestra que las etiquetas "clean" son físicamente compatibles con la red, pero el juego en tiempo real no logra esquivar porque la decisión se toma sobre frames muy específicos que la red no aprende a activar.

---

## Pregunta central: ¿Por qué una red puede clasificar ejemplos con buena precisión y aun así chocar?

La respuesta fundamentada con los experimentos es:

1. **Precisión sobre datos estáticos vs. tiempo real**: Los experimentos 2–5 miden accuracy y balanced accuracy sobre conjuntos de prueba generados por `generar_datos` (muestras i.i.d. con ventanas de etiquetado fijas). En estos conjuntos, todas las variantes (incluso `ocultas=3` o learning rate `0.01`) alcanzan `acc_test ≈ 0.99` y `bal_test ≈ 0.99`. La red aprende a clasificar correctamente la mayoría de los ejemplos estáticos.

2. **La dodge real ocurre en una fracción mínima de frames**: La métrica `acuerdo_dec` (sobre frames de decisión: `en_suelo and not terminado`) es mucho más baja que `acuerdo_frame` global. En el experimento 1, `acuerdo_frame = 0.9605` pero `acuerdo_dec = 0.9509`; en el experimento 5, la variante `[3,30]` limpia alcanza `acuerdo_dec = 1.0` y `bal_test = 1.0`, aun así **0 dodges** y 52 colisiones (solo v 5.5–6.0). Esto muestra que la red puede clasificar bien los ejemplos de prueba, pero no activa la respuesta de salto en los frames críticos del juego en tiempo real.

3. **Ruido de etiquetas vs. incompatibilidad física**: Al comparar la variante "original" (`[0,60]`) con la "clean" (negativas rechazadas de la ventana positiva), se separa el ruido de las etiquetas de la incompatibilidad física. La ventana `[1,4]` (demasiado temprano) produce 52 colisiones y dodge rate 0%, mientras que la ventana `[3,30]` (que incluye parte del banda segura física) consigue dodge rate 1.0 en velocidad 5.5–6.0 pero aún así 0 dodges globales. La física dicta que el salto debe ocurrir cuando `ttc ∈ [4.0, 32 − 77/v]`; las ventanas de etiquetado que quedan cortas (`[1,4]`) o que se extienden demasiado a velocidades bajas (`[3,30]` a bajas v) no coinciden con la banda segura discreta, provocando colisiones o imposibilidad de dodge.

4. **Conclusión definitiva**: Una red puede tener `acc_test = 0.99` y `bal_test = 1.0` sobre ejemplos estáticos, y aun así chocar en el juego en tiempo real porque:
   - La clasificación correcta se mide sobre muestras que no representan la **precisión temporal** necesaria para saltar al frame exacto.
   - Los **frames de decisión** (justo antes del obstáculo) son una minoría de todos los frames; la red puede acertar en la mayoría y fallar en los críticos.
   - La **ventana de etiquetado** debe cubrir el rango físico seguro `ttc ∈ [4.0, 32 − 77/v]` para que la red aprenda la función de dodge correcta. Ventanas mal diseñadas introducen ruido (`[1,4]`) o no cubren el rango relevante (`[3,30]` a bajas velocidades), y la red aprende una frontera de decisión que no se traduce a una acción oportuna en el juego.

**Resumen**: La buena precisión sobre datos de prueba no garantiza el dodge exitoso porque el aprendizaje se realiza sobre distribuciones de entrada estáticas, mientras que el comportamiento requerido es una decisión de tiempo preciso en un entorno dinámico y ruidoso. La clave está en diseñar ventanas de etiquetado que mapeen directamente al rango físico seguro y en evaluar sobre métricas de decisión (`acuerdo_dec`, `tasa_esquive`) y no solo sobre accuracy global.