# Análisis de miARN por small RNA-seq

Este repositorio contiene todos los archivos y scripts necesarios para reproducir un pipeline de análisis de miARN desde datos de secuenciación (small RNA-seq) hasta la obtención de matrices de conteo y el análisis de expresión diferencial.

---

## Estructura general del repositorio

- **alineamiento_y_cuantificacion_mirna.md**  
  Documentación detallada del pipeline bioinformático: organización de carpetas, control de calidad, trimming, alineamiento y cuantificación de miARNs con FastQC, cutadapt y miRDeep2.

- **analisis_de_expresion_diferencial_miARN.rmd**  
  Documento reproducible en Rmarkdown con todo el flujo de análisis estadístico: importación de matrices de conteo, filtrado, análisis de expresión diferencial con DESeq2, visualización (boxplots, violin plots, PCA, dendrogramas, volcano plots, curvas ROC) y análisis de enriquecimiento funcional.


---

## Guía rápida de uso

1. **Procesamiento bioinformático y cuantificación de miARNs**  
   Sigue los pasos detallados en `alineamiento_y_cuantificacion_mirna.md` para procesar los archivos `.fastq.gz` hasta obtener la matriz de conteo de miARNs.

2. **Análisis estadístico y funcional**  
   Continúa con `analisis_de_expresion_diferencial_miARN.rmd` para realizar el análisis estadístico en R, identificar miARNs diferencialmente expresados y explorar su impacto biológico.

3. **Reproducibilidad**  
   Puedes ejecutar directamente los scripts o los notebooks `.Rmd` siguiendo el orden sugerido.

---

## Requisitos

- **FastQC**, **MultiQC**, **cutadapt**, **miRDeep2**
- **R** (con paquetes: `DESeq2`, `edgeR`, `limma`, `clusterProfiler`, `multiMiR`, `EnhancedVolcano`, etc.)

Consulta dentro de cada archivo `.md` o `.Rmd` para detalles de versiones y comandos.

---

## Contacto

Para dudas sobre el pipeline o propuestas de colaboración, contacta al autor principal del repositorio.

