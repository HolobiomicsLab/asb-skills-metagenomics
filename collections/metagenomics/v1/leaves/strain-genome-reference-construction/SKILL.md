---
name: strain-genome-reference-construction
description: Use when a benchmark with known ground-truth strain composition and controlled
  mean sequencing depth is needed to evaluate strain-level variant detection (e.g.
  MetaQuad) on simulated shotgun metagenomic data.
when_to_use_negative:
- Input is real clinical or environmental metagenomic FASTQ with unknown strain composition
  — this skill only constructs synthetic benchmarks, not real-data references
- The goal is variant discovery itself (e.g. running MetaQuad/cellsnp-lite on already-aligned
  data) rather than building the simulation template
- A single reference genome with no strain variants is sufficient for the downstream
  analysis
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: InSilicoSeq
  role: Simulates paired-end shotgun reads (151 bp per mate) with the Novaseq error
    model for each of the 100 samples at prescribed mean depths (3x-35.5x)
- name: BBMap
  role: mutate.sh is run four times on the concatenated multi-species reference to
    generate four strain-derived genome variants at specified substitution rates
- name: MetaQuad
  role: Downstream shared-informative-variant detector that the simulated benchmark
    datasets are constructed to evaluate
  repo: https://github.com/holab-hku/MetaQuad
- name: Python
  role: Language of MetaQuad and used to compute per-sample read counts (depth x reference
    size / (2 x read length)) and orchestrate the simulation
  repo: https://github.com/holab-hku/MetaQuad.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:d008a1c4967f97c596cf9d2b2dbb6d4e929fa5c07cb8f57729d2573e5b8c90e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - BBMap
  - InSilicoSeq
  - MetaQuad
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# strain-genome-reference-construction

## Summary

Constructs multi-strain reference genomes and simulated shotgun metagenomic benchmark datasets by combining public reference genomes (e.g. Fusobacterium genomes from NCBI), BBMap mutate.sh-derived strain variants, and InSilicoSeq (iss generate) read simulation with the Novaseq error model, so that strain composition and sequencing depth (3x-35.5x) are known ground truth for evaluating shared-variant detection tools such as MetaQuad.

## When to use

Use when a benchmark with known ground-truth strain composition and controlled mean sequencing depth is needed to evaluate strain-level variant detection (e.g. MetaQuad) on simulated shotgun metagenomic data.

## When NOT to use

- Input is real clinical or environmental metagenomic FASTQ with unknown strain composition — this skill only constructs synthetic benchmarks, not real-data references
- The goal is variant discovery itself (e.g. running MetaQuad/cellsnp-lite on already-aligned data) rather than building the simulation template
- A single reference genome with no strain variants is sufficient for the downstream analysis

## Inputs

- Publicly available reference genome FASTA files from NCBI (five Fusobacterium genomes)
- Per-group target mean sequencing depths (3x to 35.5x)
- Read length (151 bp) and error model choice (Novaseq)
- Strain composition design (100 samples, three groups with defined percentages of strains)

## Outputs

- One mutated strain FASTA per BBMap mutate.sh run (four strain-derived genome variants)
- 100 gzipped paired-end FASTQ sample pairs with per-sample ground-truth strain composition
- Dataset spanning mean depths of 3x to 35.5x across three groups

## How to apply

Download the listed reference genomes (e.g. five Fusobacterium genomes from NCBI) and concatenate them into a single multi-species reference FASTA as the simulation template. Run BBMap mutate.sh four times on the combined reference at specified substitution rates to produce one mutated FASTA per strain, giving 5 reference species plus 4 mutated strains. Distribute these across 100 simulated samples organized into three groups with defined strain-composition percentages (Groups 1, 2, 3). Use InSilicoSeq (iss generate) with the NovaSeq error model (--model novaseq) to simulate paired-end 151 bp reads per sample, setting per-sample read counts as depth x reference size / (2 x read length) so that mean depths land on the prescribed values spanning 3x to 35.5x. The rationale is that recombinant strain genomes with known variant sites provide verifiable positive controls for shared informative variant discovery.

## Related tools

- **InSilicoSeq** (Simulates paired-end shotgun reads (151 bp per mate) with the Novaseq error model for each of the 100 samples at prescribed mean depths (3x-35.5x))
- **BBMap** (mutate.sh is run four times on the concatenated multi-species reference to generate four strain-derived genome variants at specified substitution rates)
- **MetaQuad** (Downstream shared-informative-variant detector that the simulated benchmark datasets are constructed to evaluate) — https://github.com/holab-hku/MetaQuad
- **Python** (Language of MetaQuad and used to compute per-sample read counts (depth x reference size / (2 x read length)) and orchestrate the simulation) — https://github.com/holab-hku/MetaQuad.git

## Examples

```
iss generate --genomes strains.fasta --model novaseq --n_reads <depth x reference_size / (2 x 151)>
```

## Evaluation signals

- Exactly 100 gzipped paired-end FASTQ sample pairs are produced
- Realized mean sequencing depths span the prescribed 3x-35.5x range across the three groups
- Each sample's reads map to its assigned strain reference and carry the expected per-strain variant sites
- Four mutated FASTA files exist, each generated at its specified substitution rate by mutate.sh
- The three sample groups each exhibit the defined strain-composition percentages

## Limitations

- Mutated strains are substitution-only derivatives of a small set of reference genomes, so the benchmark does not capture real-world structural variation or de novo strain diversity
- Read simulation uses a single error model (Novaseq, 151 bp paired-end); other sequencing technologies or read lengths are not covered by this design
- Mean depth is controlled per sample by read-count arithmetic; realized depth still depends on reference size and may not exactly hit the target

## Evidence

- [other] Run BBMap mutate.sh four times on the combined reference to generate four strain-derived genome variants, each at its specified substitution rate, producing one mutated FASTA per strain.: [evidence span withheld — anchor status: not_found]
- [other] Use InSilicoSeq (iss generate) to simulate paired-end shotgun reads of 151 bp per mate with the NovaSeq (--model novaseq) error model for each of the 100 samples.: [evidence span withheld — anchor status: not_found]
- [other] The simulation was performed on datasets consisting of 100 samples divided into three groups with different percentages of strains (Groups 1, 2, and 3), using the generated Fusobacterium genomes.: [evidence span withheld — anchor status: not_found]
- [readme] MetaQuad is a tool designed to detect shared informative variants in a population of shotgun metagenomic data.: "MetaQuad is a tool designed to detect **shared informative variants** in a population of shotgun metagenomic data."
