# GA5 answers - Abigail Holton

------------------------------------------------------------------------

## Part A: Your project dir, Git repo, and protocol

### 1: Fill in the top of the protocol

### 2: Markdown formatting

### 3: Create subdirs, and copy the scripts

```bash
mkdir -p scripts/ data/ref/ results/
```

### 4: Initialize a Git repository, and commit your first files

### 5: Download the reference transcriptome

Code to count the number of transcripts:

```bash
zcat data/ref/GCF_016801865.2_TS_CPP_V2_rna.fna.gz | grep -c ">"
```

The output of the command was:

```
28624
```

*Answer*: The number of transcripts in the file: 28,624

------------------------------------------------------------------------

## Part B: Read trimming

### 6: Run TrimGalore

***Answer*: Whether all batch jobs succeeded, and how you checked that
(and any problems you encountered, and how you solved them):** All batch jobs succeeded. Jobs were monitored with `squeue -u aeholton2005 -l`, and success of all batch jobs was confirmed with `grep -c "Successfully finished" slurm-trimgalore-*.out` with all files including this phrase. 

------------------------------------------------------------------------

## Part C: Transcript quantification with Salmon

### 7: Create the Salmon index

Code to check the size of the index dir:

```bash
du -sh data/ref/salmon_index
```

The output of the command was:

```
1.1G    data/ref/salmon_index
```

***Answer*: How long it took to create the index, and how you found out:** It took roughly 30 seconds to create the index, which I found out by monitoring the batch job with `squeue -u aeholton2005 -l`. I also used `head -n 5 slurm-salmon_index*.out` and `tail -n 5 slurm-salmon_index*.out` to view when the script started and when it was successfully completed.

***Answer*: The size of the index dir:** 1.1G

### 8: Run Salmon for all samples

### 9: The transcripts with the most reads

***Answer*: What the `NumReads` column contains:** The `NumReads` column provides an estimate of the number of sequencing reads that originates from a single transcript as some reads can align equally with differen transcripts. 

```bash
sort -k5,5nr results/salmon_quant/S63/quant.sf | head -n3
```

The output of the command was:

```
XM_039591534.2  6566    6315.086        14518.656821    24004.889
XM_039573735.1  6445    6194.086        5342.073324     8663.254
XM_039573733.1  6591    6340.086        2876.480244     4774.749
```

### 10: The gene of the top transcript

```bash
zcat data/ref/GCF_016801865.2_TS_CPP_V2_rna.fna.gz | grep ">XM_039591534.2"
```

The output of the command was:

```
>XM_039591534.2 PREDICTED: Culex pipiens pallens vitellogenin-A1-like (LOC120426760), mRNA
```

***Answer*: The gene that the top transcript belongs to:** vitellogenin-A1-like gene

### 11: Mapping rates

```bash
grep "mapping rate:*" results/salmon_quant/S*/logs/salmon_quant.log | sort -k3
```

The output of the command was:

```
results/salmon_quant/S83/logs/salmon_quant.log:mapping rate: 76.1504%
results/salmon_quant/S85/logs/salmon_quant.log:mapping rate: 76.9540%
results/salmon_quant/S74/logs/salmon_quant.log:mapping rate: 78.2753%
results/salmon_quant/S82/logs/salmon_quant.log:mapping rate: 78.7291%
results/salmon_quant/S68/logs/salmon_quant.log:mapping rate: 78.8070%
results/salmon_quant/S65/logs/salmon_quant.log:mapping rate: 79.1055%
results/salmon_quant/S71/logs/salmon_quant.log:mapping rate: 79.9047%
results/salmon_quant/S75/logs/salmon_quant.log:mapping rate: 80.3915%
results/salmon_quant/S79/logs/salmon_quant.log:mapping rate: 80.4061%
results/salmon_quant/S64/logs/salmon_quant.log:mapping rate: 81.9152%
results/salmon_quant/S66/logs/salmon_quant.log:mapping rate: 81.9350%
results/salmon_quant/S80/logs/salmon_quant.log:mapping rate: 82.1721%
results/salmon_quant/S69/logs/salmon_quant.log:mapping rate: 82.1824%
results/salmon_quant/S70/logs/salmon_quant.log:mapping rate: 82.3060%
results/salmon_quant/S77/logs/salmon_quant.log:mapping rate: 82.3281%
results/salmon_quant/S76/logs/salmon_quant.log:mapping rate: 82.3396%
results/salmon_quant/S63/logs/salmon_quant.log:mapping rate: 82.4560%
results/salmon_quant/S67/logs/salmon_quant.log:mapping rate: 83.0459%
results/salmon_quant/S86/logs/salmon_quant.log:mapping rate: 83.0532%
results/salmon_quant/S84/logs/salmon_quant.log:mapping rate: 83.2207%
results/salmon_quant/S81/logs/salmon_quant.log:mapping rate: 83.6795%
results/salmon_quant/S78/logs/salmon_quant.log:mapping rate: 84.2490%
```

