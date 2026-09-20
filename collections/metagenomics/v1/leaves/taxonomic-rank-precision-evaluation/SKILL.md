---
name: taxonomic-rank-precision-evaluation
description: Use when benchmarking a taxonomic classifier's precision and sensitivity
  at species/genus/family/phylum ranks on simulated reads from organisms whose clades
  were excluded from the reference database.
when_to_use_negative:
- Input reads are from organisms fully represented in the database — this skill is
  designed for clade-exclusion (novel organism) scenarios, not standard classification
  benchmarks
- The goal is abundance estimation from a real community rather than rank-level classification
  accuracy on simulated metagenomes
- A read's origin lacks a taxonomic entry at the measured or excluded rank, or fewer
  than two other taxa at the excluded rank share the origin's measured-rank clade
  — such reads are excluded from the experiment by design
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Classifies masked reads via exact k-mer LCA database queries; classification
    output is scored for rank-level precision and fraction classified
  repo: https://github.com/DerrickWood/kraken
- name: Mason
  role: Simulates the simBA-5 metagenome reads with its Illumina model, using mismatch
    and indel rates multiplied by five (average mismatch rate of 2%)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:7f1957e43e521a67afab32a169eefaa021951c58af346f4d7b193bbc3dd397c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DerrickWood/kraken
  tools:
  - Kraken
  - Mason
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# taxonomic-rank-precision-evaluation

## Summary

A clade-exclusion evaluation protocol that measures Kraken's rank-level precision and sensitivity on the simBA-5 metagenome by masking database hits from each read's clade of origin (species through phylum) and scoring whether classified reads land at the correct measured-rank taxon.

## When to use

Use when benchmarking a taxonomic classifier's precision and sensitivity at species/genus/family/phylum ranks on simulated reads from organisms whose clades were excluded from the reference database.

## When NOT to use

- Input reads are from organisms fully represented in the database — this skill is designed for clade-exclusion (novel organism) scenarios, not standard classification benchmarks
- The goal is abundance estimation from a real community rather than rank-level classification accuracy on simulated metagenomes
- A read's origin lacks a taxonomic entry at the measured or excluded rank, or fewer than two other taxa at the excluded rank share the origin's measured-rank clade — such reads are excluded from the experiment by design

## Inputs

- simBA-5 metagenome reads simulated with Mason from complete bacterial and archaeal RefSeq genomes (~2% average mismatch rate)
- Per-read clade-of-origin annotations from the NCBI taxonomy database
- Kraken standard k-mer LCA database built from RefSeq complete microbial genomes

## Outputs

- Per measured/excluded rank-pair precision (fraction of classified reads correct at the measured rank)
- Fraction of reads classified at all for each rank pair
- table2_clade_exclusion.csv (Table 2-equivalent results table)

## How to apply

Simulate reads with Mason (Illumina model, mismatch/indel rates multiplied by five from SNP rate 0.1% and indel rate 0.1%, i.e. ~2% average mismatch) from complete bacterial and archaeal RefSeq genomes to build the simBA-5 metagenome, recording each read's clade of origin from the NCBI taxonomy. Apply the clade-exclusion filters: drop reads whose origin lacks a taxonomic entry at either the measured or excluded rank, and keep a read only if at least two other taxa in the database at the excluded rank share the origin's taxon at the measured rank. For each measured/excluded rank pair (species through phylum), mask database hits attributable to the excluded clade of origin and classify the masked reads with Kraken's standard k-mer LCA database. Compute per-pair precision (fraction of classified reads assigned to the correct measured-rank taxon) and the fraction of reads classified at all, then assemble the results into a Table 2-equivalent CSV. Rationale: exclusion plus the two-other-taxa requirement isolates the classifier's ability to place novel reads at higher ranks rather than at their exact source genome.

## Related tools

- **Kraken** (Classifies masked reads via exact k-mer LCA database queries; classification output is scored for rank-level precision and fraction classified) — https://github.com/DerrickWood/kraken
- **Mason** (Simulates the simBA-5 metagenome reads with its Illumina model, using mismatch and indel rates multiplied by five (average mismatch rate of 2%))

## Examples

```
kraken --db kraken_std --threads 8 simBA5_masked_reads.fa > simBA5_masked.kraken
```

## Evaluation signals

- Every reported measured/excluded rank pair must achieve precision >= 93%
- Best case should classify roughly 33% of reads when the species clade is excluded; substantial shortfall indicates a failed reproduction
- All reads lacking a taxonomic entry at the measured or excluded rank must be absent from the per-pair experiment
- Reads retained for a rank pair must satisfy the two-other-taxa rule at the excluded rank
- Output CSV must contain one row per measured/excluded pair from species through phylum with both precision and fraction-classified columns

## Limitations

- Sensitivity drops dramatically under clade exclusion — at best approximately 33% of reads are classified when the species clade has never been seen, so expect low fraction-classified even in a successful run
- Reads whose origin lacks a taxonomic entry at either the measured or excluded rank cannot be evaluated for that pair
- Reads are only usable if at least two other taxa at the excluded rank share the origin's measured-rank clade; sparse databases invalidate pairs
- Kraken ignores k-mers with ambiguous nucleotides, so k-mers spanning 'N' characters (e.g. concatenated paired mates joined with 'NNNNN') do not affect classification
- Mason and Jellyfish versions are not specified in the article, hindering exact reproduction

## Evidence

- [discussion] Reads lacking taxonomy entries at either rank are excluded: [evidence span withheld — anchor status: not_found]
- [discussion] Two-other-taxa filter at the excluded rank: [evidence span withheld — anchor status: not_found]
- [discussion] simBA-5 provenance: "The simBA-5 metagenome was created by simulating reads from the set of complete bacterial and archaeal genomes in RefSeq"
- [discussion] Mason error-rate parameters for simBA-5: "we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%"
- [readme] Kraken tool installation/operation documentation: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
