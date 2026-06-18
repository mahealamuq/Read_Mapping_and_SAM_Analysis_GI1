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

## Step 2: Convert FASTQ to FASTA

```bash
seqtk seq -A SRR2584863_1.fastq > SRR2584863_1.fsa
seqtk seq -A SRR2584863_2.fastq > SRR2584863_2.fsa
```

## Step 3: Count FASTA reads 

```bash
grep -c '^>' SRR2584863_1.fsa
grep -c '^>' SRR2584863_2.fsa
```

ALternative:

```bash
grep '^>' SRR2584863_1.fsa | wc -l
grep '^>' SRR2584863_2.fsa | wc -l
```

## Step 4: Extract FASTA Headers

```bash`
grep '^>' SRR2584863_1.fsa > SRR2584863_1_header.txt
```

## Step 5: Remove FASTA Header Symbol(>)

```bash
sed 's/>//' SRR2584863_1_header.txt > SRR2584863_1_header_no_arrow.txt
```

## Step 6: Bash Script for count reads in any fastq file

```bash
#!/bin/bash

if [ $# -ne 1 ]; then
    echo "Usage: $0 fasta_file"
    exit 1
fi

count=$(grep -c "^@" "$1")

echo "Number of sequences: $count"
```
Save as:

```
count_fasta_seq_no.sh
```

Run:

```bash
chmod +x count_fasta_seq_no.sh
./count_fasta_seq_no.sh SRR2584863_1.fsa
```
**OR**

Here's the Python equivalent of your Bash script:

```
#!/usr/bin/env python3

import sys

if len(sys.argv) != 2:
    print(f"Usage: {sys.argv[0]} fasta_file")
    sys.exit(1)

fasta_file = sys.argv[1]
count = 0

with open(fasta_file, "r") as f:
    for line in f:
        if line.startswith(">"):
            count += 1

print(f"Number of sequences: {count}")
```

Save as:

```
count_fasta.py
```
Run:

```
python3 count_fasta.py SRS014776.fsa
```
## Step 7: Download Reference Genome

REL606 is the ancestral E. coli strain used in the LTEE experiment.

```
wget "https://www.ncbi.nlm.nih.gov/sviewer/viewer.cgi?id=CP000819.1&db=nuccore&report=fasta&retmode=text" -O REL606.fasta
```

## Step 8: Build BWA Index

```
bwa index REL606.fasta
```
Generated files:

```
REL606.fasta.amb
REL606.fasta.ann
REL606.fasta.bwt
REL606.fasta.pac
REL606.fasta.sa
```

## Step 9: Read Mapping

```bash
bwa mem REL606.fasta \
SRR2584863_1.fastq \
SRR2584863_2.fastq \
> SRR2584863_REL606.sam
```
## Step 10: Convert SAM to BAM

```
samtools view -bS SRR2584863_REL606.sam > SRR2584863_REL606.bam
```

## Step 11: Sort BAM

```bash
samtools sort \
SRR2584863_REL606.bam \
-o SRR2584863_REL606.sorted.bam
```

## Step 12: Index BAM
```
samtools index SRR2584863_REL606.sorted.bam
```
## Step 13: Mapping Statistics

```bash
samtools flagstat SRR2584863_REL606.sorted.bam
```

**Visual Inspection of Read Alignments Using "SAMtools tview"**

samtools tview provides a rapid terminal-based method for validating read alignments and variant calls. By comparing aligned reads to the reference genome, potential SNPs, insertions, deletions, and coverage patterns can be visually inspected before downstream analysis. This step serves as an important quality-control procedure during genome resequencing and variant discovery workflows.

```bash
samtools tview SRR2584863_REL606.sorted.bam REL606.fasta
```

<img width="1899" height="979" alt="image" src="https://github.com/user-attachments/assets/f420548e-5e7e-447e-a067-778501250908" />


## Step 14: CIGAR String Analysis
View CIGAR strings:

```bash
cut -f6 SRR2584863_REL606.sam | head
```
Example output:
```
150M
19M131S
150M
20M130S
150M
22S128M
150M
*
```

Interpretation

| CIGAR   | Meaning           |
| ------- | ----------------- |
| 150M    | Perfect alignment |
| 19M131S | Soft clipping     |
| 20M130S | Soft clipping     |
| 22S128M | Soft clipping     |
| *       | Unmapped read     |

## Step 15: Detect Deletions

```
cut -f1,6 SRR2584863_REL606.sam | grep D | head
```

Example:

```
SRR2584863.169 17M1D133M
SRR2584863.252 147M1D3M
SRR2584863.325 106M1D44M
```
Count deletions:

```bash
cut -f1,6 SRR2584863_REL606.sam | grep D | wc
```

## Step 16: Detect Insertions
```bash
cut -f1,6 SRR2584863_REL606.sam | grep I | head
```
Count insertions:

```
cut -f1,6 SRR2584863_REL606.sam | grep I | wc
```

Result:

```
4889 reads contain insertions
```

## Step 17: Variant Calling

```
bcftools mpileup \
-f REL606.fasta \
SRR2584863_REL606.sorted.bam | \
bcftools call -mv -Ov \
-o SRR2584863_REL606_variants.vcf
```
## Step 18: View Variants

```
grep -v "^#" SRR2584863_REL606_variants.vcf | head
```

<img width="1911" height="329" alt="image" src="https://github.com/user-attachments/assets/f551061a-6dea-4805-9674-a8ccbd0f4251" />

Let's decode the first line:

```
CP000819.1    9972    .    T    G    225.417    .
```

The columns mean:

| Column | Value      | Meaning                |
| ------ | ---------- | ---------------------- |
| CHROM  | CP000819.1 | Reference chromosome   |
| POS    | 9972       | Position in genome     |
| ID     | .          | No variant ID assigned |
| REF    | T          | Reference base         |
| ALT    | G          | Variant base           |
| QUAL   | 225.417    | Confidence score       |
| FILTER | .          | Passed filtering       |

**Interpretation**

At position 9972:
```
Reference genome: T
Sample genome:    G
```

This is a single nucleotide variant (SNV).

**INFO field**

Immediately after FILTER you see:

```
DP=107
VDB=0.988197
MQ=60
AC=2
AN=2
```

Important values:

| Field | Meaning                |
| ----- | ---------------------- |
| DP    | Read depth             |
| MQ    | Mapping quality        |
| AC    | Alternate allele count |
| AN    | Total alleles          |

Example:

```
DP=107
```
means:

```
107 reads cover this position
```
which is very strong evidence.

**FORMAT field**

Near the end:

```
GT:PL
1/1:255,255,0
```
| GT  | Meaning   |
| --- | --------- |
| 0/0 | Reference |
| 0/1 | Mixed     |
| 1/1 | Variant   |

Your first SNP:

```
1/1
```

means:

```
100% of reads support the variant
```

