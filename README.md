# Synalpheus-GO-Analysis-for-Differential-expression-transcripts
Gene Ontology (GO) Annotation in S. brooksi and S. elizabethae

This project aims to annotate transcriptomes from eusocial sponge shrimp species using InterProScan and perform downstream Gene Ontology (GO) enrichment analysis on differentially expressed genes (DEGs). Previous DEseq2 analysis was performed on Galaxy to acquire DEGs for input.

Transcriptome sequences were analyzed using InterProScan with GO term assignment. 

Following GO annotation, enrichment analysis was performed using methods adapted from: [Enrichment Analysis for Non-model Organisms](https://github.com/dadrasarmin/enrichment_analysis_for_non_model_organism?tab=readme-ov-file) by dadrasarmin

The goal is to identify biological processes, molecular functions, and cellular components associated with caste differentiation and eusocial behavior in Synalpheus shrimp.

Project Summary

This workflow:

Splits large transcriptome FASTA files into smaller chunks
Runs InterProScan on OSC using SLURM job arrays to assigns GO terms
Generates annotation tables for downstream analysis
Performs GO enrichment analysis on DEGs using clusterProfiler in R

Species analyzed: S. brooksi and S. elizabethae

Tools Used
Tool	Purpose
InterProScan	GO annotation
seqkit	FASTA splitting
GOQuick	GO ID to GO term matching
R	GO enrichment analysis
SLURM	HPC job scheduling
OSC HPC	Computational platform

Workflow
1. Install InterProScan

Download InterProScan:

wget https://ftp.ebi.ac.uk/pub/software/unix/iprscan/5/5.77-108.0/interproscan-5.77-108.0-64-bit.tar.gz

Extract files:

tar -xvzf interproscan-5.77-108.0-64-bit.tar.gz

cd interproscan-5.77-108.0

chmod +x interproscan.sh
2. Split Transcriptome FASTA Files

Large transcriptomes were divided into smaller FASTA chunks to enable parallel processing with SLURM array jobs.

S. brooksi
./seqkit split \
    -s 2000 \
    trinity_brooksi_full.Trinity.fasta \
    -O split_files/
S. elizabethae
./seqkit split \
    -s 2000 \
    S.elizabethae_transcriptome_assembly.fasta \
    -O S_eli_split_files/

Each split file contains 2000 transcript sequences.

3. InterProScan SLURM Workflow
Example: S. brooksi
#!/bin/bash
#SBATCH --account=PDNU0016
#SBATCH --job-name=interscan_S.brooksi
#SBATCH --time=7-00:00:00
#SBATCH --ntasks-per-node=1
#SBATCH --array=1-300
#SBATCH --cpus-per-task=4
#SBATCH --mail-type=ALL
#SBATCH --output=%x.%j.log

# Temporary directory
export TMPDIR=/fs/scratch/PDNU0016/duongm1/tmp

# Go to InterProScan directory
cd /fs/scratch/PDNU0016/duongm1/interproscan-5.77-108.0

# Get FASTA file for this array task
FILE=$(ls /path/to/split_files/*.fasta | sed -n "${SLURM_ARRAY_TASK_ID}p")

# Run InterProScan
./interproscan.sh \
    -i $FILE \
    -t n \
     -goterms \
    -f tsv \
    -cpu 4 \
    -T $TMPDIR \
    -b /path/to/interproscan_output/$(basename $FILE .fasta)

4. GO enrichment Analysis
Followed methods adapted from: [Enrichment Analysis for Non-model Organisms](https://github.com/dadrasarmin/enrichment_analysis_for_non_model_organism?tab=readme-ov-file) by dadrasarmin
R file of the analysis included in this repository.

