---
name: microbial-read-retention-quantification
description: 'Use when benchmarking host-depletion tools for precision on non-human
  reads: you need to measure microbial read retention (reads_out/reads_in) from DWGSIM/PBSIM2-simulated
  Illumina and ONT reads against Hostile, HRRT, or masked Hostile indexes.'
when_to_use_negative:
- The goal is measuring human read depletion (host removal sensitivity) rather than
  microbial read retention/precision
- The input reads are real clinical metagenomic samples rather than simulated reads
  with known ground truth, since retention percentages require reads_in/reads_out
  accounting on known microbial content
- The sample is dominated by viral reads and only the bacterial-masked index (human-t2t-hla-argos985)
  is used — the virus/phage-masked index is required when retention of viral sequences
  is a priority
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Hostile
  role: Primary decontamination tool; cleans reads via Bowtie2 (short) or Minimap2
    (long) alignment and emits JSON logs used to compute retention
  repo: https://github.com/bede/hostile
- name: Bowtie2
  role: Alignment backend for Hostile short-read (paired Illumina) cleaning with --sensitive
    preset and -k 1
- name: Minimap2
  role: Alignment backend for Hostile long-read (ONT) cleaning with -x map-ont preset
    and secondary alignments disabled
- name: DWGSIM
  role: Simulates paired Illumina short reads from reference assemblies
- name: PBSIM2
  role: Simulates Oxford Nanopore long reads from reference assemblies
- name: sra-human-scrubber (HRRT)
  role: NCBI comparator tool whose microbial read retention is benchmarked against
    Hostile
  repo: https://github.com/ncbi/sra-human-scrubber
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:98480e324a171b286ac42cbb4df97f78198b0235b1b90f77334cbf3256384c04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bedtools
  - Bowtie2
  - DWGSIM
  - HRRT
  - Hostile
  - Minimap2
  - PBSIM2
  - Samtools
  - sra-human-scrubber (HRRT)
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# microbial-read-retention-quantification

## Summary

Quantifies what fraction of simulated microbial reads survive host-read decontamination with Hostile versus the NCBI Human Read Removal Tool (HRRT), on Illumina and ONT data, and evaluates masked indexes (e.g. human-t2t-hla-argos985-mycob140) for improving retention. Grounded in the Hostile paper's guarantee of ≥99.989% retention of simulated bacterial reads.

## When to use

Use when benchmarking host-depletion tools for precision on non-human reads: you need to measure microbial read retention (reads_out/reads_in) from DWGSIM/PBSIM2-simulated Illumina and ONT reads against Hostile, HRRT, or masked Hostile indexes.

## When NOT to use

- The goal is measuring human read depletion (host removal sensitivity) rather than microbial read retention/precision
- The input reads are real clinical metagenomic samples rather than simulated reads with known ground truth, since retention percentages require reads_in/reads_out accounting on known microbial content
- The sample is dominated by viral reads and only the bacterial-masked index (human-t2t-hla-argos985) is used — the virus/phage-masked index is required when retention of viral sequences is a priority

## Inputs

- Simulated paired Illumina FASTQ(.gz) reads from bacterial/mycobacterial assemblies (DWGSIM)
- Simulated ONT long-read FASTQ(.gz) from the same assemblies (PBSIM2)
- Reference genome assemblies (985 FDA-ARGOS bacterial genomes, BioProject 231221; 140 mycobacterial assemblies)
- Hostile alignment index (human-t2t-hla default or masked index such as human-t2t-hla-argos985-mycob140)
- HRRT/SRA human scrubber k-mer database

## Outputs

- Per-run Hostile JSON logs with reads_in, reads_out, reads_removed, reads_removed_proportion
- Retention comparison table (e.g. masked_retention_results.csv) of Hostile vs HRRT retention percentages on Illumina and ONT data
- Cleaned FASTQ files of retained microbial reads

## How to apply

