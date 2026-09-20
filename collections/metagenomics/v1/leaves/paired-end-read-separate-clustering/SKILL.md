---
name: paired-end-read-separate-clustering
description: Use when processing paired-end 16S FASTQ reads with highly overlapping
  mates where quality drops in the overlap region make read merging lossy, and you
  want annotated OTUs without a merging step.
when_to_use_negative:
- Your analysis requires merged/consensus full-length amplicon sequences or ASV inference
  (e.g. QIIME2-DADA2 or deblur workflows) — this approach entirely avoids read merging
  rather than performing it.
- Input is already a merged-read or single-end dataset, or a precomputed feature/OTU
  table — the separate-clustering scheme depends on intact R1/R2 mate pairs.
- Reads are not cropped/filtered appropriately (e.g. quality collapses before 100
  nt remain) — Trimmomatic's MINLEN 100 cutoff would discard the reads entirely.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: CD-HIT-OTU-Miseq
  role: 'Source of the separate-clustering scheme: cd-hit-est clusters R1 and R2 independently
    at 99%, and non-matching cluster voting between the two clusterizations detects
    chimeric pairs without merging'
- name: CD-HIT
  role: Provides the cd-hit-est clustering utility invoked on each mate file separately
- name: CDSnake
  role: Snakemake pipeline that wraps the consecutive launch of the CD-HIT-OTU-Miseq
    utilities into a one-command, reproducible run
  repo: github.com/ydkondratenko/cdsnake
- name: Trimmomatic
  role: Crops R1/R2 to the good-quality prefix (p and q parameters) and discards cropped
    reads shorter than 100 nucleotides (MINLEN)
- name: Snakemake
  role: Workflow engine that executes the Snakefile control loop over input samples
    with default settings via the snakemake command
  repo: github.com/ydkondratenko/cdsnake
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:1a24b99c52e5808a6c8e3c68e3a2e8d53a7d86f970f241f2b037172c24ef4585
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - CD-HIT-OTU-Miseq
  - CDSnake
  - Snakemake
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# paired-end-read-separate-clustering

## Summary

Cluster R1 and R2 reads of paired-end 16S amplicon data separately (never merging or concatenating mates), following the CD-HIT-OTU-Miseq scheme, so chimeric read pairs can be detected by discordant cluster voting and OTUs retrieved with less read discarding than merging-based pipelines.

## When to use

Use when processing paired-end 16S FASTQ reads with highly overlapping mates where quality drops in the overlap region make read merging lossy, and you want annotated OTUs without a merging step.

## When NOT to use

- Your analysis requires merged/consensus full-length amplicon sequences or ASV inference (e.g. QIIME2-DADA2 or deblur workflows) — this approach entirely avoids read merging rather than performing it.
- Input is already a merged-read or single-end dataset, or a precomputed feature/OTU table — the separate-clustering scheme depends on intact R1/R2 mate pairs.
- Reads are not cropped/filtered appropriately (e.g. quality collapses before 100 nt remain) — Trimmomatic's MINLEN 100 cutoff would discard the reads entirely.

## Inputs

- Paired-end 16S reads as unzipped FASTQ files, forward/reverse named with '_R1_'/'_R2_' or ending '_1'/'_2' before '.fastq'
- Annotated 16S reference database (e.g. Greengenes or SILVA) in FASTA form, reformatted as required by CD-HIT-OTU-Miseq installation step 4
- Edited Snakefile with INPUT_FOLDER, DATABASE, CDHIT, TRIMMOMATIC, p, q, THREADS, MINLEN parameters set

## Outputs

- 99% identity clusterings of R1 reads and of R2 reads (cd-hit-est output)
- Chimera-filtered read pairs (pairs whose mates vote for non-matching large clusters discarded)
- 97% similarity annotated OTU clusters of pooled samples
- OTU count table for all pooled samples written to pooled/OTU.txt

## How to apply

