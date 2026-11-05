# RNA-seq analysis of _Culex pipiens_

- Author: Abigail Holton
- Date created: 2026-11-05
- Date last modified: 2026-11-05
- This was run at the Pitzer cluster of the Ohio Supercomputer Center (<https://www.osc.edu>)
- OSC working dir: `/fs/ess/PAS3493/people/aeholton2005/GA5`
- GitHub repository: https://github.com/aholton104/PLNTPTH5006-GA5

## Prerequisites

- **Input files**:
  - Paired-end RNA-seq reads are in `../garrigos/fastq`

- **Software**:
  - The scripts run programs using Apptainer containers,
    with links contained inside the scripts.
  - If running at OSC, no software installations are needed.
  - If you're not at OSC, you need to have Apptainer and Slurm installed.

- Run all code from the working dir listed above.
  You don't need to create the output dirs, because the scripts will do so.

- Each script writes a Slurm log file (`slurm-<script>-<jobID>.out`) to the working dir.
  After the jobs of a step have finished, check these logs for errors,
  then move them into a `logs/` dir within that step's output dir.

## Set up: Download the reference transcriptome FASTA file from NCBI

```bash
# Downloaded on 2026-11-05 at NCBI FTP
URL=https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/016/801/865/GCF_016801865.2_TS_CPP_V2/GCF_016801865.2_TS_CPP_V2_rna.fna.gz
wget -P data/ref "$URL"
```

## Set up: Set file paths

```bash
# Inputs:
fastq_dir=../garrigos/fastq
ref_transcripts=data/ref

# Base output dir:
outdir=results
```

## Step 1: Read trimming with TrimGalore

`trimgalore.sh` runs TrimGalore on paired-end FASTQ samples of 1 sample and takes 3 arguments: FASTQ R1 file, FASTQ R2 file, and the output dir. 

```bash
for fastq_r1 in "$fastq_dir"/*_R1.fastq.gz; do
  fastq_r2="${fastq_r1/_R1/_R2}"
  sbatch scripts/trimgalore.sh "$fastq_r1" "$fastq_r2" "$outdir"/trimgalore
done
```

## Step 2: Creating a transcriptome index with Salmon

`salmon_index.sh` runs Salmon to create index from `_rna.fna.gz` file and takes 2 arguments: `_rna.fna.gz` file and output dir. 

```bash
sbatch scripts/salmon_index.sh "$ref_transcripts"/GCF_016801865.2_TS_CPP_V2_rna.fna.gz "$ref_transcripts"/salmon_index
```

## Step 3: Pseudo-alignment and quantification with Salmon

`salmon_quant.sh` runs Salmon on trimmed FASTQ files and takes 4 arguments: FASTQ R1 file, FASTA R2 file, index dir, and output dir. 

```bash
for fastq_r1 in "$outdir"/trimgalore/*_R1.fastq.gz; do
	fastq_r2="${fastq_r1/_R1/_R2}"
	sbatch scripts/salmon_quant.sh "$fastq_r1" "$fastq_r2" "$ref_transcripts"/salmon_index "$outdir"/salmon_quant
done
```

## Step 4: Summarizing the results of all steps with MultiQC

`multiqc.sh` runs MultiQC on TrimGalore & Salmon results and takes 2 arguments: TrimGalore & Salmon results dir and output dir. 

```bash
sbatch scripts/multiqc.sh "$outdir"/ "$outdir"/multiqc
```