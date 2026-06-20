# liga1-xg-ml-showcase

Extracto de un proyecto de Machine Learning más amplio (tesis de pregrado) orientado a predecir el rendimiento ofensivo de los equipos de la **Liga 1 de Perú** a partir de su forma reciente.

> ⚠️ **Importante:** este repositorio es un **extracto / muestra** del proyecto completo, no el código íntegro. No incluye el web scraping, la limpieza de datos ni la ingeniería de características completa (viven en otros notebooks del proyecto principal). El objetivo aquí es mostrar dos piezas clave del pipeline: **cómo se definieron los umbrales de las variables objetivo** y **cómo se entrenó y validó el modelo final**. Los resultados de cada paso ya quedaron impresos como output de las celdas, por lo que pueden revisarse directamente sin necesidad de re-ejecutar los notebooks.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `Seleccion_Umbrales_Target.ipynb` | Análisis exploratorio que justifica los umbrales usados para convertir las métricas continuas en variables objetivo binarias. |
| `Modelo_Predictivo_Goles.ipynb` | Entrenamiento, carga del modelo ya optimizado, validación y backtesting para la variable objetivo `Goles ≥ 2`. |
| `data_ejemplo_liga1.csv` | Muestra cruda de un partido tal como se obtiene del web scraping (estadísticas `_local` / `_visitante` por equipo). |

## ¿Por qué esos umbrales?

Las variables objetivo del proyecto completo (`xG ≥ 1.5`, `Tiros a puerta ≥ 5`, `Goles ≥ 2`) no se fijaron de forma arbitraria. Se determinaron en `Seleccion_Umbrales_Target.ipynb` mediante:

1. **Análisis de distribución**: histogramas, percentiles y media de cada métrica calculados sobre el histórico real de la Liga 1 de Perú, comparando distintos puntos de corte y el balance de clases (0/1) que cada uno produce.
2. **Criterio de balance**: se buscó un corte que generara una proporción razonable entre clase 0 y clase 1 (evitando umbrales extremos que dejaran clases con <15-20% de los datos), sin perder el sentido futbolístico de la métrica.
3. **Contraste con literatura deportiva**: los valores finales también se validaron contra criterios cualitativos usados habitualmente en artículos y análisis de fútbol (por ejemplo, un `xG ≥ 1.5` suele considerarse una jornada de alta generación de ocasiones, y `5+ tiros a puerta` es un umbral común para calificar un partido como ofensivamente sólido).

Estos umbrales están calibrados para las características estadísticas **específicas de la Liga 1 peruana** y pueden no ser directamente trasladables a otras ligas o fuentes de datos (otra API podría calcular `xG` distinto, o pertenecer a una liga con otro nivel ofensivo). Si se replica este enfoque en otro contexto, se recomienda repetir el análisis de distribución antes de fijar los umbrales.

## Modelado: comparación de algoritmos

En el proyecto completo se entrenaron y optimizaron (con Optuna) **4 algoritmos clasificadores** para cada variable objetivo: **XGBoost, LightGBM, Random Forest y Logistic Regression**. Para la variable `Goles ≥ 2` (la incluida en este repositorio), el comparativo de métricas sobre el set de test fue:

| Modelo | Accuracy | AUC-ROC | F1-Score | Precision | Recall |
|---|---|---|---|---|---|
| XGBoost | 63.03% | 0.5911 | 0.4265 | 0.5524 | 0.3473 |
| LightGBM | 64.69% | 0.6131 | 0.4419 | 0.5900 | 0.3533 |
| Random Forest | 61.85% | 0.6056 | 0.4390 | 0.5250 | 0.3772 |
| Logistic Regression | 59.24% | 0.6452 | 0.5497 | 0.4884 | 0.6287 |

Los 4 modelos mostraron un desempeño **relativamente cercano entre sí** (diferencias de pocas décimas en accuracy/AUC), sin un ganador absoluto en todas las métricas. Para las **predicciones finales del sistema se eligió XGBoost**, priorizando su consistencia de desempeño a través de las 3 variables objetivo del proyecto completo (`xG`, `Tiros a puerta`, `Goles`), su madurez para producción (despliegue como API) y su buen soporte de interpretabilidad vía SHAP, usado en el análisis de importancia de variables.

## Datos de entrenamiento y validación

- **Train/Test (80/20, división temporal)**: partidos de la Liga 1 desde **2023** hasta el **27/04/2026** (`bd_liga1_Peru_entrenamiento.csv`). La división es cronológica, no aleatoria, para evitar que el modelo "vea" partidos futuros durante el entrenamiento.
- **Backtesting real**: el modelo ya entrenado se evaluó contra los resultados reales de las **jornadas 13, 14, 15, 16 y 17** (partidos posteriores al corte de entrenamiento, nunca vistos por el modelo). En este backtesting el modelo de `Goles ≥ 2` obtuvo **58.89% de accuracy** sobre 90 partidos evaluados, con resultados por ronda de 61%, 33%, 67%, 78% y 56% respectivamente.

## Stack tecnológico

`Python` · `pandas` / `numpy` · `XGBoost` · `LightGBM` · `scikit-learn` · `Optuna` (optimización de hiperparámetros) · `imbalanced-learn` (SMOTE) · `SHAP` (interpretabilidad) · `matplotlib`
