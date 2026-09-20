---
name: taxonomic-read-mapping-validation
description: Use when a newly built taxonomic database (e.g. fungal marker genes)
  must be shown to reject reads from off-target organisms such as bacteria before
  deployment on real metagenomes.
when_to_use_negative:
- Validating the functional database FunOMIC-P, which uses DIAMOND blastx thresholds
  (coverage >95%, identity >99%, e-value <10e-10) rather than the Bowtie2 read-mapping
  workflow
- Assessing the taxonomic database's coverage or recall of true fungal targets — this
  skill measures off-target (bacterial) rejection only
- Input data are real metagenome reads rather than simulated mock communities with
  known ground truth
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: ART
  role: Simulates Illumina formatted sequencing reads from the bacterial and mixed
    mock community genomes
- name: FunOMIC
  role: Pipeline that maps simulated reads to the FunOMIC-T database using Bowtie2
    and applies Samtools filtering
  repo: https://github.com/ManichanhLab/FunOMIC
- name: bowtie2
  role: Aligns the ART-simulated reads against the 1.69 million fungal marker genes
    of FunOMIC-T
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:c6a4466efd058709566c01049469d3840e0655f523c0ccbb82f4acb401153238
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
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# taxonomic-read-mapping-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates the specificity of a taxonomic marker-gene database (e.g. FunOMIC-T) by simulating bacterial Illumina reads with ART and confirming they fail to map, quantifying cross-kingdom false-positive mapping.

## When to use

Use when a newly built taxonomic database (e.g. fungal marker genes) must be shown to reject reads from off-target organisms such as bacteria before deployment on real metagenomes.

## When NOT to use

- Validating the functional database FunOMIC-P, which uses DIAMOND blastx thresholds (coverage >95%, identity >99%, e-value <10e-10) rather than the Bowtie2 read-mapping workflow
- Assessing the taxonomic database's coverage or recall of true fungal targets — this skill measures off-target (bacterial) rejection only
- Input data are real metagenome reads rather than simulated mock communities with known ground truth

## Inputs

- FunOMIC-T taxonomic database (1.69 million fungal single-copy marker genes from 4,839 high-quality fungal genomes)
- Mock community genome composition (Supplementary Table S1)
- 903 bacterial genomes (458 species) inhabiting the human body, from NCBI
- ART-simulated Illumina FASTQ reads from bacterial-only and mixed (bacterial:fungal ~1000:1) mock communities

## Outputs

- validation_metrics.csv with bacterial mapping rate and specificity
- Specificity of 0.9999 for the mixed mock community
- Bacterial mapping rate <0.01% for the 903-genome bacterial community

## How to apply

Simulate Illumina reads with ART from the off-target panel (here 903 bacterial genomes from 458 human-associated species downloaded from NCBI) and from a mixed mock community built from the top 20 fungal and top 20 bacterial species of the 2,679 human metagenomes, tuned to a bacterial:fungal read ratio of ~1000 (~999,021 bacterial reads). Align both simulated read sets against the database (FunOMIC-T, 1.69 million fungal single-copy marker genes) through the FunOMIC pipeline using Bowtie2, keeping mapped reads only when more than 80% of the read length aligns with a q-score over 30 (Samtools filtering). Compute the bacterial mapping rate for the 903-genome panel and the specificity false positive / (false positive + true negative) for the mixed community, and write the metrics into validation_metrics.csv. The validation passes if the 903-genome community shows <0.01% mapping and the mixed community yields a specificity of 0.9999 (none of the 999,021 bacterial reads aligning).

## Related tools

- **ART** (Simulates Illumina formatted sequencing reads from the bacterial and mixed mock community genomes)
- **FunOMIC** (Pipeline that maps simulated reads to the FunOMIC-T database using Bowtie2 and applies Samtools filtering) — https://github.com/ManichanhLab/FunOMIC
- **bowtie2** (Aligns the ART-simulated reads against the 1.69 million fungal marker genes of FunOMIC-T)

## Examples

```
FunOMIC.sh -1 bacterial_mocks_R1.fastq -2 bacterial_mocks_R2.fastq -p bacterial_mock -o validation_out -a bacterialDB_path -b FunOMIC-T_folder -c FunOMIC-P_folder -t 8
```

## Evaluation signals

- None of the 999,021 simulated bacterial reads from the mixed mock community align against FunOMIC-T
- Specificity = false positive / (false positive + true negative) reaches 0.9999
- Bacterial mapping rate of the 903-genome community (19,301,201 simulated Illumina reads) is below 0.01%
- All retained mappings satisfy the filter: >80% of read length aligned with q-score over 30
- validation_metrics.csv exists and contains both computed metrics (mapping rate and specificity)

## Limitations

- The strict mapping threshold (>80% alignment length, q>30) suppresses cross-kingdom false positives but may also reduce sensitivity for divergent fungi
- High inter-kingdom conservation of protein-coding genes means bacterial contamination was not totally preventable, particularly for the functional database
- FunOMIC-P currently includes no bacterial read filtration step before functional annotation; filtration was only proposed for a future update

## Evidence
