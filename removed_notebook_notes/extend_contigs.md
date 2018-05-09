
Install FASTX-Toolkit using Conda
conda install -c biobuilds fastx-toolkit=0.0.14

Convert FASTQ to FASTA
# fastx-toolkit is not pair-aware, use seqtk instead
fastq_to_fasta -v -i infile.fastq -o outfile.fasta
gunzip -c infile.fq.qc.gz | fastq_to_fasta -v -Q33 -o outfile.qc.fasta

Use AlignGraph
AlignGraph --read1 reads_1.fa --read2 reads_2.fa --contig contigs.fa --genome genome.fa --distanceLow 100 --distanceHigh 1500 --extendedContig extendedContigs.fa --remainingContig remainingContigs.fa