Crop R1/R2 reads to their 'good' quality parts (defaults 200 bp for R1 and 180 bp for R2, quality generally above 25 with no sudden drops) and quality-filter them with Trimmomatic using configured THREADS and MINLEN settings that discard cropped reads shorter than 100 nucleotides. Cluster filtered R1 reads separately at 99% identity with cd-hit-est, then repeat the clustering with identical parameters for R2 reads, so read pairs are never merged or concatenated. Detect chimeric read pairs whose mates vote for non-matching sufficiently large clusters across the two 99% clusterizations and discard those reads — this replaces merging-based chimera removal. Finally cut the annotated 16S database (e.g. Greengenes or SILVA FASTA) into read-length fragments and cluster the remaining reads together with database fragments at 97% similarity to yield annotated OTUs at species or close taxonomic levels. Orchestrate all steps with the CDSnake Snakefile so a single snakemake command reproduces the full workflow.

## Related tools

- **CD-HIT-OTU-Miseq** (Source of the separate-clustering scheme: cd-hit-est clusters R1 and R2 independently at 99%, and non-matching cluster voting between the two clusterizations detects chimeric pairs without merging)
- **CD-HIT** (Provides the cd-hit-est clustering utility invoked on each mate file separately)
- **CDSnake** (Snakemake pipeline that wraps the consecutive launch of the CD-HIT-OTU-Miseq utilities into a one-command, reproducible run) — github.com/ydkondratenko/cdsnake
- **Trimmomatic** (Crops R1/R2 to the good-quality prefix (p and q parameters) and discards cropped reads shorter than 100 nucleotides (MINLEN))
- **Snakemake** (Workflow engine that executes the Snakefile control loop over input samples with default settings via the snakemake command) — github.com/ydkondratenko/cdsnake

## Examples

```
cd $INPUT_FOLDER && snakemake --cores 8
```

## Evaluation signals

- pooled/OTU.txt exists in the input folder after the run and contains per-sample OTU counts for all pooled samples.
- On mock communities, the reported OTU counts per sample are comparable to the number of strains in the analyzed mock community (e.g. closer to the real strain count than DADA2/deblur on Balanced and HMP datasets).
- No read-merging step appears in the executed workflow; each mate file is clustered independently at 99% with identical cd-hit-est parameters.
- Total read counts retained are higher than merging-based pipelines, reflecting that a smaller portion of reads is discarded during OTU retrieval.
- Chimera filtering removed read pairs whose mates voted for non-matching sufficiently large clusters across the two 99% clusterizations.

## Limitations

- CDSnake is generally slower than DADA2 and deblur pipelines.
- Software versions for DADA2, deblur, CD-HIT-OTU-MiSeq, and Snakemake are not stated, complicating exact reproduction.
- The p and q crop-length parameters (default 200/180 bp) must be tuned per dataset using quality reports (e.g. fastqc); a sudden quality drop earlier than expected requires manual adjustment.
- Availability/accessions for the mock benchmark datasets are not provided, limiting direct benchmarking.

## Evidence

- [methods] Chimeric reads detection is possible when reads from singe pair vote for non-matching clusters of two clusterizations (and these clusters are large enough).: "Chimeric reads detection is possible when reads from singe pair vote for non-matching clusters of two clusterizations (and these clusters are large enough)."
- [methods] Filtered sequences are clustered at 99% using cd-hit-est utility to discard chimeric reads.: "Filtered sequences are clustered at 99% using cd-hit-est utility to discard chimeric reads."
- [abstract] Recently CD-HIT-OTU-Miseq was presented as a new approach for 16S analysis using the paired-end reads, it completely avoids the reads merging process: "Recently CD-HIT-OTU-Miseq was presented as a new approach for 16S analysis using the paired-end reads, it completely avoids the reads merging process"
- [readme] Results of clustering for pooled samples will be stored in folder “pooled” in input folder. OTU table for all samples will be saved in file “OTU.txt”.: "Results of clustering for pooled samples will be stored in folder “pooled” in input folder. OTU table for all samples will be saved in file “OTU.txt” in “pooled” folder."
- [readme] Trimmomatic will try to use THREADS number of threads and will discard cropped reads if they become shorter than 100 nucleotides.: "Trimmomatic will try to use THREADS number of threads and will discard cropped reads if they become shorter than 100 nucleotides."
