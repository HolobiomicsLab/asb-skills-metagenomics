---
name: mycobiome-abundance-table-generation
description: Use when shotgun metagenomic FASTQ samples must be profiled for fungal
  species composition, e.g. building per-sample abundance/prevalence tables for gut,
  infant, or disease-cohort mycobiome analysis.
when_to_use_negative:
- Input is amplicon (ITS) data — FunOMIC-T is designed for shotgun metagenomic reads
  mapped to single-copy marker genes
- The goal is fungal functional profiling of proteins — use the FunOMIC-P database
  with Diamond blastx instead
- Data are non-fungal-targeted RNA-seq or metatranscriptomic reads where the marker-gene
  mapping model was not validated
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: FunOMIC
  role: Pipeline main program (FunOMIC.sh) orchestrating marker-gene mapping and per-sample
    abundance aggregation via the in-house GitHub script
  repo: https://github.com/ManichanhLab/FunOMIC
- name: bowtie2
  role: Maps cleaned sequencing reads to the 1.69 million fungal single-copy marker
    genes in FunOMIC-T
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: Filters bowtie2 alignments, retaining mapped reads with >80% read-length aligned
    and q-score above 30
  repo: http://www.htslib.org/
- name: FLASH2
  role: Merges paired-end reads as a required third-party dependency of the FunOMIC
    pipeline installation
  repo: https://github.com/dstreett/FLASH2
- name: KneadData
  role: Quality control and decontamination of public SRA shotgun metagenomes prior
    to FunOMIC mapping (upstream step)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:c88b106d815128a60b5b379d2888a502a8d33329421f8846274bbf50683a5252
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
  - FLASH2
  - FunOMIC
  - KneadData
  - Samtools
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mycobiome-abundance-table-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates per-sample fungal species abundance tables from human shotgun metagenomes by mapping quality-controlled reads to the FunOMIC-T database of 1.69 million fungal single-copy marker genes, enabling downstream taxonomic profiling and prevalence statistics of the human mycobiome.

## When to use

Use when shotgun metagenomic FASTQ samples must be profiled for fungal species composition, e.g. building per-sample abundance/prevalence tables for gut, infant, or disease-cohort mycobiome analysis.

## When NOT to use

- Input is amplicon (ITS) data — FunOMIC-T is designed for shotgun metagenomic reads mapped to single-copy marker genes
- The goal is fungal functional profiling of proteins — use the FunOMIC-P database with Diamond blastx instead
- Data are non-fungal-targeted RNA-seq or metatranscriptomic reads where the marker-gene mapping model was not validated

## Inputs

- Quality-controlled paired-end shotgun metagenomic FASTQ files (e.g. KneadData-decontaminated reads, read1.fastq and read2.fastq)
- FunOMIC-T taxonomic database folder (1.69 million fungal single-copy marker genes) downloaded from https://manichanh.vhir.org/funomic/
- Optional bacterial database path (bacterialDB) for host/bacterial subtraction via the -a flag

## Outputs

- Per-sample fungal species abundance table (species x sample matrix) built by the in-house FunOMIC aggregation script
- Per-species prevalence statistics (e.g. Saccharomyces cerevisiae detected in 52.4% of samples)
- Summary metrics: fraction of samples with detectable fungal DNA and mean FunOMIC-T mapping rate

## How to apply

Run the FunOMIC.sh pipeline main program, which maps cleaned paired-end reads against the FunOMIC-T taxonomic database with bowtie2. Filter alignments with samtools, keeping only mapped reads where more than 80% of the read length aligned to the reference sequence with a q-score over 30; this strict threshold preserves fungal specificity (validated at 0.9999 against simulated bacterial reads). Aggregate the filtered alignments per sample using the in-house FunOMIC script from the ManichanhLab GitHub to build the per-sample fungal species abundance table. From the table, compute per-species sample prevalence and overall fungal detection rate (fraction of samples with detectable fungal DNA) and the mean FunOMIC-T mapping rate. Decision point: reads failing the 80%/q30 filter are discarded rather than reallocated, since the marker-gene design trades sensitivity for fungal-specificity.

## Related tools

- **FunOMIC** (Pipeline main program (FunOMIC.sh) orchestrating marker-gene mapping and per-sample abundance aggregation via the in-house GitHub script) — https://github.com/ManichanhLab/FunOMIC
- **bowtie2** (Maps cleaned sequencing reads to the 1.69 million fungal single-copy marker genes in FunOMIC-T) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (Filters bowtie2 alignments, retaining mapped reads with >80% read-length aligned and q-score above 30) — http://www.htslib.org/
- **FLASH2** (Merges paired-end reads as a required third-party dependency of the FunOMIC pipeline installation) — https://github.com/dstreett/FLASH2
- **KneadData** (Quality control and decontamination of public SRA shotgun metagenomes prior to FunOMIC mapping (upstream step))

## Examples

```
FunOMIC.sh -1 read1.fastq -2 read2.fastq -p mysample -o out_folder -a bacterialDB_path -b FunOMIC-T_folder -c FunOMIC-P_folder -t 8
```

## Evaluation signals

- Fungal detection rate across the sample cohort matches the reported 72.9% of samples with detectable fungal DNA
- Mean FunOMIC-T mapping rate matches the reported 4.72E-05
- Per-species prevalence is consistent with reported patterns, e.g. Saccharomyces cerevisiae present in 52.4% of healthy gut samples
- Only alignments with >80% read-length coverage and q-score >30 are retained in the filtered SAM/BAM output
- Zero or near-zero bacterial read mapping in mock-community validation (specificity 0.9999 for simulated bacterial reads)

## Limitations

- Extraction and quality of single-copy marker genes rely on the completeness of available fungal genomes, which may result in lower coverage of fungal taxonomies compared with ITS amplicon databases
- The low mean mapping rate (4.72E-05) means low-biomass or low-fungal-load samples may yield no detectable fungal species
- Strict filtering (80% read-length aligned, q-score >30) discards borderline alignments rather than retaining lower-confidence calls

## Evidence

- [results] none of the 999,021 bacterial reads aligned against FunOMIC-T, leading to a specificity (false positive / (false positive + true negative)) of 0.9999: [evidence span withheld — anchor status: not_found]
