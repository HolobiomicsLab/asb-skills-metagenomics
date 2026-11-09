---
name: simulated-metagenome-read-generation
description: Use when benchmarking a metagenomic classifier (Kraken, Megablast, PhymmBL,
  NBC) or measuring reads-per-minute throughput and you need reads with known taxonomic
  origins, or when a standard-error (HiSeq/MiSeq-like) and a high-error (simBA-5,
  2% mismatch) read set must be contrasted.
when_to_use_negative:
- You already have real metagenomic reads with known or trusted origins (e.g. SRA
  accessions for HiSeq/MiSeq) and simulation would only add model-artifact bias
- You need to characterize organisms absent from the database rather than benchmark
  classifier accuracy — the clade exclusion experiments require masking database hits,
  not new simulations
- Your sequencing platform is not Illumina-like, since the article grounds the error
  model solely in Mason's Illumina model
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Mason
  role: Illumina-model read simulator used to create the simBA-5 metagenome from complete
    RefSeq bacterial and archaeal genomes; its default mismatch and indel rates are
    multiplied by five to reach an average mismatch rate of 2%
- name: Kraken
  role: 'Downstream consumer of the simulated reads: classifies them via exact k-mer
    LCA queries for accuracy (genus-level sensitivity/precision vs Megablast, PhymmBL,
    NBC) and reads-per-minute benchmarks'
  repo: https://github.com/DerrickWood/kraken
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_007
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:26b52a49acf8563863e0f092f50f03664c56522fbeb10dc9e44933d0899c58f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DerrickWood/kraken
  tools:
  - Kraken
  - Mason
  - Megablast
  - MetaPhlAn
  - PhymmBL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# simulated-metagenome-read-generation

## Summary

Generates benchmark metagenomic read sets by simulating Illumina reads with Mason from complete bacterial and archaeal RefSeq genomes (e.g. the simBA-5 metagenome at a five-fold elevated error rate, average mismatch rate 2%), enabling controlled comparison of classifiers like Kraken against Megablast, PhymmBL and NBC where every read's true origin is known.

## When to use

Use when benchmarking a metagenomic classifier (Kraken, Megablast, PhymmBL, NBC) or measuring reads-per-minute throughput and you need reads with known taxonomic origins, or when a standard-error (HiSeq/MiSeq-like) and a high-error (simBA-5, 2% mismatch) read set must be contrasted.

## When NOT to use

- You already have real metagenomic reads with known or trusted origins (e.g. SRA accessions for HiSeq/MiSeq) and simulation would only add model-artifact bias
- You need to characterize organisms absent from the database rather than benchmark classifier accuracy — the clade exclusion experiments require masking database hits, not new simulations
- Your sequencing platform is not Illumina-like, since the article grounds the error model solely in Mason's Illumina model

## Inputs

- Set of complete bacterial and archaeal genomes from NCBI RefSeq (simulation template for simBA-5)
- Mason read simulator with Illumina model parameters (default SNP rate 0.1%, indel rate 0.1%; multiplied by five for simBA-5)
- Real metagenome read sets for contrast: NCBI Sequence Read Archive accessions SRS019120, SRS014462 and SRS015055 (HiSeq and MiSeq metagenomes)
- NCBI taxonomy database (to record each simulated read's clade of origin)

## Outputs

- Simulated FASTQ/FASTA metagenome (simBA-5) with per-read ground-truth taxonomic origin labels
- Simulation log of Mason parameters (Illumina model, scaled mismatch/indel rates) for reproducibility
- Benchmark-ready read sets for classifier sensitivity/precision and rpm throughput tables (e.g. table1_genus_sensitivity_precision.csv, speed_table.csv inputs)

## How to apply

Simulate reads with the Mason read simulator using its Illumina model, starting from the set of complete bacterial and archaeal genomes in RefSeq; for the standard-error metagenomes (HiSeq, MiSeq) use the default error model, whereas for simBA-5 multiply Mason's default SNP rate (0.1%) and indel rate (0.1%) by five, producing an average mismatch rate of 2%. Draw sequences so each simulated metagenome contains reads from ten genomes, recording each read's clade of origin from the NCBI taxonomy so downstream sensitivity and precision can be scored. When mixing with real data or preparing reads for Kraken, trim all sequences to remove low-quality bases and adapter sequences, and concatenate paired mates with 'NNNNN' so each pair is classified as a single unit (Kraken ignores k-mers spanning ambiguous bases). Use the resulting metagenomes both for accuracy benchmarks (e.g. 10,000 sequences scored at genus level) and for speed benchmarks where throughput in reads per minute is measured on HiSeq, simBA-5 and MiSeq sets.

## Related tools

- **Mason** (Illumina-model read simulator used to create the simBA-5 metagenome from complete RefSeq bacterial and archaeal genomes; its default mismatch and indel rates are multiplied by five to reach an average mismatch rate of 2%)
- **Kraken** (Downstream consumer of the simulated reads: classifies them via exact k-mer LCA queries for accuracy (genus-level sensitivity/precision vs Megablast, PhymmBL, NBC) and reads-per-minute benchmarks) — https://github.com/DerrickWood/kraken

## Evaluation signals

- Each simulated metagenome contains sequences from ten genomes (per Additional file 1: Table S1), and every read's origin taxon is recorded from the NCBI taxonomy
- simBA-5 reads show an average mismatch rate of approximately 2%, consistent with Mason's default SNP/indel rates (0.1%) multiplied by five
- When the simulated reads are classified, Kraken's throughput should reach reported thresholds (>1.5M rpm on HiSeq, >1.3M rpm on simBA-5, >890K rpm on MiSeq) and genus-level sensitivity within 2.5 percentage points of Megablast
- Clade-exclusion validation on simulated reads: precision >= 93% for all measured/excluded rank pairs, with roughly 33% of reads classified at best when the species clade is excluded

## Limitations

- Mason version and Jellyfish version are not specified in the article, so exact reproducibility of the simulated error profile is not guaranteed
- The article's genome-composition details (which ten genomes per metagenome) are in Additional file 1, which is not included in the provided text
- Simulation quality is bounded by the known-genes bias: organisms like Pelosinus fermentans (HiSeq metagenome) and Proteus vulgaris (MiSeq metagenome) cannot be correctly identified at the genus level, reflecting limits of the database rather than the simulation

## Evidence

- [discussion] The simBA-5 metagenome was created by simulating reads from the set of complete bacterial and archaeal genomes in RefSeq: "The simBA-5 metagenome was created by simulating reads from the set of complete bacterial and archaeal genomes in RefSeq"
- [discussion] Five-fold error scaling for simBA-5: "we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%"
- [discussion] Paired-read concatenation for Kraken classification: [evidence span withheld — anchor status: not_found]
- [abstract] Kraken's role as the classifier being benchmarked on the simulated reads: "Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences."
- [readme] Kraken tooling documentation pointer for running classifications on simulated data: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
