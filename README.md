# Read Mapping and SAM Analysis Using Real Sequencing Data
This repository demonstrates a complete bioinformatics workflow for processing real Illumina sequencing data from the Long-Term Evolution Experiment (LTEE) Escherichia coli strain SRR2584863. The project covers sequence retrieval, FASTA/FASTQ manipulation, read mapping using BWA, SAM/BAM analysis with SAMtools, interpretation of FLAG values and CIGAR strings, variant calling with BCFtools, and SNP/indel detection. The workflow follows the practical command-line examples for learning genome analysis and read mapping.

## Overview

This project demonstrates:

- FASTA and FASTQ processing
- Sequence counting
- Header extraction using grep
- Text processing using sed
- Read mapping using BWA
- SAM/BAM file manipulation
- FLAG interpretation
- CIGAR string analysis
- SNP detection
- Variant calling
- Basic structural variant analysis

## Software Requirements

```bash
conda install -c bioconda sra-tools
conda install -c bioconda seqtk
conda install -c bioconda bwa
conda install -c bioconda samtools
conda install -c bioconda bcftools
```

## Step 1: Download Sequencing Data
Download the LTEE E. coli sequencing dataset.
```bash
prefetch SRR2584863
fasterq-dump --split-files SRR2584863
```
Files generated:

```
SRR2584863_1.fastq
SRR2584863_2.fastq
```

