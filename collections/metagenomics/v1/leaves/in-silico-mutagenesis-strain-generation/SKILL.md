---
name: in-silico-mutagenesis-strain-generation
description: Use when benchmarking strain-variant discovery (e.g. MetaQuad) and you
  need simulated shotgun metagenomic samples with known strain composition, planted
  substitutions, and controlled mean depths instead of real clinical reads.
when_to_use_negative:
- Input is real clinical shotgun metagenomic data (e.g. the H. pylori post-eradication-therapy
  samples in this study) — skip simulation and go straight to bwa/bowtie2 alignment,
  cellsnp-lite pileup, and metaquad
- The goal is single-cell mitochondrial variant discovery (MQuad's original scope),
  not strain mixtures in shotgun metagenomic data
- No ground-truth strain composition is needed, so simulated data would add no evaluation
  value over real reads
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0102
tools:
- name: BBMap
  role: mutate.sh is run four times on the concatenated Fusobacterium reference to
    generate strain variants at specified substitution rates
- name: InSilicoSeq
  role: iss generate simulates 151 bp paired-end shotgun reads per sample with the
    NovaSeq error model at target depths
- name: bwa
  role: aligns simulated reads to the reference database in the downstream benchmark
    pipeline producing .filtered.sorted.bam files
- name: bowtie2
  role: alternative alignment tool for mapping simulated reads to the reference database
- name: cellsnp-lite
  role: piles up variants from aligned simulated reads with minMAF = 0.02 and minCOUNT
    = 100 before variant detection
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MetaQuad
  role: detects shared informative variants in the simulated 100-sample population
    to benchmark false-positive and true-positive rates
  repo: https://github.com/holab-hku/MetaQuad
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:36a751f64db5fce85483ca321c3ddbb51c4954c29abdc475e0608dec2e9d10de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - BBMap
  - InSilicoSeq
  - MetaQuad
  - Python
  - bowtie2
  - bwa
  - cellsnp-lite
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# in-silico-mutagenesis-strain-generation

## Summary

Builds simulated shotgun metagenomic benchmark data by mutating reference Fusobacterium genomes into strain variants with BBMap mutate.sh, then simulating 151 bp NovaSeq paired-end reads with InSilicoSeq across 100 samples at prescribed mean depths (3x-35.5x) to evaluate MetaQuad's shared-variant discovery.

## When to use

Use when benchmarking strain-variant discovery (e.g. MetaQuad) and you need simulated shotgun metagenomic samples with known strain composition, planted substitutions, and controlled mean depths instead of real clinical reads.

## When NOT to use

- Input is real clinical shotgun metagenomic data (e.g. the H. pylori post-eradication-therapy samples in this study) — skip simulation and go straight to bwa/bowtie2 alignment, cellsnp-lite pileup, and metaquad
- The goal is single-cell mitochondrial variant discovery (MQuad's original scope), not strain mixtures in shotgun metagenomic data
- No ground-truth strain composition is needed, so simulated data would add no evaluation value over real reads

## Inputs

- Five Fusobacterium reference genomes from NCBI (FASTA)
- Concatenated multi-species reference FASTA (simulation template)
- Per-strain substitution rates for mutate.sh
- Strain-composition specification for the 3 groups x 100 samples
- Target mean sequencing depths per sample (3x-35.5x)

## Outputs

- Four mutated strain FASTA files (BBMap mutate.sh output)
- 100 gzipped paired-end FASTQ sample pairs (151 bp mates, NovaSeq error model)
- Simulated benchmark dataset with known strain composition and realized depths for MetaQuad evaluation

## How to apply

Download the five Fusobacterium genomes from NCBI and concatenate them into a single multi-species reference FASTA to serve as the simulation template. Run BBMap mutate.sh four times on the combined reference to generate four strain-derived genome variants, each at its specified substitution rate, producing one mutated FASTA per strain. Assign the five reference species plus four mutated strains across 100 simulated samples organized into three groups with defined strain percentages. For each sample run InSilicoSeq (iss generate) with the --model novaseq error model to produce 151 bp paired-end reads, setting the read count per sample as depth x reference size / (2 x read length) so mean depths land on prescribed values spanning 3x to 35.5x. The resulting FASTQs feed the MetaQuad pipeline (bwa/bowtie2 alignment, cellsnp-lite pileup with minMAF=0.02 and minCOUNT=100, metaquad with minSample=5%).

## Related tools

- **BBMap** (mutate.sh is run four times on the concatenated Fusobacterium reference to generate strain variants at specified substitution rates)
- **InSilicoSeq** (iss generate simulates 151 bp paired-end shotgun reads per sample with the NovaSeq error model at target depths)
- **bwa** (aligns simulated reads to the reference database in the downstream benchmark pipeline producing .filtered.sorted.bam files)
- **bowtie2** (alternative alignment tool for mapping simulated reads to the reference database)
- **cellsnp-lite** (piles up variants from aligned simulated reads with minMAF = 0.02 and minCOUNT = 100 before variant detection) — https://github.com/single-cell-genetics/cellsnp-lite
- **MetaQuad** (detects shared informative variants in the simulated 100-sample population to benchmark false-positive and true-positive rates) — https://github.com/holab-hku/MetaQuad

## Examples

```
iss generate --genomes fusobacterium_strains.fasta --model novaseq --n_reads <round(depth * ref_size / 302)>
```

## Evaluation signals

- Exactly 100 gzipped paired-end FASTQ sample pairs are produced
- Realized mean sequencing depths land on the prescribed values spanning 3x to 35.5x across the three groups
- Each sample's reads match its assigned strain reference and show the expected per-strain variant sites
- Per-sample read counts equal depth x reference size / (2 x read length) with 151 bp mates
- Downstream pipeline (alignment, cellsnp-lite with minMAF=0.02/minCOUNT=100, metaquad with minSample=5%) recovers the planted substitutions at the expected group strain percentages

## Limitations

- Strain diversity is introduced only via substitutions from BBMap mutate.sh; other variant classes are not modeled in this benchmark
- Read errors follow InSilicoSeq's NovaSeq error model, so results may not transfer to other sequencing platforms or library preparations
- Benchmark coverage is limited to mean depths of 3x-35.5x; deeper or shallower regimes are not addressed by this simulation design

## Evidence

- [methods] Run BBMap mutate.sh four times on the combined reference to generate four strain-derived genome variants, each at its specified substitution rate, producing one mutated FASTA per strain.: [evidence span withheld — anchor status: not_found]
- [methods] InSilicoSeq NovaSeq read simulation at 151 bp per mate across 100 samples: [evidence span withheld — anchor status: not_found]
- [methods] Per-sample depth-derived read count formula spanning 3x-35.5x: [evidence span withheld — anchor status: not_found]
- [methods] Validation of simulated sample count and depth range: [evidence span withheld — anchor status: not_found]
- [readme] README context on MetaQuad's shotgun metagenomic scope: "MetaQuad is an extension of [MQuad](https://github.com/single-cell-genetics/MQuad), specifically tailored to support shotgun metagenomic data."
- [intro] Downstream alignment step applied to simulated reads: [evidence span withheld — anchor status: not_found]
