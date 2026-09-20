---
name: sequencing-error-model-configuration
description: Use when you must generate simulated shotgun metagenomic FASTQ benchmarks
  at prescribed mean depths with an instrument-specific error model (NovaSeq) for
  validating variant-discovery tools.
when_to_use_negative:
- When the goal is to analyze real (non-simulated) metagenomic FASTQ data — this skill
  only generates synthetic benchmarks
- When the target sequencing platform is not NovaSeq — using --model novaseq would
  produce unrealistic quality/error profiles for other instruments
- When single-end reads are required — the benchmark prescribes paired-end 151 bp
  per mate
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3697
tools:
- name: InSilicoSeq
  role: Simulates paired-end shotgun reads per sample with the NovaSeq error model
    (--model novaseq) and computed read counts
- name: BBMap
  role: mutate.sh creates the four strain-derived genome variants that serve as simulation
    templates
- name: Python
  role: Computes per-sample read counts as depth x reference size / (2 x read length)
    to hit prescribed depths
- name: MetaQuad
  role: Downstream shared-informative-variant detection tool that the simulated benchmark
    datasets evaluate
  repo: https://github.com/holab-hku/MetaQuad
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:8bad46bff941df8574e992ea3ba01e4d4853a06e3cb71443523089cbb660301c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - BBMap
  - InSilicoSeq
  - MetaQuad
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sequencing-error-model-configuration

## Summary

Configure InSilicoSeq (iss generate) with the --model novaseq error model to simulate paired-end 151 bp shotgun metagenomic reads, per-sample read counts computed from prescribed mean depths (3x–35.5x) so benchmark datasets faithfully mimic NovaSeq error profiles for evaluating MetaQuad.

## When to use

Use when you must generate simulated shotgun metagenomic FASTQ benchmarks at prescribed mean depths with an instrument-specific error model (NovaSeq) for validating variant-discovery tools.

## When NOT to use

- When the goal is to analyze real (non-simulated) metagenomic FASTQ data — this skill only generates synthetic benchmarks
- When the target sequencing platform is not NovaSeq — using --model novaseq would produce unrealistic quality/error profiles for other instruments
- When single-end reads are required — the benchmark prescribes paired-end 151 bp per mate

## Inputs

- Multi-species reference FASTA (five Fusobacterium genomes from NCBI, concatenated)
- Four strain-derived mutated FASTA variants generated with BBMap mutate.sh
- Per-sample target mean depths spanning 3x to 35.5x and group-to-strain composition assignments

## Outputs

- 100 gzipped paired-end FASTQ sample pairs (151 bp per mate) simulated with the NovaSeq error model
- 100 samples across three groups with realized mean depths spanning 3x–35.5x

## How to apply

First build the simulation template by concatenating reference genomes (five Fusobacterium genomes from NCBI) and generating four strain variants with BBMap mutate.sh at specified substitution rates, assigning strains across three sample groups. Then run InSilicoSeq (iss generate) with the --model novaseq error model on each of the 100 samples, producing paired-end reads of 151 bp per mate. Compute the per-sample read count from the target mean depth as depth x reference size / (2 x read length), choosing the read count that lands the sample on its prescribed depth within the 3x–35.5x range. The rationale is that MetaQuad's shared-variant detection depends on realistic base-quality error distributions, so the instrument model must match the intended platform rather than a default.

## Related tools

- **InSilicoSeq** (Simulates paired-end shotgun reads per sample with the NovaSeq error model (--model novaseq) and computed read counts)
- **BBMap** (mutate.sh creates the four strain-derived genome variants that serve as simulation templates)
- **Python** (Computes per-sample read counts as depth x reference size / (2 x read length) to hit prescribed depths)
- **MetaQuad** (Downstream shared-informative-variant detection tool that the simulated benchmark datasets evaluate) — https://github.com/holab-hku/MetaQuad

## Examples

```
iss generate --genomes strain_variants.fasta --model novaseq --n_reads <depth*ref_size/(2*151)> --read_length 151 --cpus 8
```

## Evaluation signals

- Exactly 100 gzipped paired-end FASTQ sample pairs are produced
- Each sample's reads match its assigned strain reference with the expected per-strain variant sites
- Realized mean depths span the prescribed 3x–35.5x range across the three groups
- Reads are 151 bp per mate and were generated with the --model novaseq error model
- Per-sample read counts equal depth x reference size / (2 x read length) for the target depth

## Limitations

- Simulated data reflect only the NovaSeq error model; results may not transfer to other sequencing platforms or error profiles
- Read count formula assumes uniform depth across the reference, so realized per-base depth can deviate for uneven reference coverage
- Strain composition is fixed by design (three groups, five reference species plus four BBMap-mutated strains), so other community structures require re-design

## Evidence

- [other] Use InSilicoSeq (iss generate) to simulate paired-end shotgun reads of 151 bp per mate with the NovaSeq (--model novaseq) error model for each of the 100 samples.: [evidence span withheld — anchor status: not_found]
- [other] Per-sample read count formula: [evidence span withheld — anchor status: not_found]
- [other] Strain template generation with BBMap: [evidence span withheld — anchor status: not_found]
