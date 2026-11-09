---
name: fungal-database-specificity-assessment
description: Use when validating that a fungal marker-gene database does not falsely
  map bacterial sequencing reads — i.e., you need a specificity metric for a taxonomic
  profiler before applying it to mixed metagenomic samples.
when_to_use_negative:
- Assessing functional profiling performance — that uses the FunOMIC-P protein database
  with Diamond blastx thresholds (coverage >95%, identity >99%, e-value < 10e-10),
  not the marker-gene mapping validated here
- Measuring sensitivity/recall of fungal detection — this skill only measures false-positive
  bacterial cross-mapping against a fungal database
- Profiling real (non-simulated) metagenomic samples, where true community composition
  is unknown and a specificity metric cannot be computed directly
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ART
  role: Simulates Illumina formatted sequencing output reads from the bacterial genomes
    and the mixed mock community
- name: FunOMIC
  role: Pipeline that maps simulated reads to the FunOMIC-T database; invoked via
    FunOMIC.sh
  repo: https://github.com/ManichanhLab/FunOMIC
- name: bowtie2
  role: Aligns the simulated reads to the FunOMIC-T fungal single-copy marker gene
    database
- name: samtools
  role: Filters mapped reads, retaining alignments where >80% of the read length aligns
    with q-score over 30
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:e4d69ae8989fb50c1df4b7d6d52f64427eb23305480d6af625b623291ffc1403
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
  - FunOMIC
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fungal-database-specificity-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A mock-community validation procedure that quantifies how specific a fungal taxonomic database (FunOMIC-T, 1.69 million fungal single-copy marker genes) is by simulating bacterial Illumina reads with ART and measuring their false-mapping rate through the FunOMIC/bowtie2 pipeline.

## When to use

Use when validating that a fungal marker-gene database does not falsely map bacterial sequencing reads — i.e., you need a specificity metric for a taxonomic profiler before applying it to mixed metagenomic samples.

## When NOT to use

- Assessing functional profiling performance — that uses the FunOMIC-P protein database with Diamond blastx thresholds (coverage >95%, identity >99%, e-value < 10e-10), not the marker-gene mapping validated here
- Measuring sensitivity/recall of fungal detection — this skill only measures false-positive bacterial cross-mapping against a fungal database
- Profiling real (non-simulated) metagenomic samples, where true community composition is unknown and a specificity metric cannot be computed directly

## Inputs

- FunOMIC-T taxonomic database (1.69 million fungal single-copy marker genes)
- ART-simulated Illumina FASTQ reads from 903 bacterial genomes (458 species, NCBI)
- ART-simulated Illumina FASTQ reads from a mixed mock community (top 20 fungal + top 20 bacterial species, ~999,021 bacterial reads at bacterial:fungal ratio ~1000)
- Mock community genome composition from Supplementary Table S1

## Outputs

- Bacterial mapping rate to FunOMIC-T for the 903-genome mock community
- Specificity score (false positive / (false positive + true negative)) for the mixed mock community
- validation_metrics.csv with the computed validation metrics

## How to apply

Obtain the FunOMIC-T database (1.69 million fungal single-copy marker genes) and the mock community genome composition from Supplementary Table S1. Use ART to simulate Illumina formatted sequencing reads from 903 bacterial genomes (458 species) that inhabit the human body, downloaded from NCBI, and from a mixed mock community built from the top 20 fungal and top 20 bacterial species of the 2,679 human metagenomes, with a bacterial:fungal read ratio of ~1000 (~999,021 bacterial reads). Align both simulated read sets to FunOMIC-T through the FunOMIC pipeline using bowtie2, keeping mapped reads only when more than 80% of the read length aligns with a q-score over 30 (Samtools filtering). Compute the bacterial mapping rate for the 903-genome community and the specificity (false positive / (false positive + true negative)) for the mixed community, and write the metrics to validation_metrics.csv. The validation passes if the mixed community yields specificity 0.9999 (none of the 999,021 bacterial reads align) and the 903-genome community shows a mapping rate below 0.01% to FunOMIC-T.

## Related tools

- **ART** (Simulates Illumina formatted sequencing output reads from the bacterial genomes and the mixed mock community)
- **FunOMIC** (Pipeline that maps simulated reads to the FunOMIC-T database; invoked via FunOMIC.sh) — https://github.com/ManichanhLab/FunOMIC
- **bowtie2** (Aligns the simulated reads to the FunOMIC-T fungal single-copy marker gene database)
- **samtools** (Filters mapped reads, retaining alignments where >80% of the read length aligns with q-score over 30)

## Examples

```
FunOMIC.sh -1 mock_bacterial_r1.fastq -2 mock_bacterial_r2.fastq -p bacteria_mock -o specificity_check/ -a bacterialDB_path -b FunOMIC-T_folder -c FunOMIC-P_folder -t 8
```

## Evaluation signals

- None of the 999,021 simulated bacterial reads from the mixed mock community align against FunOMIC-T
- Mixed mock community specificity equals 0.9999 (false positive / (false positive + true negative))
- 903-genome bacterial mock community (19,301,201 Illumina reads) shows a mapping rate below 0.01% to FunOMIC-T
- Read filtering enforced: mapped reads retained only when >80% of read length aligns with q-score over 30
- validation_metrics.csv is produced and contains both the mapping-rate and specificity metrics

## Limitations

- High inter-kingdom conservation of protein-coding genes means bacterial contamination was not totally preventable, even after applying an exceedingly strict mapping threshold to the fungal functional annotation
- No bacterial read filtration step is included before functional annotation in the current pipeline; it is only proposed for a future update
- Extraction and quality of single-copy marker genes depend on completeness of available fungal genomes, which may limit fungal taxonomy coverage relative to amplicon databases

## Evidence
