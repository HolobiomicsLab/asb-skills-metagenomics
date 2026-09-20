---
name: fungal-marker-gene-read-mapping
description: Use when shotgun metagenomic FASTQ samples need fungal taxonomic profiling
  (species detection rates, prevalence, abundance tables) via read mapping to a curated
  fungal marker-gene database rather than ITS amplicon-style analysis.
when_to_use_negative:
- The goal is functional (protein-level) annotation of reads — that requires DIAMOND
  blastx against FunOMIC-P (coverage >95%, identity >99%, e-value <10e-10), not marker-gene
  read mapping
- The input is already a taxonomy/abundance table rather than raw sequencing reads
- Fungal genome assembly and marker gene extraction from Candida/isolate data is the
  goal (BBNorm/SPAdes/BUSCO database-construction steps), not profiling of metagenomes
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: FunOMIC
  role: Automated pipeline whose main program FunOMIC.sh performs the read mapping
    to the FunOMIC-T database and aggregates filtered alignments into fungal abundance
    profiles
  repo: https://github.com/ManichanhLab/FunOMIC
- name: bowtie2
  role: Aligns shotgun reads to the 1.69 million fungal single-copy marker gene sequences
    in FunOMIC-T (v2.3.4.3)
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Samtools
  role: Filters bowtie2 alignments, retaining only mapped reads with >80% of read
    length aligned and q-score over 30 (v1.9)
  repo: http://www.htslib.org/
- name: FLASH2
  role: Required third-party tool for the full FunOMIC installation (merges overlapping
    paired-end reads)
  repo: https://github.com/dstreett/FLASH2
- name: Diamond
  role: Required third-party tool for the full FunOMIC installation; used for the
    FunOMIC-P functional arm, not taxonomic marker mapping
  repo: https://github.com/bbuchfink/diamond/wiki
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:e094757d675b1b7d28c5acfb43d8a30a27ed3e95b19ba8e8a6750b468eb4a65b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - ART
  - Diamond
  - FLASH2
  - FunOMIC
  - KneadData
  - Samtools
  - bowtie2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# fungal-marker-gene-read-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps metagenomic shotgun reads against the FunOMIC-T database of 1.69 million fungal single-copy marker genes using bowtie2, then filters alignments with samtools (>80% read length aligned, q-score >30) to build per-sample fungal species abundance/prevalence profiles.

## When to use

Use when shotgun metagenomic FASTQ samples need fungal taxonomic profiling (species detection rates, prevalence, abundance tables) via read mapping to a curated fungal marker-gene database rather than ITS amplicon-style analysis.

## When NOT to use

- The goal is functional (protein-level) annotation of reads — that requires DIAMOND blastx against FunOMIC-P (coverage >95%, identity >99%, e-value <10e-10), not marker-gene read mapping
- The input is already a taxonomy/abundance table rather than raw sequencing reads
- Fungal genome assembly and marker gene extraction from Candida/isolate data is the goal (BBNorm/SPAdes/BUSCO database-construction steps), not profiling of metagenomes

## Inputs

- Paired-end shotgun metagenomic FASTQ files (e.g. KneadData-cleaned reads from NCBI SRA)
- FunOMIC-T taxonomic database folder (1.69 million fungal single-copy marker gene index from https://manichanh.vhir.org/funomic/)
- Optional bacterial contaminant database path and FunOMIC-P functional database folder

## Outputs

- Per-sample fungal species abundance table
- Per-species prevalence across samples (e.g. Saccharomyces cerevisiae in 52.4% of samples)
- Overall fungal detection rate (fraction of samples with detectable fungal DNA) and mean FunOMIC-T mapping rate

## How to apply

Run the FunOMIC.sh pipeline main program so cleaned paired-end reads (e.g. KneadData decontaminated FASTQ from NCBI SRA samples) are mapped with bowtie2 v2.3.4.3 against the FunOMIC-T folder containing the 1.69 million fungal single-copy marker gene index. Optionally supply a bacterial database path (-a) and the FunOMIC-P functional database folder (-c) if functional profiling is also needed; set thread count with -t. Filter the resulting alignments with samtools v1.9, keeping only mapped reads where more than 80% of the read length aligned to the reference and the q-score is over 30 — this strict threshold protects specificity, as validated by <0.01% of fungal marker genes mapping to UHGG. Aggregate filtered alignments per sample with the in-house script from github.com/ManichanhLab/FunOMIC to produce per-sample fungal species abundance tables, from which per-species prevalence (fraction of samples positive) and overall detection rate are computed.

## Related tools

- **FunOMIC** (Automated pipeline whose main program FunOMIC.sh performs the read mapping to the FunOMIC-T database and aggregates filtered alignments into fungal abundance profiles) — https://github.com/ManichanhLab/FunOMIC
- **bowtie2** (Aligns shotgun reads to the 1.69 million fungal single-copy marker gene sequences in FunOMIC-T (v2.3.4.3)) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Samtools** (Filters bowtie2 alignments, retaining only mapped reads with >80% of read length aligned and q-score over 30 (v1.9)) — http://www.htslib.org/
- **FLASH2** (Required third-party tool for the full FunOMIC installation (merges overlapping paired-end reads)) — https://github.com/dstreett/FLASH2
- **Diamond** (Required third-party tool for the full FunOMIC installation; used for the FunOMIC-P functional arm, not taxonomic marker mapping) — https://github.com/bbuchfink/diamond/wiki

## Examples

```
FunOMIC.sh -1 read1.fastq -2 read2.fastq -p output_prefix -o output_folder -a bacterialDB_path -b FunOMIC-T_folder -c FunOMIC-P_folder -t 8
```

## Evaluation signals

- Specificity check: mapping the 1.69M FunOMIC-T markers to UHGG (204,938 genomes) via bowtie2 yields <0.01% cross-mapping
- Mock community check: a bacterial mock community of 903 genomes from 458 species (19,301,201 simulated Illumina reads) shows <0.01% mapping rate to FunOMIC-T; in a ~1000:1 bacterial:fungal mix, none of 999,021 bacterial reads align (specificity 0.9999)
- Reproduction benchmark on the 2,679 human metagenomes: fungal detection rate of 72.9% of samples and mean FunOMIC-T mapping rate of 4.72E-05 are recovered
- Per-species prevalence sanity: Saccharomyces cerevisiae dominates healthy gut mycobiome at 52.4% detection; Aspergillus recurvatus dominates CD/T1D, Malassezia dominates infants
- Alignment filter invariant: retained reads all have >80% of read length aligned with q-score >30

## Limitations

- Extraction and quality of single-copy marker genes depend on completeness of available fungal genomes, which may result in lower fungal taxonomy coverage compared with amplicon-based databases
- High inter-kingdom conservation of protein-coding genes means some bacterial contamination is not totally preventable even under strict thresholds (mainly relevant to the FunOMIC-P functional arm; no pre-filtration of bacterial reads existed in this version)
- The pipeline requires Python 3.3+, R 4.0+ (KEGGREST, RCurl), bowtie2, samtools, flash2 and diamond to be installed and the databases downloaded separately from https://manichanh.vhir.org/funomic/

## Evidence
