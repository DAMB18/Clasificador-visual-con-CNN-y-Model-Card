# Declaración de uso de IA

## Herramienta utilizada
Claude (Anthropic), en su modalidad Cowork, vinculada al computador del estudiante mediante un puente
de dispositivo. La IA no ejecutó comandos de forma remota en ningún momento: toda ejecución real
(entrenamiento, pruebas, notebook, git) la realizó el estudiante en su propia máquina; la IA leyó y
escribió archivos del proyecto e interpretó la salida que el estudiante pegó en la conversación.

## Tareas en las que se usó IA como apoyo
- Revisión de la implementación ya existente del proyecto (`src/inf8239_u02_cv`, `scripts/train_cv.py`)
  frente al mandato del Ejercicio 04 y la rúbrica oficial, para identificar qué faltaba.
- Interpretación de las curvas de pérdida y exactitud por época (a partir del log real pegado por el
  estudiante) y de las imágenes de matriz de confusión y errores ya generadas por su ejecución.
- Identificación de un conflicto entre `.gitignore` y el paso de Git de la guía: el archivo excluía en
  silencio `models/*.keras`, `reports/*.png` y `reports/*.json`, justo los artefactos que la guía pide
  versionar como evidencia. Se corrigió antes del primer commit.
- Redacción de `MODEL_CARD.md`, del notebook de verificación (`notebooks/resumen_resultados.ipynb`) y
  del cierre interpretativo añadido a `README.md`, siempre a partir de cifras generadas por la
  ejecución real del estudiante, nunca estimadas.
- Estructuración de los commits de Git (separar el esqueleto del proyecto de la evidencia del
  experimento y del cierre interpretativo) y redacción de los mensajes de commit.

## Partes verificadas con datos reales (no inventadas)
- Las métricas de `reports/cv_metrics.json` y la tabla de precisión/recall/F1 por clase provienen de la
  ejecución real de `scripts/train_cv.py --epochs 8` por el estudiante en su equipo.
- El notebook `notebooks/resumen_resultados.ipynb` vuelve a verificar esas métricas de forma
  independiente: carga el checkpoint real (`models/best_cnn.keras`), recalcula las predicciones sobre
  el set de prueba completo y compara el F1 macro obtenido contra el reportado en el JSON.
- Las imágenes `confusion_cnn.png` y `cnn_errors.png` son las generadas por esa misma ejecución del
  estudiante, no recreaciones ni aproximaciones.
- El hallazgo central del ejercicio ("Green AI": la CNN rinde peor y cuesta más que la red densa bajo
  el mismo presupuesto de épocas) se basa en los números reales reportados, no en un resultado asumido
  de antemano — incluyendo el matiz de que la CNN no había convergido en la época 8.

## Correcciones realizadas con ayuda de IA
- Corrección del `.gitignore` para no excluir silenciosamente los artefactos que debían versionarse.
- Corrección de la línea de autor en `README.md`, que contenía un nombre de plantilla sin reemplazar.
- División de los commits de Git en una estructura con sentido (esqueleto del proyecto → evidencia del
  experimento → cierre interpretativo) en lugar de un único `git add .`.

## Responsabilidad del estudiante
El estudiante (Daniel Alexander Batista) ejecutó personalmente todo el código en su propio equipo,
revisó cada archivo antes de incorporarlo al repositorio, y puede explicar cualquier fragmento de
código, resultado o conclusión presentada en esta entrega. Ningún dato, métrica o referencia fue
inventado: todo proviene de ejecuciones reales documentadas en este repositorio.