Simulate paired Illumina short reads with DWGSIM and ONT long reads with PBSIM2 from reference assemblies (e.g. 985 FDA-ARGOS bacterial genomes from BioProject 231221, or 140 mycobacterial assemblies). Clean the reads with `hostile clean` — Bowtie2 mode with the --sensitive preset (-k 1, no secondary alignments) for Illumina pairs, Minimap2 mode with the -x map-ont preset and secondary alignments disabled for ONT — against the default human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51). Run HRRT (ncbi/sra-human-scrubber) on the same read sets as the comparator. Parse each Hostile JSON log for reads_in, reads_out and reads_removed, and compute retention as reads_out/reads_in; compare against expected values such as Hostile ONT bacterial retention of 99.98918% (891 reads removed) and mycobacterial Illumina retention of 99.99998%. For maximum retention, repeat with a masked index fetched via `hostile index fetch` (e.g. human-t2t-hla-argos985-mycob140), expecting ≥99.997% bacterial retention with ≤0.001% reduction in human read removal performance, and tabulate all values into a comparison CSV.

## Related tools

- **Hostile** (Primary decontamination tool; cleans reads via Bowtie2 (short) or Minimap2 (long) alignment and emits JSON logs used to compute retention) — https://github.com/bede/hostile
- **Bowtie2** (Alignment backend for Hostile short-read (paired Illumina) cleaning with --sensitive preset and -k 1)
- **Minimap2** (Alignment backend for Hostile long-read (ONT) cleaning with -x map-ont preset and secondary alignments disabled)
- **DWGSIM** (Simulates paired Illumina short reads from reference assemblies)
- **PBSIM2** (Simulates Oxford Nanopore long reads from reference assemblies)
- **sra-human-scrubber (HRRT)** (NCBI comparator tool whose microbial read retention is benchmarked against Hostile) — https://github.com/ncbi/sra-human-scrubber

## Examples

```
hostile clean --fastq1 reads_1.fastq.gz --fastq2 reads_2.fastq.gz --index human-t2t-hla-argos985-mycob140
```

## Evaluation signals

- Hostile ONT bacterial retention on FDA-ARGOS reads equals 99.98918% with exactly 891 reads removed, matching the reported value
- Simulated mycobacterial retention reproduces Hostile Illumina ~99.99998% and ONT ~99.99948%, exceeding the ≥99.989% abstract guarantee; HRRT removes ~15x more mycobacterial reads on Illumina
- Masked-index run shows bacterial Illumina retention of 99.99994%, exactly 251 ONT bacterial reads removed, and 100% retention of masked mycobacterial Illumina and ONT reads
- Human retention change from masking stays within reported bounds: Illumina 0.131738% → 0.131979% and ONT 0.038029% → 0.038069% (≤0.001% reduction in removal performance)
- Every Hostile clean run emits a parseable JSON log with consistent reads_in ≥ reads_out and reads_removed_proportion consistent with the computed retention percentage

## Limitations

- Retention figures are grounded in simulated reads; real samples with unknown microbial content cannot reproduce exact percentages
- Non-human contamination (e.g. EBV in 1000 Genomes data, up to 1.3% in ERR3242202) inflates apparent 'human retention' and must be subtracted with a custom index (NC_007605.1/NC_009334.1) for accurate figures
- Masked indexes trade a small (≤0.001%) reduction in human read removal for higher microbial retention; the virus/phage-masked index should be chosen when viral retention is a priority
- Hostile memory footprint is non-trivial: 4GB RAM for Bowtie2 short reads and 13GB for Minimap2 long reads

## Evidence

- [abstract] removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads: "our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [abstract] masked index retention: "Using Hostile with a masked reference genome further increases bacterial read retention (≥99.997%) with negligible (≤0.001%) reduction in human read removal performance"
- [methods] default index composition: "The default index `human-t2t-hla` comprises [T2T-CHM13v2.0](https://www.ncbi.nlm.nih.gov/assembly/11828891) and [IPD-IMGT/HLA](https://www.ebi.ac.uk/ipd/imgt/hla/) v3.51"
- [readme] RAM requirements: "Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2)"
- [readme] masked index usage: "For best possible retention of microbial reads, optionally use an existing index masked against bacterial and/or viral genomes, or make your own using the built-in masking utility."
