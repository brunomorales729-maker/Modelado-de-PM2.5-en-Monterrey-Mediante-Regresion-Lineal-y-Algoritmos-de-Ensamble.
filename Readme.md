# Modelado Predictivo de PM2.5 en la ZMM: Regresión Lineal Múltiple vs. Random Forest

## Descripción General del Proyecto
Este proyecto de Ciencia de Datos analiza y predice las concentraciones diarias de material particulado fino (PM2.5) en la Zona Metropolitana de Monterrey (ZMM) a lo largo de un año. 

El objetivo es evaluar el impacto relativo de factores meteorológicos (presión atmosférica, precipitación) frente a la dinámica antropogénica (parque vehicular de automóviles, camiones de pasajeros y la pausa de los fines de semana), contrastando la eficacia de un modelo paramétrico tradicional (Regresión Lineal Múltiple) contra un algoritmo de aprendizaje automático basado en árboles de decisión (Random Forest Regressor).

## Origen de los Datos
La base de datos consta de 365 observaciones diarias generadas a partir de la consolidación y limpieza de múltiples fuentes oficiales:
* **Ambientales:** Registros históricos de calidad del aire extraídos del portal SIMA (Sistema Integral de Monitoreo Ambiental) y validados con el SINAICA federal[cite: 4].
* **Meteorológicas:** Datos físicos de la atmósfera extraídos mediante API (Meteostat).
* **Movilidad y Empresarial** Parque vehicular urbano y movimiento empresariales cuantificados a través de los registros del INEGI.

---

## Glosario Metodológico (Decisiones Técnicas Explicadas)
Para garantizar la validez matemática y evitar errores comunes en el Machine Learning, este proyecto implementó técnicas rigurosas que vale la pena destacar:

### 1. Blindaje contra la Fuga de Datos (Data Leakage)
Un error muy común es limpiar toda la base de datos antes de entrenar al modelo. En este proyecto, se aplicó un corte **Train/Test Split (80/20) como primer paso absoluto**. La detección de valores atípicos (*outliers*) y la Matriz de Correlación se calcularon **exclusivamente sobre los datos de entrenamiento**. Esto asegura que el algoritmo no haga trampa "espiando" la volatilidad climática de los datos del futuro que usaremos para examinarlo.

### 2. Variables Dummy (El manejo del Fin de Semana)
Los algoritmos matemáticos no entienden palabras ni conceptos de calendario. Para medir el impacto de la reducción de movilidad en sábados y domingos, se utilizó el truco estadístico de las **Variables Dummy**[cite: 3]. Se forzó esta característica a ser un número binario, donde los días hábiles equivalen a 0 y los fines de semana a 1, permitiendo que la ecuación sume o reste contaminación en base a esta etiqueta[cite: 3].

### 3. Selección de Variables (Backward Stepwise) y el P-Value
En lugar de forzar todas las variables disponibles en el modelo, se utilizó un algoritmo de eliminación hacia atrás. Este evaluó las variables una por una basándose en su **p-value** (valor de probabilidad). En ciencia de datos, el p-value funciona como un "detector de mentiras"; si el valor es menor a 0.05, rechazamos la Hipótesis Nula y comprobamos matemáticamente que la influencia de esa variable en la contaminación no es pura coincidencia[cite: 1, 3]. Las variables que no superaron esta prueba estadística (como la velocidad del viento) fueron eliminadas del modelo final.

### 4. R-cuadrado Ajustado como Métrica Guía
Al trabajar con regresiones múltiples, el R-cuadrado normal es una métrica "tramposa" que siempre sube si se agregan más variables, sin importar si son inútiles[cite: 3]. Las decisiones de este proyecto se basaron en el **R-cuadrado Ajustado**, ya que este penaliza al modelo y baja su calificación si se intentan introducir predictores que no aportan valor predictivo real[cite: 3].

---

## Resultados Principales

Ambos modelos compitieron de manera justa al ser evaluados sobre los mismos datos de prueba (Test) utilizando únicamente las variables sobrevivientes del análisis Stepwise:

1. **Regresión Lineal Múltiple (Ganador):**
   * **Desempeño:** Logró explicar casi el **35% de la variabilidad** de los datos en un entorno caótico.
   * **Hallazgos:** Demostró que la presión atmosférica y las lluvias actúan como los principales mitigadores físicos del PM2.5, mientras que el transporte público (camiones de pasajeros) presenta una correlación inversa con el aumento del contaminante, sugiriendo dinámicas ocultas en la movilidad urbana.

2. **Random Forest Regressor:**
   * **Desempeño:** Su R-cuadrado colapsó a menos del **10%** en la fase de prueba.
   * **Hallazgos:** Confirmó empíricamente que los algoritmos de ensamble complejos sufren un sobreajuste severo (*overfitting*) cuando intentan modelar sistemas altamente volátiles con un conjunto de datos pequeño (n=365). Ante la ausencia de la variable del viento, el bosque otorgó más del 52% de la importancia de decisión a un solo factor: la presión atmosférica.

## Conclusión
El enfoque estrictamente lineal demostró ser más robusto y adecuado para el volumen actual de registros. Sin embargo, para superar el techo de precisión del 35%, futuras iteraciones de este proyecto requerirán escalar la recolección de datos a frecuencias horarias, lo que permitirá explotar el verdadero potencial predictivo de los algoritmos de naturaleza no lineal.
