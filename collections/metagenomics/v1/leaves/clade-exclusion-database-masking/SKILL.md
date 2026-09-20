---
name: clade-exclusion-database-masking
description: 'Use when evaluating a k-mer LCA classifier on reads from novel organisms:
  reads simulated with Mason (2% average mismatch) from RefSeq genomes whose clade
  of origin (species through phylum) is excluded from the classification database.'
when_to_use_negative:
- The goal is absolute abundance estimation of a real metagenome rather than controlled
  evaluation of classifier sensitivity on held-out clades
- Input reads come from organisms fully represented in the database (no clade exclusion);
  use standard Kraken classification evaluation instead
- Reads were generated with mismatch/indel rates other than the 2% simBA-5 model,
  so published sensitivity/precision benchmarks no longer apply directly
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Classifies masked simBA-5 reads using its exact k-mer LCA database; the classifier
    under evaluation for rank-level precision and sensitivity
  repo: github.com/DerrickWood/kraken
- name: Mason
  role: Simulates the simBA-5 reads from complete RefSeq genomes with mismatch and
    indel rates multiplied by five (average mismatch rate 2%)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:0cf66b7b9c0a018d924982c286f7d6fd69e7f96b8d1b7777fb16878aa80913ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Kraken
  - Mason
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# clade-exclusion-database-masking

## Summary

A clade exclusion benchmark that measures Kraken's rank-level sensitivity and precision on the simBA-5 metagenome by masking database hits attributable to the excluded clade of a read's origin, from species through phylum.

## When to use

Use when evaluating a k-mer LCA classifier on reads from novel organisms: reads simulated with Mason (2% average mismatch) from RefSeq genomes whose clade of origin (species through phylum) is excluded from the classification database.

## When NOT to use

- The goal is absolute abundance estimation of a real metagenome rather than controlled evaluation of classifier sensitivity on held-out clades
- Input reads come from organisms fully represented in the database (no clade exclusion); use standard Kraken classification evaluation instead
- Reads were generated with mismatch/indel rates other than the 2% simBA-5 model, so published sensitivity/precision benchmarks no longer apply directly

## Inputs

- simBA-5 metagenome reads (Mason-simulated from complete bacterial and archaeal RefSeq genomes, ~2% average mismatch rate)
- Per-read clade-of-origin annotations from the NCBI taxonomy database
- Kraken standard k-mer LCA database built from the same RefSeq genomes

## Outputs

- table2_clade_exclusion.csv — per measured/excluded rank pair: precision and fraction of reads classified
- Per-pair sensitivity estimates (fraction of reads classified) across measured/excluded rank combinations

## How to apply

Load the simBA-5 metagenome reads (simulated with Mason from complete bacterial and archaeal RefSeq genomes with the default mismatch and indel rates multiplied by five, giving ~2% average mismatch) and record each read's clade of origin from the NCBI taxonomy. Apply the two exclusion filters: drop reads whose origin lacks a taxonomic entry at either the measured or the excluded rank, and keep a read only if at least two other taxa in the database at the excluded rank share the origin's taxon at the measured rank. For each measured/excluded rank pair (species through phylum), mask database hits attributable to the excluded clade of origin, then classify the masked reads with Kraken's standard k-mer LCA database. Compute per-pair precision (fraction of classified reads assigned to the correct measured-rank taxon) and the fraction of reads classified at all, and assemble the results into a Table 2-equivalent table.

## Related tools

- **Kraken** (Classifies masked simBA-5 reads using its exact k-mer LCA database; the classifier under evaluation for rank-level precision and sensitivity) — github.com/DerrickWood/kraken
- **Mason** (Simulates the simBA-5 reads from complete RefSeq genomes with mismatch and indel rates multiplied by five (average mismatch rate 2%))

## Examples

```
kraken --db masked_db --threads 8 simba5_reads.fa > table2_classifications.tsv; python table2_clade_exclusion.py --output table2_clade_exclusion.csv
```

## Evaluation signals

- Every reported measured/excluded rank pair must achieve precision >= 93%
- Best case should classify roughly 33% of reads when the species clade is excluded (novel species never seen)
- Sensitivity must drop dramatically as the excluded rank moves from species toward phylum, consistent with Table 2
- Reads lacking an NCBI taxonomy entry at the measured or excluded rank must be excluded before computing metrics
- Output table must cover all measured/excluded pairs from species through phylum

## Limitations

- Sensitivity drops dramatically under clade exclusion — at best ~33% of reads are classified when the species clade has never been seen
- Reads whose origin lacks a taxonomic entry at the measured or excluded rank, or with fewer than two other taxa at the excluded rank, cannot be used in the experiment
- simBA-5 relies on a specific error model (default Mason rates multiplied by five, ~2% mismatch); results are tied to that simulation setting
- Mason read simulator version and Jellyfish version are not specified in the article

## Evidence

- [results] When clades of origin were excluded, Kraken maintained rank-level precision at or above 93%: [evidence span withheld — anchor status: not_found]
- [results] sensitivity dropped dramatically with novel species: [evidence span withheld — anchor status: not_found]
- [methods] read origin lacking a rank entry excluded: [evidence span withheld — anchor status: not_found]
- [methods] two other taxa requirement at excluded rank: [evidence span withheld — anchor status: not_found]
- [discussion] simBA-5 error model: "we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%"
