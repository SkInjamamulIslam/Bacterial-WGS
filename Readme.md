# Bacterial Genome Assembly Pipeline (Short + Long Reads)

This repository contains a complete pipeline for assembling a **bacterial genome** using both **Illumina short reads** and **Nanopore long reads**. It walks you through downloading real sequencing data, quality control, trimming, long-read assembly, polishing, genome annotation, and quality evaluation.

---

## Tools Used in the Pipeline

- [FastQC](https://anaconda.org/bioconda/fastqc) – Quality check of raw reads  
- [MultiQC](https://github.com/MultiQC/MultiQC) – Summary reports of QC  
- [fastp](https://github.com/OpenGene/fastp) – Trimming short reads  
- [Filtlong](https://anaconda.org/bioconda/filtlong) – Filtering long reads  
- [Flye](https://github.com/mikolmogorov/Flye) – Long-read assembly  
- [Bandage](https://rrwick.github.io/Bandage/) – Assembly graph visualization  
- [bwa-mem2](https://anaconda.org/bioconda/bwa-mem2) – Mapping short reads  
- [Polypolish](https://anaconda.org/bioconda/polypolish) – Assembly polishing  
- [BUSCO](https://busco.ezlab.org/busco_userguide.html) – Genome completeness check  
- [Prokka](https://github.com/tseemann/prokka) – Genome annotation

---

## Step-by-Step Usage Guide

### 1. Prerequisites

Ensure you are using **Linux or macOS**.  
Install [Miniconda](https://docs.conda.io/en/latest/miniconda.html) if not already installed (recommended for managing environments).

### 2. Install Required Tools

Use the following Conda commands:

```bash
# Create a new environment
conda create -n genome-assembly -y
conda activate genome-assembly

# Install dependencies
conda install -c bioconda fastqc multiqc fastp filtlong flye bandage bwa-mem2 polypolish busco prokka
```

> ❗ **Note:** Bandage is a GUI tool. You can install it separately if needed via a system package manager or manually.

---

## 3. Retrieving the sequencing reads

```bash
# Download the long reads
wget -q https://zenodo.org/record/10669812/files/DRR187567.fastq.bz2 && bzcat DRR187567.fastq.bz2 > long_reads.fastq
```

```bash
# Download the paired end short reads
wget -q https://zenodo.org/record/10669812/files/DRR187559_1.fastqsanger.bz2 && bzcat DRR187559_1.fastqsanger.bz2 > illumina_1.fastq

wget -q https://zenodo.org/record/10669812/files/DRR187559_2.fastqsanger.bz2 && bzcat DRR187559_2.fastqsanger.bz2 > illumina_2.fastq
```



---

## 4. Quality control of raw reads and trimming

```bash
fastqc -t 3 *.fastq 2> fastqc.log && multiqc .
```

### 4.1 Trim the short reads 

```bash
fastp -i illumina_1.fastq -I illumina_2.fastq -f 15 -F 15 -t 2 -T 2 -w 4 -o illumina_1.trimmed.fastq -O illumina_2.trimmed.fastq
```

### 4.2 Trim the long reads

```bash
filtlong --min_length 1000 -1 illumina_1.trimmed.fastq -2 illumina_2.trimmed.fastq long_reads.fastq > long_reads.filtered.fastq
```

---

## 5. Assembly of long reads

```bash
flye --nano-corr long_reads.filtered.fastq -t 4 -o flye_Saureus
```

---

## 6. Visualize the assembly graph

```bash
Bandage image flye_Saureus/assembly_graph.gfa flye_Saureus/assembly_graph.png --width 800 --height 800
```

---

## 7. Polishing the assembly with short reads

### 7.1 BWA-MEM2 Indexing

```bash
bwa-mem2 index flye_Saureus/assembly.fasta 2> polishing_indexing.log
```

### 7.2 BWA-MEM2 Mapping

```bash
bwa-mem2 mem -t 4 -o Saureus_polishing_map.sam flye_Saureus/assembly.fasta illumina_1.trimmed.fastq illumina_2.trimmed.fastq 2> polishing_mapping.log
```

### 7.3 Polypolish Polishing

```bash
polypolish polish flye_Saureus/assembly.fasta Saureus_polishing_map.sam > Saureus_polished_assembly.fasta
```

---

## 8. Evaluating the assembly

```bash
busco -i Saureus_polished_assembly.fasta -l bacteria -o Saureus_BUSCO -m geno -c 4 -f > busco.log
```

---

## 9. Annotation of the assembled genome

```bash
prokka Saureus_polished_assembly.fasta --genus Staphylococcus --species aureus --cpus 4 --outdir Saureus_PROKKA --force 2> prokka.log 
```


This script performs the following:

1. Downloads short and long reads from Zenodo
2. Performs quality control and trimming
3. Assembles the genome using Flye
4. Visualizes the assembly graph
5. Polishes the assembly using BWA and Polypolish
6. Evaluates completeness using BUSCO
7. Annotates the genome using Prokka

---

## 📁 Output Structure

After successful execution, you will find:

| Folder/File                        | Description                               |
|-----------------------------------|-------------------------------------------|
| `flye_Saureus/`                   | Assembly results from Flye                |
| `Saureus_polishing_map.sam`       | Mapping of reads for polishing            |
| `Saureus_polished_assembly.fasta` | Final polished genome                     |
| `Saureus_BUSCO/`                  | BUSCO genome quality report               |
| `Saureus_PROKKA/`                 | Genome annotation results from Prokka     |
| `fastqc.log`, `prokka.log`, etc.  | Logs for troubleshooting                  |

---

## 📷 Visualizing Assembly Graph

The pipeline generates a GFA file and visual graph:

- View it in the generated `flye_Saureus/assembly_graph.png`
- Or open the GFA file manually with `Bandage`

---

## Reference Dataset Used

The pipeline uses real data hosted on [Zenodo](https://zenodo.org/record/10669812):

- Long-read (Nanopore): `DRR187567`
- Short-read (Illumina): `DRR187559`

---

## Citation

If you use this pipeline in your work, please consider citing the tools used, especially:

- Wick RR et al. Polypolish: Short-read polishing of long-read bacterial genome assemblies.
- Seemann T. Prokka: rapid prokaryotic genome annotation.
- Simão FA et al. BUSCO: assessing genome assembly and annotation completeness.

---

## License

This project is licensed under the [MIT License](LICENSE).

---

## Acknowledgements

This pipeline was built to help researchers and students understand bacterial genome assembly using real data and widely adopted open-source tools.


