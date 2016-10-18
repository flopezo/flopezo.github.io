---
layout:     page
title:      Mapping reads from RNA-seq data to mitochondrial genome 
date:       2016-10-18
summary:    Test Post
categories: Test
---

These are notes for aligning reads from RNA-seq data to a mitochondrial genome using [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml), since mitochondrial DNA, in general, lacks introns, and therefore a splice-aware read aligner would not be necessary; other tools such as [BWA](https://github.com/lh3/bwa) or [STAR](https://github.com/alexdobin/STAR) may be used for this task, too. General information about read alignment can be found in this [review](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-016-0881-8) or [here](https://en.wikipedia.org/wiki/List_of_RNA-Seq_bioinformatics_tools#Alignment_Tools). In this example, I will use the mitochondrial genome of the German wasp (*Vespula germanica*). 

First, download `Bowtie2` and add the directory with the executables for Mac OS X/macOS to the system's [PATH environment variable](http://en.wikipedia.org/wiki/PATH_(variable)):

```bash
# Download Bowtie2
curl -L http://downloads.sourceforge.net/project/bowtie-bio/bowtie2/2.2.9/bowtie2-2.2.9-macos-x86_64.zip \
	> bowtie2-2.2.9-macos-x86_64.zip

# Unpack file
unzip -a bowtie2-2.2.9-macos-x86_64.zip

# Update PATH 
# Here I use the text editor vi
# This assumes that the file .bash_profile exists in the home directory
# .bash_profile is a hidden file. Show hidden files in Finder by
# using the triple combination of ‘shift‘ + ‘command‘ + ‘.‘
# Change into home directory 
cd ~
# Use vi to open file
vi .bash_profile

# Use vi's i command to begin inserting text
i

# Insert path to Bowtie2 executables for Mac OS X/macOS
export PATH=$PATH:/path/to/bowtie2-2.2.9

# Press escape key
esc

# Use vi's commands :wq to quit and save edits
:wq

# Confirm that Bowtie2 is on the system's PATH
which bowtie2
```

Second, download the mitochondrial genome of *Vespula germanica* as a FASTA file and index the genome. In this example, I employ `efetch` from the [NCBI's E-utilities](https://www.ncbi.nlm.nih.gov/books/NBK25500/) to download the FASTA file.

```bash
# Download FASTA file using the NCBI's efetch utility
curl "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?\
db=nuccore&id=KR703583&rettype=fasta&retmode=text" > vgermanica_mitogenome.fasta

# Build genome index
# Usage: bowtie2-build file.fasta index_name
bowtie2-build vgermanica_mitogenome.fasta vgermanica_mitogenome
```

Third, align reads. Reads in FASTQ format were previously trimmed using [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic). Here I am using paired-end (2 x 100 bp) Illumina reads from a species of the genus *Dolichovespula*; no mitochondrial genome of *Dolichovespula* is currently available on GenBank.

```bash
# Map reads
# Usage: bowtie2 -x index_name -1 reads_1.fq -2 reads_2.fq -S outfile.sam
bowtie2 -x vgermanica_mitogenome -1 ../reads_1_paired.fq.qc.gz -2 ../reads_2_paired.fq.qc.gz -S reads_x_vgermanica.bowtie.sam -p 2 --very-sensitive
```

The overall alignment rate was 0.34% or 484,347 reads.

Next, use [samtools](https://github.com/samtools/samtools) to convert the SAM file into a sorted BAM file.

```bash
# Convert SAM to BAM
# Usage: samtools view -bS file.sam > file.bam
samtools view -bS reads_x_vgermanica.bowtie.sam > reads_x_vgermanica.bowtie.bam

# Convert BAM file to a sorted BAM file
# .bam.bai (the index of the sorted BAM file)
# Usage: samtools sort file.bam -o file.sorted.bam
samtools sort reads_x_vgermanica.bowtie.bam -o reads_x_vgermanica.bowtie.sorted.bam

# Optional: Get only mapped reads from BAM file
$ Usage: samtools view -b -F 4 file.sorted.bam > file.sorted.mapped.bam
samtools view -b -F 4 reads_x_vgermanica.bowtie.sorted.bam > reads_x_vgermanica.bowtie.sorted.mapped.bam
```