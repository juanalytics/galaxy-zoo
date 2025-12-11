```markdown
# Clasificación Morfológica de Galaxias — Desafío Galaxy Zoo (Kaggle)

Este proyecto implementa una solución de **Aprendizaje Profundo** para la clasificación automática de galaxias, abordando el *Galaxy Zoo Challenge* de Kaggle. El objetivo es predecir directamente las características morfológicas galácticas a partir de imágenes astronómicas, automatizando el trabajo de miles de voluntarios de ciencia ciudadana.

---

## 1. Contexto del Proyecto

### 1.1. Justificación Científica

La astronomía moderna genera volúmenes inmensos de imágenes (SDSS, LSST, Euclid, JWST). Clasificarlas manualmente es impracticable. Este proyecto aborda tres problemas fundamentales:

- **Escalabilidad**: automatiza la clasificación morfológica a gran escala.  
- **Complejidad Cognitiva**: usa modelos de Deep Learning capaces de identificar estructuras sutiles (brazos espirales, elipses, interacciones).  
- **Generación de Conocimiento**: permite estudiar la evolución de las galaxias y su distribución en el universo observable.

Los beneficiarios principales incluyen astrónomos profesionales, investigadores en astrofísica y grupos académicos.

### 1.2. Producto Final

El proyecto entrega un modelo basado en **Redes Neuronales Convolucionales (CNNs)** capaz de predecir 37 atributos morfológicos continuos. Esta herramienta puede integrarse en flujos de análisis astronómico para prefiltrar y clasificar imágenes provenientes de misiones actuales y futuras.

---

## 2. Datos y Entendimiento del Problema

### 2.1. Origen y Estructura del Dataset

Los datos provienen del **Galaxy Zoo Challenge (Kaggle)**, una iniciativa de ciencia ciudadana en colaboración con Zooniverse.

**Resumen del dataset:**

| Característica | Detalle |
|----------------|---------|
| Volumen total | 141.553 imágenes (train: 61.578) |
| Tamaño total | ~3.73 GB |
| Formato | Imágenes `.jpg` y archivo tabular `.csv` |
| Tipo de problema | Regresión multietiqueta continua (37 clases “ClassX.Y”) |
| Etiquetas | Probabilidades entre 0 y 1 asignadas por voluntarios |

**Particularidades detectadas:**

- **Integridad:** sin datos faltantes ni archivos corruptos.  
- **Desbalanceo:** distribuciones desiguales en las probabilidades de clase.  
- **Redundancia:** correlaciones negativas casi perfectas entre pares de clases (ej. Class1.1 vs Class1.2, Class6.1 vs Class6.2), indicando exclusión mutua.

### 2.2. Preprocesamiento

Se implementó un pipeline de reducción eficiente:

- **Recorte centrado:** 180×180 px para eliminar bordes irrelevantes.  
- **Redimensión:** 160×160 px para reducir carga computacional.  
- **Normalización:** reescalado entre 0 y 1.

---

## 3. Diseño e Implementación Experimental

Se exploraron dos escenarios de CNNs, buscando equilibrio entre simplicidad y rendimiento.

### 3.1. Arquitecturas Consideradas

| Escenario | Modelo | Propósito |
|----------|--------|-----------|
| CNN simple | Arquitectura secuencial personalizada | Funciona como baseline y valida ideas iniciales. |
| Transfer Learning | MobileNetV2 (y pruebas con ResNet50) | Reutiliza aprendizaje previo (ImageNet) para mejorar generalización y acelerar convergencia. |

**Capa de salida:**  
Ambos modelos usan una capa densa de **37 neuronas** con activación **sigmoid**.  
**Función de pérdida:** `binary_crossentropy` (adecuada para tareas independientes por etiqueta).

### 3.2. Estrategia de Implementación

- **Generación de datos:** `ImageDataGenerator` + `flow_from_dataframe` con `class_mode='raw'`.  
- **Data Augmentation (solo Transfer Learning):** rotaciones, flips horizontales, etc.

---

## 4. Entrenamiento y Evaluación

### 4.1. Proceso de Entrenamiento

MobileNetV2 se entrenó en dos fases:

1. **Entrenamiento inicial**: capas convolucionales congeladas; solo se entrenan las capas densas agregadas.  
2. **Fine-tuning**: descongelamiento parcial (desde la capa 100) + tasa de aprendizaje baja (`1e−5`).  
   - Esto generó una caída temporal de rendimiento, esperable cuando la red reajusta pesos previamente optimizados para ImageNet.

### 4.2. Resultados

| Escenario | Métrica | Valor validación | Observación |
|----------|---------|------------------|-------------|
| CNN simple | Pérdida | 0.2451 | Precisión aproximada: 77.05% |
| Transfer Learning (MobileNetV2) | Pérdida | 0.2625 | Precisión aproximada: 67.21% |

### Conclusión técnica

La **accuracy** no es una métrica apropiada para problemas de **regresión multietiqueta continua**, ya que penaliza parcialmente las predicciones correctas.  
En futuras iteraciones se recomienda evaluar con:

- F1-score por etiqueta  
- AUC por clase  
- MAE o MSE por dimensión del vector objetivo

---
```
