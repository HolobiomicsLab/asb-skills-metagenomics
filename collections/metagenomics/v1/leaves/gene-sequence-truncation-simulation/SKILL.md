---
name: gene-sequence-truncation-simulation
description: Use when you need to benchmark HMM-based annotation-recovery tools (e.g.
  KEMET with --hmm_mode onebm) against fragmented or partial gene sequences that mimic
  gapped metagenome-assembled genome (MAG) assemblies.
when_to_use_negative:
- Input gene sequences are complete and untruncated — the mock removal or HMM-search
  workflow should be used directly instead of truncation simulation.
- The goal is evaluating KEGG Module completeness rather than testing annotation recovery
  from fragmented sequences (use KEMET's --skip_hmm recap tables instead).
- Sequences are protein-only without the nucleotide context KEMET's nucleotidic HMM
  searches expect.
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: KEMET
  role: HMM-search mode (--hmm_mode onebm, default threshold 0.43) run on truncated
    FASTA files to recover KO annotations
  repo: https://github.com/Matteopaluh/KEMET
- name: Python
  role: Script that generates truncated test FASTA files by deleting 30% and 70% of
    each KO-annotated gene sequence's original length
- name: eggNOG-Mapper
  role: Source of the KO annotations (eggnog format, .emapper.annotations) used as
    input annotation format for KEMET
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:9908169c15b2c76ecf0d51d14ca58e0bfcb5f75e0eb4668096a6cf174eac1288
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Matteopaluh/KEMET
  repo_url: https://github.com/Matteopaluh/KEMET
  tools:
  - KEMET
  - Python
  - eggNOG-Mapper
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-sequence-truncation-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A robustness-testing skill that simulates fragmented MAG assemblies by deleting 30% or 70% of the length of KO-annotated gene sequences from complete genomes and MAGs, then measures how well KEMET's HMM-search mode recovers the original annotations. It quantifies annotation-recovery decline (20–33% at 30% truncation) as sequence truncation increases.

## When to use

Use when you need to benchmark HMM-based annotation-recovery tools (e.g. KEMET with --hmm_mode onebm) against fragmented or partial gene sequences that mimic gapped metagenome-assembled genome (MAG) assemblies.

## When NOT to use

- Input gene sequences are complete and untruncated — the mock removal or HMM-search workflow should be used directly instead of truncation simulation.
- The goal is evaluating KEGG Module completeness rather than testing annotation recovery from fragmented sequences (use KEMET's --skip_hmm recap tables instead).
- Sequences are protein-only without the nucleotide context KEMET's nucleotidic HMM searches expect.

## Inputs

- FASTA files of KO-annotated gene sequences from a complete-genome dataset and a 12-MAG dataset
- eggNOG-mapper annotations (.emapper.annotations) for the tested genomes
- KEMET with its KEGG GENES database and curated HMM profiles (github.com/Matteopaluh/KEMET)
- Python script for sequence-record trimming to 30% and 70% of original length

## Outputs

- Truncated test FASTA files at 30% and 70% of original gene length
- KEMET HMM-search hit lists per truncation condition and dataset
- Annotation recovery rates (recovered KOs / total truncated KOs) for 30% and 70% truncation in complete genomes and MAGs

## How to apply

Generate truncated test FASTA files with a Python script that trims each sequence record by 30% and 70% of its original length, applied to both a complete-genome dataset and a 12-MAG dataset of KO-annotated genes. Run KEMET in HMM-search mode on each truncated dataset with './kemet.py [FASTA_file] -a eggnog --hmm_mode onebm', keeping the default HMM hit quality threshold of 0.43. Parse the KEMET HMM-search outputs to count how many truncated KO-annotated genes are correctly recovered as HMM hits per dataset and truncation condition. Compute recovery rates (recovered KOs / total truncated KOs) and compare against the reported behavior of 20–33% recovery at 30% truncation (42% assuming unbiased gene prediction) with a further, clearest decline at 70% truncation in the MAG dataset. The rationale is to emulate the gapped nature of metagenomic assemblies and test whether curated HMM profiles can still rescue KEGG KO annotations from partial sequences.

## Related tools

- **KEMET** (HMM-search mode (--hmm_mode onebm, default threshold 0.43) run on truncated FASTA files to recover KO annotations) — https://github.com/Matteopaluh/KEMET
- **Python** (Script that generates truncated test FASTA files by deleting 30% and 70% of each KO-annotated gene sequence's original length)
- **eggNOG-Mapper** (Source of the KO annotations (eggnog format, .emapper.annotations) used as input annotation format for KEMET)

## Examples

```
./kemet.py truncated_30pct_genomes.fasta -a eggnog --hmm_mode onebm
```

## Evaluation signals

- Recovery rates at 30% truncation fall within the reported 20–33% range (42% assuming unbiased gene prediction).
- Recovery at 70% truncation is monotonically lower than at 30% truncation in both the complete-genome and 12-MAG datasets.
- The decline from 30% to 70% truncation is most clearly visible in the MAG dataset, matching the reported pattern.
- KEMET runs used the default HMM hit quality threshold of 0.43; changing this threshold would invalidate comparability with reported rates.
- Recovery rate denominators equal the number of truncated KO-annotated genes present in each dataset's FASTA files.

## Limitations

- Reported recovery figures are confounded by truncation's impact on gene prediction: 20–33% accounting for gene-prediction impact versus 42% assuming unbiased gene prediction at 30% truncation.
- High truncation (70%) pushes recovery sharply downward, especially for MAGs, so results from heavily fragmented assemblies should be treated cautiously.
- Results depend on the KEMET HMM quality filter (default 0.43); the --threshold_value parameter can alter which hits count as recovered.

## Evidence
