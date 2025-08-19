# Genome Assembly and Annotation Pipeline (Short-Read)

## Introduction  
This guide walks through a standard short-read (Illumina) genome assembly and annotation pipeline. Each step includes a brief note explaining **why** it is required.

---

## Step 1 – System Update and Core Dependencies  
**Why this step:** Updating the system ensures compatibility and prevents installation errors.

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y build-essential wget unzip git python3 python3-pip
```

---

## Step 2 – Download Raw FASTQ Data (SRA)  
**Why this step:** Public raw reads can be retrieved from NCBI using the SRA Toolkit.

```bash
# Install SRA Toolkit
cd ~
wget https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz
tar -xvzf sratoolkit.current-ubuntu64.tar.gz
export PATH=$PATH:~/sratoolkit.*-ubuntu64/bin

# Download FASTQ files
fastq-dump --split-files SRRXXXXXXX
```

---

## Step 3 – Quality Control of Raw Reads  
**Why this step:** Detect low-quality regions and adapter contamination before assembly.

```bash
# Install FastQC
sudo apt install -y fastqc

# Run FastQC
mkdir fastqc_reports
fastqc SRRXXXXXXX_1.fastq SRRXXXXXXX_2.fastq -o fastqc_reports

# (Optional) summarize with MultiQC
pip3 install multiqc
multiqc fastqc_reports
```

---

## Step 4 – Trim Low-Quality Bases and Adapters  
**Why this step:** Trimming improves assembly efficiency and accuracy.

```bash
# Install Trim Galore
pip3 install trim-galore

# Run Trimming
trim_galore --paired --quality 20 --length 30 --cores 4 SRRXXXXXXX_1.fastq SRRXXXXXXX_2.fastq
```

---

## Step 5 – Quality Check of Trimmed Reads  
**Why this step:** Confirm that the trimming step resolved quality issues.

```bash
fastqc SRRXXXXXXX_1_val_1.fq SRRXXXXXXX_2_val_2.fq -o fastqc_reports/trimmed
multiqc fastqc_reports/trimmed
```

---

## Step 6 – de novo Genome Assembly with SPAdes  
**Why this step:** Assemble cleaned reads into genome contigs.

```bash
# Install SPAdes
sudo apt install -y spades

# Run Assembly
spades.py -1 SRRXXXXXXX_1_val_1.fq -2 SRRXXXXXXX_2_val_2.fq -o spades_output --threads 8 --memory 32
```

---

## Step 7 – Evaluate Assembly Quality (QUAST)  
**Why this step:** Evaluate the quality of the draft genome assembly.

```bash
# Install QUAST
sudo apt install -y quast

# Run QUAST
quast spades_output/contigs.fasta -o quast_report
```

---

## Step 8 – Genome Annotation (Prokka)  
**Why this step:** Annotate the genome to identify genes and functional elements.

```bash
# Install Prokka
sudo apt install -y prokka

# Run Annotation
prokka --outdir annotation_results --prefix sample1 spades_output/contigs.fasta
```

---

## Step 9 – Genome Completeness Assessment (BUSCO)  
**Why this step:** Estimate completeness by checking for conserved single-copy genes.

```bash
# Install BUSCO and dependencies
sudo apt install -y busco augustus

# Download lineage dataset
busco_download.py --lineage bacteria --output bacteria_odb10

# Run BUSCO
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
