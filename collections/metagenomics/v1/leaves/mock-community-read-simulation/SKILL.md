---
name: mock-community-read-simulation
description: Use when validating a taxonomic or functional profiling database against
  cross-kingdom false-positive mapping, by simulating mock-community FASTQ reads with
  ART and measuring mapping rates or specificity.
when_to_use_negative:
- Input is real shotgun metagenome FASTQ rather than simulated reads with known ground
  truth — specificity cannot be computed without truth labels
- The target database is bacterial (e.g. UHGG) and the question is fungal-to-bacterial
  contamination measured by marker-gene alignment, not read-level mock simulation
- Profiling a real sample through the FunOMIC.sh pipeline (this skill is the validation/benchmarking
  step, not the profiling step)
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: ART
  role: Generates synthetic Illumina-format next-generation sequencing reads from
    mock community genome compositions
- name: bowtie2
  role: Aligns simulated reads to the FunOMIC-T marker-gene database within the FunOMIC
    pipeline
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Diamond blastx v2.0.8
  role: Aligns simulated mixed-community reads to FunOMIC-P under the nine swept identity/coverage/e-value
    threshold combinations
  repo: https://github.com/bbuchfink/diamond/wiki
- name: FunOMIC
  role: Automated pipeline (Bowtie2 + Samtools filtering, >80% alignment length, q-score
    >30) against which simulated read mapping rates are computed
  repo: https://github.com/ManichanhLab/FunOMIC
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:b730a074131a161e87d1737ab82a541a3303b6fb3882c29aa2087bcfc3cdb62f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - ART
  - Diamond blastx v2.0.8
  - FunOMIC
  - bowtie2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mock-community-read-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate synthetic Illumina reads with ART from defined bacterial and mixed bacterial:fungal mock communities to benchmark the specificity of the FunOMIC-T taxonomic database and the Diamond blastx thresholds of the FunOMIC-P functional database.

## When to use

Use when validating a taxonomic or functional profiling database against cross-kingdom false-positive mapping, by simulating mock-community FASTQ reads with ART and measuring mapping rates or specificity.

## When NOT to use

- Input is real shotgun metagenome FASTQ rather than simulated reads with known ground truth — specificity cannot be computed without truth labels
- The target database is bacterial (e.g. UHGG) and the question is fungal-to-bacterial contamination measured by marker-gene alignment, not read-level mock simulation
- Profiling a real sample through the FunOMIC.sh pipeline (this skill is the validation/benchmarking step, not the profiling step)

## Inputs

- Bacterial genome FASTA set (e.g. 903 genomes / 458 species collected from NCBI)
- Mixed mock community genome composition (top 20 fungal + top 20 bacterial species, from Supplementary Table S1)
- FunOMIC-T database (1.69 million fungal single-copy marker genes)
- FunOMIC-P database (3,413,239 non-redundant fungal proteins)
- ART simulation parameters for Illumina-formatted output reads

## Outputs

- Simulated Illumina FASTQ read files (e.g. 999,021 bacterial reads for the mixed community; 19,301,201 reads for the 903-genome community)
- validation_metrics.csv with bacterial mapping rate (<0.01%) and specificity (0.9999)
- Tab-separated table of nine Diamond blastx threshold combinations with retained fungal read hit fractions

## How to apply

Build mock community compositions (e.g. 903 bacterial genomes from 458 human-body species from NCBI, or a mixed community of the top 20 fungal plus top 20 bacterial species with a ~1000:1 bacterial:fungal read ratio) and use ART to simulate Illumina-formatted FASTQ reads from them. For taxonomic validation, align both simulated read sets to the 1.69-million-marker-gene FunOMIC-T database via the FunOMIC pipeline's Bowtie2 step, keeping mapped reads only when >80% of the read length aligns with q-score over 30 (Samtools filtering), then compute bacterial mapping rate and specificity as false positive / (false positive + true negative). For functional threshold optimization, simulate reads from the mixed mock community coding gene sequences and align them to FunOMIC-P with Diamond blastx v2.0.8 (e-value < 10e-10), sweeping a 3x3 grid of identity (>90%, >95%, >99%) by read coverage (>90%, >95%, >99%) thresholds and recording the retained fungal hit fraction per combination. The rationale is that simulated reads with known ground truth allow exact quantification of cross-kingdom contamination and threshold selection before applying the pipeline to real 2,679-sample metagenome data.

## Related tools

- **ART** (Generates synthetic Illumina-format next-generation sequencing reads from mock community genome compositions)
- **bowtie2** (Aligns simulated reads to the FunOMIC-T marker-gene database within the FunOMIC pipeline) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Diamond blastx v2.0.8** (Aligns simulated mixed-community reads to FunOMIC-P under the nine swept identity/coverage/e-value threshold combinations) — https://github.com/bbuchfink/diamond/wiki
- **FunOMIC** (Automated pipeline (Bowtie2 + Samtools filtering, >80% alignment length, q-score >30) against which simulated read mapping rates are computed) — https://github.com/ManichanhLab/FunOMIC

## Evaluation signals

- Mixed mock community (~999,021 bacterial reads at ~1000:1 bacterial:fungal ratio) must yield zero bacterial reads aligning to FunOMIC-T, giving specificity 0.9999
- 903-genome / 458-species bacterial mock community (19,301,201 simulated Illumina reads) must show <0.01% mapping rate to FunOMIC-T
- Threshold sweep table must contain all nine identity x coverage combinations (identity >90/>95/>99% x coverage >90/>95/>99% at e-value < 10e-10)
- The coverage >95%, identity >99% combination must show the highest retained fungal read hit rate (~70%), confirming it as the FunOMIC-P optimum
- Mapped reads retained only when >80% of read length aligns with q-score over 30 (Samtools filtering) before computing metrics

## Limitations

- High inter-kingdom conservation of protein-coding genes means bacterial contamination in functional annotation was not totally preventable even with strict Diamond thresholds
- No bacterial read filtration step exists before functional annotation; only proposed for a future FunOMIC update
- Specificity depends on the completeness of available fungal genomes and the simulation ground truth; results apply to ART-simulated Illumina reads, not real sequencer error profiles

## Evidence