***Answer*: The sample with the lowest mapping rate:** S83

------------------------------------------------------------------------

## Part D: Summarizing all steps with MultiQC

### 12: Update your MultiQC script

### 13: Run MultiQC

***Answer*: For each program, the number of reports that MultiQC found, and why it found this number:** MultiQC found 22 reports for Salmon and 44 reports for TrimGalore and FASTQC. Salmon uses paired-end data, where R1 and R2 files for a sample are processed together. Conversely, TrimGalore and FASTQC run on one read, or one FASTQ file, not on paired-end samples together. This can be seen in how `results/salmon_quant` is organized by sample (S63, S64, etc.) while `results/trimgalore` has reports for each read (S63_R1, S63_R2, S64_R1, etc.).

### 14: The General Statistics table

***Answer*: The sample with the lowest Salmon mapping rate, and whether this matches your answer to question 11:** According to the MultiQC report, S83 has the lowest Salmon mapping rate (76.2%), which is the same as what was found in question 11 (76.15%). 

### 15: Adapter lengths

***Answer*: The most common adapter lengths, whether this surprised you, and why you think these lengths are the most common:** The most common adapter lengths are between 1-5bp. This does not suprise me, since I would think that these lengths are most common as a result of overlap between adapter sequences and reads. Random overlap of nucleotides in the end of a read and the beginning of the adapter will be trimmed by TrimGalore, indicating the trimmed adapter lengths. 

### 16: Ask Claude about the MultiQC report

***Answer*: What you asked, and what you learned from Claude's answer (in your own words):** I asked about the cause of a minor secondary spike in FastQC sequence duplication levels. Claude responded that this often results from a smaller set of abundant sequences, depending on the sequencing data. For RNA-seq, this is to be expected, as duplication of highly expressed genes is common and usually inconsequential. For DNA-seq, it is likely this secondary spike is the result of PCR over-amplification, which can happen when too many cycles are run. 

------------------------------------------------------------------------

## Part E: Improving the workflow with genAI

### 17: Claude's review of your protocol and scripts

***Answer*: At least three of Claude's suggestions, and whether you agree or disagree with each one, and why:**

1. **Use "Set file paths" variables throughout:** I had not noticed that I did not use all the variables defined here. I agree with the suggestion to incorporate these variables in the `protocol.md` so that variables are used consistently and can be easily tracked. 
2. **`multiqc.sh` resources:** One suggestion pointed out that the no `--time` or ``--cpus-per-task`` lines were provided for the `multiqc.sh` script compared to the other scripts. I agree that it would be beneficial to include these, though their values would depend on the jobs being run. 
3. **Log dirs:** Claude recommends having the script automatically `mv` its log to `"$outdir"/logs` at the end of the script. However, I think that it is better to manually move these files since it is easier to remove logs of failed jobs this way. 


### 18: Address a suggestion

*Answer*: Which suggestion(s) you addressed, and what you changed:

1. I incorporated the first suggestion into my `protocols.md` file by making sure all variables are used throughout. I checked steps 1-3 to make sure variables were used and changed code where variables could be incorporated. 
2. Other issues I addressed had to do with copy-pasting issues or typos when repeating code or transfering the output of a command that was printed to the terminal to a separated file. 

------------------------------------------------------------------------

## Part F: Publish your repo on GitHub

### 19: Create a GitHub repository and push

### 20: Get a (test) DOI with Zenodo

*Answer*: Your Zenodo Sandbox DOI:
