# Model Card · Fashion-MNIST CNN

## Modelo y versión
- Arquitectura: `small_cnn` (Conv2D 32 → MaxPooling2D → Conv2D 64 → MaxPooling2D → GlobalAveragePooling2D → Dropout(0.25) → Dense(10, softmax)), definida en `src/inf8239_u02_cv/models.py`.
- Framework: TensorFlow/Keras `>=2.21,<2.22`.
- Checkpoint entrenado: `models/best_cnn.keras`, guardado por `ModelCheckpoint(save_best_only=True)` durante un entrenamiento de 8 épocas con semilla fija (`SEED=42`, `TF_DETERMINISTIC_OPS=1`).
- Comparado contra una línea base `dense_baseline` (Flatten → Dense(64, relu) → Dense(10, softmax)) entrenada con el mismo protocolo, mismos datos y mismo número de épocas.

## Uso previsto
- Clasificar imágenes en escala de grises de 28×28 px de prendas y calzado del dominio Fashion-MNIST en una de 10 categorías (T-shirt/top, Trouser, Pullover, Dress, Coat, Sandal, Shirt, Sneaker, Bag, Ankle boot).
- Uso educativo y comparativo: sirve como caso de estudio para contrastar el costo real de una CNN frente a una red densa simple bajo el enfoque "Green AI" — no está pensado como modelo de producción.

## Usos fuera de alcance
- Fotografías reales de prendas (fondos variados, iluminación, ángulos, color): el modelo solo vio imágenes centradas, en escala de grises, de 28×28 px, normalizadas entre 0 y 1.
- Distinguir de forma confiable prendas superiores similares: la clase "Shirt" obtiene F1 de solo 0.427 y se confunde sistemáticamente con T-shirt/top, Pullover y Coat.
- Decisiones de catálogo, inventario o atención al cliente sin revisión humana, dado el error alto y concentrado en una categoría.
- Cualquier dominio de imágenes distinto a Fashion-MNIST (otros datasets de moda, otra resolución, otro recorte).

## Dataset y particiones
- Fashion-MNIST: 60,000 imágenes de entrenamiento + 10,000 de prueba, 10 clases balanceadas (1,000 imágenes por clase en el set de prueba).
- División interna: las últimas 6,000 filas del set de entrenamiento original se separan como validación (54,000 entrenamiento / 6,000 validación), sin mezclarse con el set de prueba.
- Descarga automática vía `tf.keras.datasets.fashion_mnist.load_data()`.

## Preprocesamiento
- `normalize_images`: conversión a `float32`, escalado a rango [0, 1] (división por 255) y adición de canal — de (28, 28) a (28, 28, 1).
- `validate_images`: verifica forma `(N, 28, 28, 1)`, correspondencia de longitud entre imágenes y etiquetas, y que los valores estén normalizados en [0, 1]. Contrato cubierto por `tests/test_data.py` (2 pruebas, ambas superadas).

## Métricas globales y por clase
Resultado sobre el set de prueba completo (10,000 imágenes), ejecución real de `uv run python scripts/train_cv.py --epochs 8`.

Global: **accuracy 0.793**, **F1 macro 0.789** (consistente con `reports/cv_metrics.json`).

| Clase | Precisión | Recall | F1 |
|---|---|---|---|
| 0 · T-shirt/top | 0.643 | 0.806 | 0.715 |
| 1 · Trouser | 0.989 | 0.931 | 0.959 |
| 2 · Pullover | 0.707 | 0.643 | 0.674 |
| 3 · Dress | 0.754 | 0.843 | 0.796 |
| 4 · Coat | 0.660 | 0.664 | 0.662 |
| 5 · Sandal | 0.951 | 0.884 | 0.916 |
| 6 · Shirt | 0.500 | 0.373 | 0.427 |
| 7 · Sneaker | 0.854 | 0.953 | 0.901 |
| 8 · Bag | 0.916 | 0.937 | 0.926 |
| 9 · Ankle boot | 0.935 | 0.896 | 0.915 |

Clase más débil: **Shirt (6)**, F1 0.427. La precisión baja (0.500) muestra que el problema no es solo que a "Shirt" le cueste ser reconocida (recall 0.373): además, muchas prendas de otras clases —sobre todo T-shirt/top y Pullover— se predicen incorrectamente como "Shirt". Es un error bidireccional dentro del grupo de prendas superiores.

## Comparación de costo
- Hardware: CPU local del estudiante, sin GPU.
- Parámetros: `dense_baseline` 50,890 vs. `small_cnn` 19,466 — la CNN tiene menos parámetros, pero sus capas convolucionales son mucho más costosas de ejecutar por parámetro.
- Tiempo de entrenamiento (8 épocas): dense 13.92 s vs. CNN 101.82 s → **7.3× más caro**.
- Tiempo de inferencia: dense 0.090 ms/imagen vs. CNN 0.166 ms/imagen → **1.8× más caro**.
- F1 macro: dense 0.863 vs. CNN 0.789 → la CNN además rinde peor.
- Conclusión Green AI: bajo este presupuesto de 8 épocas, la arquitectura más simple (dense) es estrictamente mejor en las dos dimensiones que importan — exactitud y costo. No hay justificación costo-beneficio para preferir la CNN en este punto del experimento.

## Limitaciones y riesgos
- El entrenamiento de la CNN no había convergido en la época 8: su pérdida seguía bajando con pendiente pronunciada (0.75→0.70 entre las épocas 7 y 8), mientras la dense ya se acercaba a una meseta. La comparación es válida para el presupuesto de cómputo fijado (8 épocas, igual para ambos modelos), pero no demuestra cuál sería el techo de la CNN con más épocas de entrenamiento.
- Riesgo principal de uso: la clase Shirt concentra la mayoría de los errores (62.7% de los shirts reales no se clasifican como shirt), lo que hace al modelo inadecuado para cualquier flujo que dependa de distinguir esa categoría con confianza.
- El dataset es de baja resolución (28×28, escala de grises) y no representa la variabilidad de una fotografía real de producto.

## Supervisión y monitoreo
- Antes de reutilizar esta arquitectura en otro dominio de imágenes, repetir la auditoría de las clases con peor F1 y confirmar si el patrón de confusión entre prendas superiores se mantiene.
- Tratar cualquier predicción de la clase Shirt con baja confianza por defecto y, si es posible, solicitar confirmación humana.
- Antes de descartar definitivamente la CNN frente a la dense, repetir el experimento con un presupuesto de épocas mayor (y el costo adicional que eso implica), dado que la CNN no llegó a converger bajo las 8 épocas evaluadas.
