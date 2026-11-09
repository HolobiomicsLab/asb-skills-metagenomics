---
name: simulated-read-generation
description: Use when building a classifier-evaluation benchmark that requires simulated
  Illumina metagenomic reads with known clade-of-origin labels per read, such as Kraken
  clade-exclusion experiments on simBA-5.
when_to_use_negative:
- Evaluating sensitivity to real sequencing artifacts (quality trimming, adapters)
  — simulation with Mason at 2% mismatch does not capture real-run error profiles
  from data such as the HiSeq or MiSeq metagenomes
- Abundance-estimation benchmarks, since reads are simulated from complete RefSeq
  genomes with full ground truth rather than from real community profiles
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Mason
  role: Simulates Illumina reads from RefSeq genomes; default mismatch and indel rates
    multiplied by five to yield the 2% average mismatch rate used for simBA-5
- name: Kraken
  role: Classifies the simulated reads with its standard k-mer LCA database after
    clade masking, to measure rank-level precision and fraction classified
  repo: github.com/DerrickWood/kraken
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:cde3f1dd82917620389a14f77328343b4d36857aba384c7dcfc90d6e0280ccca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Kraken
  - Mason
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# Simulated metagenomic read generation with known clade-of-origin ground truth

## Summary

Generates benchmark metagenomic read sets (e.g., simBA-5) by simulating Illumina reads with Mason from complete bacterial and archaeal RefSeq genomes at an elevated 2% average mismatch rate, retaining each read's NCBI-taxonomy clade of origin so classifier precision and sensitivity can be evaluated per rank.

## When to use

Use when building a classifier-evaluation benchmark that requires simulated Illumina metagenomic reads with known clade-of-origin labels per read, such as Kraken clade-exclusion experiments on simBA-5.

## When NOT to use

- Evaluating sensitivity to real sequencing artifacts (quality trimming, adapters) — simulation with Mason at 2% mismatch does not capture real-run error profiles from data such as the HiSeq or MiSeq metagenomes
- Abundance-estimation benchmarks, since reads are simulated from complete RefSeq genomes with full ground truth rather than from real community profiles

## Inputs

- Complete bacterial and archaeal genomes from NCBI RefSeq
- Mason read simulator with Illumina model (default SNP rate 0.1%, indel rate 0.1%, multiplied by five for simBA-5)
- NCBI taxonomy database for clade-of-origin assignment

## Outputs

- Simulated 100 bp Illumina read set (e.g., simBA-5 metagenome) in FASTA/FASTQ
- Per-read ground-truth table mapping each read to its clade of origin (species through phylum) in the NCBI taxonomy

## How to apply

Simulate reads with the Mason read simulator using its Illumina model, and for simBA-5 multiply Mason's default SNP and indel rates by five so the average mismatch rate reaches 2% (0.1% SNP and 0.1% indel defaults scaled up). Draw source genomes from the set of complete bacterial and archaeal genomes in NCBI RefSeq, and record for every read the NCBI-taxonomy clade of origin down through species, genus, family, order, and phylum. These ground-truth labels are what allow downstream clade-exclusion analyses: reads whose origin lacks a taxonomic entry at a measured or excluded rank are dropped, and a read is kept only if at least two other taxa in the database at the excluded rank share the origin's taxon at the measured rank. The resulting labeled read set (with database hits attributable to an excluded clade masked) is classified with Kraken's standard k-mer LCA database to compute per-rank precision and fraction classified.

## Related tools

- **Mason** (Simulates Illumina reads from RefSeq genomes; default mismatch and indel rates multiplied by five to yield the 2% average mismatch rate used for simBA-5)
- **Kraken** (Classifies the simulated reads with its standard k-mer LCA database after clade masking, to measure rank-level precision and fraction classified) — github.com/DerrickWood/kraken

## Evaluation signals

- Every read carries a resolvable clade-of-origin label in the NCBI taxonomy at species, genus, family, order, and phylum ranks
- Average mismatch rate in the simulated reads matches the intended 2% (five-fold the Mason default 0.1% SNP and 0.1% indel rates)
- Downstream clade-exclusion reproduction achieves precision >= 93% for every measured/excluded rank pair
- Best-case sensitivity of roughly 33% of reads classified when the species clade is excluded — deviation signals simulation or labeling errors
- Filter invariants hold: reads lacking a taxonomic entry at measured or excluded rank, or without two other database taxa at the excluded rank sharing the measured-rank clade, are excluded from experiments

## Limitations

- Mason simulator version is not specified in the article, limiting exact reproducibility
- Simulation parameters (5x default mismatch/indel rates ≈ 2% mismatch) make simBA-5 harder than default-Mason simulations; results do not transfer to lower error rates
- Only complete RefSeq bacterial and archaeal genomes are used, so clade-exclusion sensitivity reflects the RefSeq complete-genome database composition, not draft or environmental diversity

## Evidence

- [discussion] Simulate reads with Mason using SNP rate 0.1% and indel rate 0.1%, multiplied by five for simBA-5: "we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%"
- [discussion] simBA-5 source genomes: "The simBA-5 metagenome was created by simulating reads from the set of complete bacterial and archaeal genomes in RefSeq"
- [discussion] Mason Illumina model: "We then used the Mason read simulator [22] with its Illumina model"
- [discussion] clade-exclusion filter for reads without rank entries: [evidence span withheld — anchor status: not_found]
- [discussion] two-other-taxa exclusion filter: [evidence span withheld — anchor status: not_found]
