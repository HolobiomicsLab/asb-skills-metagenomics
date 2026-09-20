---
name: per-sample-depth-calibration
description: Use when constructing simulated shotgun metagenomic benchmark datasets
  where each sample must hit a target mean depth, and per-sample read counts must
  be computed from depth, reference size, and read length.
when_to_use_negative:
- The input is real sequencing data with fixed observed depths — depth calibration
  here applies only to in silico read generation.
- The downstream analysis needs single-cell or amplicon data where the depth x reference
  size / (2 x read length) paired-end shotgun read-count formula does not apply.
- No target depth schedule has been defined — the skill presumes prescribed per-sample/group
  depth values to calibrate against.
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3941
tools:
- name: InSilicoSeq
  role: Generates the paired-end shotgun reads per sample (`iss generate --model novaseq`)
    at read counts computed from the target mean depth
- name: BBMap
  role: mutate.sh creates strain-derived genome variants at specified substitution
    rates to serve as per-sample references
- name: Python
  role: Used to compute per-sample read counts as depth x reference size / (2 x read
    length) and orchestrate the 100-sample simulation
- name: MetaQuad
  role: 'Downstream consumer: the depth-calibrated simulated FASTQ files are aligned
    and used to evaluate MetaQuad''s shared-variant detection'
  repo: https://github.com/holab-hku/MetaQuad
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:c310f12608f19f79e87d77bf720e270ec954a7188180cdd54e482bd0b0e9ed5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - BBMap
  - InSilicoSeq
  - MetaQuad
  - Python
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# per-sample-depth-calibration

## Summary

A benchmark-simulation skill for setting per-sample read counts in shotgun metagenomic simulations so that each sample realizes a prescribed mean sequencing depth (here 3x–35.5x), enabling depth-controlled evaluation of strain-variant callers such as MetaQuad.

## When to use

Use when constructing simulated shotgun metagenomic benchmark datasets where each sample must hit a target mean depth, and per-sample read counts must be computed from depth, reference size, and read length.

## When NOT to use

- The input is real sequencing data with fixed observed depths — depth calibration here applies only to in silico read generation.
- The downstream analysis needs single-cell or amplicon data where the depth x reference size / (2 x read length) paired-end shotgun read-count formula does not apply.
- No target depth schedule has been defined — the skill presumes prescribed per-sample/group depth values to calibrate against.

## Inputs

- Multi-species reference FASTA (e.g., concatenated Fusobacterium genomes from NCBI)
- Strain-derived mutated FASTA files generated with BBMap mutate.sh
- Sample-to-strain assignment table for the sample groups
- Target mean depth values per sample/group (3x to 35.5x)

## Outputs

- 100 gzipped paired-end FASTQ file pairs (151 bp per mate, NovaSeq error model)
- Per-sample read counts computed as depth x reference size / (2 x read length)
- Simulated cohort spanning mean depths from 3x to 35.5x across three strain-composition groups

## How to apply

First build the simulation template by downloading reference genomes (e.g., five Fusobacterium genomes from NCBI) and generating strain variants with BBMap mutate.sh at specified substitution rates, then assign strains across samples (e.g., 100 samples in three groups with defined strain compositions). For each sample, compute the required read count as depth x reference size / (2 x read length), where read length is 151 bp per mate for paired-end reads. Generate reads with InSilicoSeq (`iss generate`) using the --model novaseq error model so simulated errors match the NovaSeq instrument profile. Verify the skill was applied correctly by confirming the expected number of gzipped paired-end FASTQ pairs exist, each sample's reads match its assigned strain with expected variant sites, and realized depths span the prescribed range (3x to 35.5x) across groups.

## Related tools

- **InSilicoSeq** (Generates the paired-end shotgun reads per sample (`iss generate --model novaseq`) at read counts computed from the target mean depth)
- **BBMap** (mutate.sh creates strain-derived genome variants at specified substitution rates to serve as per-sample references)
- **Python** (Used to compute per-sample read counts as depth x reference size / (2 x read length) and orchestrate the 100-sample simulation)
- **MetaQuad** (Downstream consumer: the depth-calibrated simulated FASTQ files are aligned and used to evaluate MetaQuad's shared-variant detection) — https://github.com/holab-hku/MetaQuad

## Examples

```
iss generate --genomes strain.fasta --model novaseq --n_reads $(( depth * ref_size / (2 * 151) )) --cpus 4 --output sample_001
```

## Evaluation signals

- 100 gzipped paired-end FASTQ sample pairs are produced (one pair per simulated sample).
- Each sample's reads align to its assigned strain reference with the expected per-strain variant sites.
- Realized mean sequencing depths span the prescribed 3x to 35.5x range across the three groups.
- Per-sample read counts match the formula depth x reference size / (2 x read length) with 151 bp mates.
- Downstream MetaQuad evaluation on the aligned simulated data (bwa/bowtie2 .filtered.sorted.bam, cellsnp-lite minMAF=0.02, minCOUNT=100) runs without depth-related failures.

## Limitations

- Depths are only realized approximately; per-sample read counts target the mean depth, so realized coverage varies around the prescribed values.
- Calibration depends on the NovaSeq error model and 151 bp paired-end reads; other instruments or read lengths require recomputing the read-count formula.
- Depth control alone does not control variant frequency composition — strain percentages must be set separately via the three-group strain-assignment design.

## Evidence

- [other] Per-sample read-count computation from target depth: [evidence span withheld — anchor status: not_found]
- [other] InSilicoSeq read generation parameters: [evidence span withheld — anchor status: not_found]
- [other] Validation of the simulated dataset: [evidence span withheld — anchor status: not_found]
- [readme] Downstream alignment of simulated reads: "Filter and align reads from shotgun metagenomic sequencing to a reference database using alignment tools such as bwa or bowtie2."
