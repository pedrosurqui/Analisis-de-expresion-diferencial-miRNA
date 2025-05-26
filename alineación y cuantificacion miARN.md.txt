Análisis de secuencias pequeñas de RNA (small RNA-seq) para la identificación y cuantificación de miARNs usando herramientas como FastQC, cutadapt, y miRDeep2.

1. Organización del directorio de trabajo
Se crearon las siguientes carpetas para organizar los archivos generados durante cada etapa del análisis.

mkdir -p 02_trimmed                      # Lecturas recortadas
mkdir -p 02_trimmed_unzipped	# Lecturas recortadas descomprimidas
mkdir -p 03_qc_fastp                     # Resultados de calidad
mkdir -p cutadapt_logs                   # Logs del trimming
mkdir -p 04_mapper                       #Archivos intermedios para mapeo
mkdir -p mapper_logs                     # Logs del mapeo
mkdir -p 05_mirdeep/results              # Resultados finales de miRDeep2

2.	Control de calidad inicial (FastQC + MultiQC)
Antes del preprocesamiento, se evaluó la calidad de las secuencias crudas para detectar posibles contaminaciones, sobrecorte o baja calidad.

fastqc /mnt/e/raw_data_merged/*.fastq.gz -o 03_qc_fastp
multiqc 03_qc_fastp -o 03_qc_fastp

Herramientas: FastQC v0.11.9, MultiQC v1.14

3.	Recorte de adaptadores con Cutadapt

Se eliminaron adaptadores específicos para bibliotecas de small RNA y artefactos técnicos (poli-A, poli-G), aceptando secuencias entre 18-30 nucleótidos:

cutadapt \
  -a TGGAATTCTCGGGTGCCAAGG \                      # Adaptador 3′ típico small RNA
  -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC \         # Adaptador universal TruSeq
  -a "A{10}" -a "G{10}" \                         # Artefactos homopolímeros
  --overlap 3 -m 18 -M 30 --trim-n --quiet \
  -o 02_trimmed/${sample}_trim.fastq.gz "$fq" \
  > cutadapt_logs/${sample}.txt
Herramienta: Cutadapt v4.5

4.	Colapso y mapeo de secuencias (miRDeep2 mapper.pl)

Las secuencias recortadas se colapsaron para reducir la redundancia y se mapearon contra el genoma de referencia (GRCh37). Se generaron archivos .fa y .arf para cada muestra:

mapper.pl "$fq" -e -h -m -l 18 -o 4 \
  -p "$GENOME_IDX" \
  -s "04_mapper/${sample}.fa" \
  -t "04_mapper/${sample}.arf" \
  -v 2> "mapper_logs/${sample}.log"

Parámetros relevantes: 

 -e: elimina secuencias con caracteres no estándar
 -h: colapsa lecturas idénticas
 -m: elimina adaptadores
 -l 18: longitud mínima
 -o 4: nombre de los archivos de salida

Herramienta: miRDeep2 mapper.pl (v2.0.1.2)

5.	Preparación para miRDeep2

Se agruparon los archivos colapsados y alineados para el análisis conjunto:

cat 04_mapper/*.fa  > 05_mirdeep/reads_collapsed.fa
cat 04_mapper/*.arf > 05_mirdeep/reads_vs_genome.arf

6.	Identificación y cuantificación de miARN (miRDeep2.pl)

Se ejecutó el pipeline principal de miRDeep2, que predice y cuantifica miARNs conocidos y potencialmente nuevos. Para cada muestra se emplearon archivos .fa y .arf y secuencias de referencia de miRBase (maduras y precursores)

miRDeep2.pl "$FA" "$GENOME" "$ARF" \
  "$MIRBASE_DIR/mature_${SPECIES}_fix.fa" \
  none \
  "$MIRBASE_DIR/hairpin_${SPECIES}_fix.fa" \
  -d -c -t hsa -v 2> "${SAMPLE}_mirdeep2.log"
  LAST=$(ls -td result_* | head -1)
  mv "$LAST" "result_${SAMPLE}"
done

-d: genera archivos de depuración
-c: genera archivos de resultados extendidos
-t: especifica la especie (hsa para Homo sapiens)

Herramienta: miRDeep2 v2.0.1.2

Cada resultado se almacenó como result_<muestra> dentro del directorio 05_mirdeep/results.

