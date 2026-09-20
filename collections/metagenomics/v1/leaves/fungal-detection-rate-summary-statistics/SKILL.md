---
name: fungal-detection-rate-summary-statistics
description: Use when profiling fungal taxonomic presence across a large cohort of
  human shotgun metagenomes and you need cohort-level detection rates and per-species
  prevalence tables (e.g. prevalence of S. cerevisiae across thousands of samples).
when_to_use_negative:
- The input is amplicon (ITS) data rather than shotgun metagenomes — FunOMIC-T marker-gene
  mapping is designed for shotgun reads and the article notes its taxonomy coverage
  may be lower than fungal amplicon databases
- The goal is fungal functional profiling (proteins) rather than taxonomic detection
  — that requires the FunOMIC-P database and Diamond blastx thresholds instead
- Only a single sample or small cohort is available, where cohort-level detection-rate
  and prevalence statistics are not meaningful
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3316
tools:
- name: FunOMIC
  role: Automated mycobiome pipeline (FunOMIC.sh) that maps cleaned reads against
    FunOMIC-T and aggregates alignments into per-sample species abundance tables via
    an in-house script
  repo: https://github.com/ManichanhLab/FunOMIC
- name: KneadData
  role: Quality control and decontamination of SRA shotgun metagenomic reads before
    mapping (v0.7.7-alpha)
- name: Bowtie2
  role: Maps cleaned reads to the FunOMIC-T fungal single-copy marker gene database
    (v2.3.4.3)
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Samtools
  role: Filters Bowtie2 alignments, keeping reads with >80% alignment length and q-score
    >30 (v1.9)
  repo: http://www.htslib.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:a98588ac0fd5ddaf1086f127ea0616bc02df16be0291922d21e1d424e33b66d6
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
  - Bowtie2
  - FunOMIC
  - KneadData
  - Samtools
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# fungal-detection-rate-summary-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute large-scale fungal detection statistics by mapping quality-controlled human shotgun metagenomes against the FunOMIC-T database (1.69 million fungal single-copy marker genes) and summarizing the fraction of samples with detectable fungal DNA plus per-species prevalence. This skill reproduces the FunOMIC study's headline metrics (72.9% detection rate, 4.72E-05 mean mapping rate).

## When to use

Use when profiling fungal taxonomic presence across a large cohort of human shotgun metagenomes and you need cohort-level detection rates and per-species prevalence tables (e.g. prevalence of S. cerevisiae across thousands of samples).

## When NOT to use

- The input is amplicon (ITS) data rather than shotgun metagenomes — FunOMIC-T marker-gene mapping is designed for shotgun reads and the article notes its taxonomy coverage may be lower than fungal amplicon databases
- The goal is fungal functional profiling (proteins) rather than taxonomic detection — that requires the FunOMIC-P database and Diamond blastx thresholds instead
- Only a single sample or small cohort is available, where cohort-level detection-rate and prevalence statistics are not meaningful

## Inputs

- 2,679 public human shotgun metagenomic FASTQ datasets from NCBI SRA
- FunOMIC-T taxonomic database (1.69 million fungal single-copy marker genes, from https://manichanh.vhir.org/funomic/)
- Sample metadata table (Supplementary Table S2) for per-species prevalence stratification

## Outputs

- Fraction of samples with detectable fungal DNA (target: 72.9%)
- Mean FunOMIC-T mapping rate (target: 4.72E-05)
- Per-sample fungal species abundance table
- Per-species prevalence table cross-referenced against sample metadata

## How to apply

Download the public shotgun metagenomic datasets from NCBI SRA and quality-control/decontaminate each sample with KneadData v0.7.7-alpha. Map cleaned reads to the FunOMIC-T taxonomic database (1.69 million fungal single-copy marker genes from 4,839 high-quality fungal genomes) using Bowtie2 v2.3.4.3 via the FunOMIC.sh pipeline, passing the FunOMIC-T folder with the -b flag. Filter alignments with Samtools v1.9, keeping only mapped reads with more than 80% of read length aligned and a q-score above 30 — this strict threshold guards against cross-kingdom false positives. Aggregate the filtered alignments per sample with the in-house FunOMIC GitHub script to build a per-sample fungal species abundance table, then compute the fraction of samples with any detectable fungal DNA, the mean FunOMIC-T mapping rate, and per-species prevalence cross-referenced against sample metadata (Supplementary Table S2). Validation: the summary is accepted if the reproduced fungal detection rate matches the reported 72.9% of samples, the mean mapping rate matches 4.72E-05, and per-species prevalence reproduces reported patterns (e.g. Saccharomyces cerevisiae present in 52.4% of samples).

## Related tools

- **FunOMIC** (Automated mycobiome pipeline (FunOMIC.sh) that maps cleaned reads against FunOMIC-T and aggregates alignments into per-sample species abundance tables via an in-house script) — https://github.com/ManichanhLab/FunOMIC
- **KneadData** (Quality control and decontamination of SRA shotgun metagenomic reads before mapping (v0.7.7-alpha))
- **Bowtie2** (Maps cleaned reads to the FunOMIC-T fungal single-copy marker gene database (v2.3.4.3)) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Samtools** (Filters Bowtie2 alignments, keeping reads with >80% alignment length and q-score >30 (v1.9)) — http://www.htslib.org/

## Examples

```
FunOMIC.sh -1 read1.fastq -2 read2.fastq -p sample1 -o out/ -a bacterialDB -b FunOMIC-T_folder -c FunOMIC-P_folder -t 8
```

## Evaluation signals

- Reproduced fungal detection rate across the 2,679 samples matches the reported 72.9%
- Mean FunOMIC-T mapping rate matches the reported 4.72E-05
- Saccharomyces cerevisiae is detected in ~52.4% of samples, consistent with the reported prevalence pattern
- Specificity check: bacterial reads should not map — in the mock community with ~1000:1 bacterial:fungal read ratio, none of the 999,021 bacterial reads aligned to FunOMIC-T (specificity 0.9999)
- Alignment filters were applied: only reads with >80% of length aligned and q-score >30 retained

## Limitations

- Extraction and quality of single-copy marker genes rely on the completeness of available fungal genomes, which may result in lower coverage of fungal taxonomies compared with fungal amplicon databases
- Fungal taxonomic coverage is limited to genomes with at least 30 identified single-copy marker genes from the 4,839 high-quality fungal genomes
- Mapping-based prevalence depends on the strict filter thresholds (>80% aligned length, q>30); looser filtering would change detection rates

## Evidence
