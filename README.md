# Genome Assembly and Annotation Pipeline (Short-Read, Conda-Friendly)

## Introduction  
This guide walks through a short-read (Illumina) genome assembly and annotation pipeline.  
All required tools are installed via **conda** to make the workflow easily reproducible and portable.

---

## Step 1 – Create Conda Environment and Install Core Tools  
**Why this step:** Using a clean environment avoids conflicts and ensures reproducibility.

```bash
conda create -n genome_assembly_env python=3.9 -y
conda activate genome_assembly_env
```

---

## Step 2 – Install and Run SRA Toolkit  
**Why this step:** Used to download raw FASTQ files from NCBI SRA.

```bash
conda install -c bioconda sra-tools -y

# Download FASTQ files
fastq-dump --split-files SRRXXXXXXX
```

---

## Step 3 – Quality Control of Raw Reads  
**Why this step:** Detect low-quality regions and adapter contamination.

```bash
conda install -c bioconda fastqc multiqc -y

# Run FastQC
mkdir fastqc_reports
fastqc SRRXXXXXXX_1.fastq SRRXXXXXXX_2.fastq -o fastqc_reports

# Summarize results
multiqc fastqc_reports
```

---

## Step 4 – Trim Low-Quality Bases and Adapters  
**Why this step:** Trimming improves assembly quality.

```bash
conda install -c bioconda trim-galore -y

# Run Trimming
trim_galore --paired --quality 20 --length 30 --cores 4 SRRXXXXXXX_1.fastq SRRXXXXXXX_2.fastq
```

---

## Step 5 – Quality Check of Trimmed Reads  
**Why this step:** Make sure trimming resolved quality issues.

```bash
fastqc SRRXXXXXXX_1_val_1.fq SRRXXXXXXX_2_val_2.fq -o fastqc_reports/trimmed
multiqc fastqc_reports/trimmed
```

---

## Step 6 – de novo Genome Assembly with SPAdes  
**Why this step:** Assemble cleaned reads into draft genome.

```bash
conda install -c bioconda spades -y

spades.py -1 SRRXXXXXXX_1_val_1.fq -2 SRRXXXXXXX_2_val_2.fq -o spades_output --threads 8
```

---

## Step 7 – Evaluate Assembly Quality (QUAST)  
**Why this step:** Evaluate the quality of the draft assembly.

```bash
conda install -c bioconda quast -y

quast spades_output/contigs.fasta -o quast_report
```

---

## Step 8 – Genome Annotation (Prokka)  
**Why this step:** Identify genomic features and assign functional annotations.

```bash
conda install -c bioconda prokka -y

prokka --outdir annotation_results --prefix sample1 spades_output/contigs.fasta
```

---

## Step 9 – Genome Completeness Assessment (BUSCO)  
**Why this step:** Check for the presence of conserved single-copy genes.

```bash
conda install -c bioconda busco -y
conda install -c bioconda augustus -y

busco_download.py --lineage bacteria --output bacteria_odb10

busco -i spades_output/contigs.fasta -l bacteria_odb10 -o busco_out -m genome --cpu 8
```

---

## ✅ Final Output Checklist

| Step               | Output                               |
|--------------------|--------------------------------------|
| Raw read QC        | fastqc_reports/                      |
| Trimmed reads      | *_val_1.fq / *_val_2.fq              |
| Assembly           | spades_output/contigs.fasta          |
| Assembly stats     | quast_report/report.txt              |
| Annotation         | annotation_results/                  |
| Completeness       | busco_out/short_summary.txt          |

---
