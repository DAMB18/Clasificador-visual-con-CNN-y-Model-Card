# INF-8239 · Unidad 02 · Proyecto de visión

Autor académico: Edwin Ramón José Nolasco

## CPU
```bash
uv python install 3.12
uv sync --extra cpu
uv run python scripts/check_runtime.py
uv run python scripts/train_cv.py --epochs 8
```

## GPU
La ruta GPU se utiliza solamente en WSL2/Linux con NVIDIA correctamente configurada:
```bash
uv sync --extra gpu
```

Complete `MODEL_CARD.md` después de analizar métricas, errores y costo.

## Cierre interpretativo

- **Resultado principal:** bajo un presupuesto fijo de 8 épocas, la CNN (`small_cnn`) no superó a la línea base densa (`dense_baseline`). F1 macro: dense 0.863 vs. CNN 0.789 (accuracy CNN 0.793).
- **Evidencia predictiva:** `reports/cv_metrics.json`, `reports/confusion_cnn.png`, `reports/cnn_errors.png` y el `classification_report` por clase (ver `MODEL_CARD.md`).
- **Clase más difícil:** Shirt (6), F1 0.427 — precisión 0.500 y recall 0.373. No es solo que "Shirt" cueste reconocer: T-shirt/top y Pullover también se confunden hacia "Shirt", es un error bidireccional dentro del grupo de prendas superiores.
- **Costo comparado:** la CNN cuesta 7.3× más tiempo de entrenamiento (101.82 s vs. 13.92 s) y 1.8× más tiempo de inferencia (0.166 ms vs. 0.090 ms por imagen) que la densa, con menos parámetros (19,466 vs. 50,890) pero capas convolucionales más caras de ejecutar.
- **¿La mejora justifica el costo?** No. Bajo el presupuesto evaluado, la CNN es a la vez peor y más cara — no hay mejora que justificar. Matiz importante: ninguno de los dos modelos activó `EarlyStopping` y, a la época 8, la pérdida de la CNN seguía bajando con pendiente pronunciada mientras la densa ya se acercaba a una meseta — es decir, la CNN no tuvo tiempo de converger bajo este presupuesto, así que esta comparación no determina su techo real, solo su rendimiento bajo un gasto de cómputo igual al de la densa.
- **Limitación del benchmark:** Fashion-MNIST son imágenes pequeñas (28×28), en gris y centradas — no representan la variabilidad de una fotografía real de producto. Además, comparar ambos modelos con el mismo número fijo de épocas puede penalizar injustamente a la arquitectura que converge más lento (la CNN), aunque sea la más adecuada para otro tipo de datos.
- **Decisión antes de usar otro dominio:** antes de aplicar esta CNN (o descartarla) en un dominio nuevo, repetir la comparación con un criterio de convergencia igualado entre modelos (por ejemplo, early stopping con la misma paciencia y un presupuesto de épocas suficiente para ambos, no un número fijo) y auditar específicamente el desempeño en clases visualmente similares, como se hizo aquí con "Shirt".
