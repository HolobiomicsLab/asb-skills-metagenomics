---
name: cross-catalogue-gene-overlap-analysis
description: Use when a newly built gene catalogue must be benchmarked against a published
  catalogue by reciprocal gene-level similarity search, to report directional coverage
  percentages and catalogue expansion.
when_to_use_negative:
- The comparison target is genome-level or taxonomic overlap (e.g. Kraken2/Bracken
  profiling against SGBs), not protein-coding gene-set overlap.
- The two catalogues use incompatible gene prediction or clustering units (e.g. nucleotide
  contigs vs 95%-identity clusters) so identity/coverage thresholds are not comparable.
- You need novel gene discovery rather than a benchmark against an existing catalogue
  (use de novo clustering at 50/90/95% identity instead).
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MMseqs2
  role: All-vs-all protein similarity search between VMGC-90 and VIRGO-90 at 90% amino
    acid identity and 80% sequence coverage thresholds
- name: GNU parallel
  role: Parallelises the search/parsing workflow steps across the large catalogue
    files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:647d2b63bc89f870374adb2a1c1025b43e2299c6597ae2edd7524e7142e0cd68
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNU parallel
  - MMseqs2
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cross-catalogue-gene-overlap-analysis

## Summary

Quantifies mutual gene coverage between two protein-coding gene catalogues (here VMGC-90 versus the published VIRGO-90 vaginal catalogue) by all-vs-all sequence search at fixed identity/coverage thresholds, yielding directional overlap fractions and a protein-space expansion statistic (Fig. 6b: 70.6%, 30.8%, 162%).

## When to use

Use when a newly built gene catalogue must be benchmarked against a published catalogue by reciprocal gene-level similarity search, to report directional coverage percentages and catalogue expansion.

## When NOT to use

- The comparison target is genome-level or taxonomic overlap (e.g. Kraken2/Bracken profiling against SGBs), not protein-coding gene-set overlap.
- The two catalogues use incompatible gene prediction or clustering units (e.g. nucleotide contigs vs 95%-identity clusters) so identity/coverage thresholds are not comparable.
- You need novel gene discovery rather than a benchmark against an existing catalogue (use de novo clustering at 50/90/95% identity instead).

## Inputs

- VMGC-90 protein catalogue (from Zenodo record 10457006)
- VIRGO human vaginal non-redundant gene catalogue (0.95 million genes; 90%-identity subset VIRGO-90, ~0.62M genes)

## Outputs

- vmgc_virgo90_overlap_stats.tsv with three statistics: % VIRGO-90 genes covered (70.6%), % VMGC-90 genes shared (30.8%), relative protein-space expansion (162%)

## How to apply

Download the new catalogue's proteins (VMGC-90 from Zenodo record 10457006) and the reference catalogue (VIRGO-90, ~0.62M of the 0.95M VIRGO genes clustered at 90% amino acid identity). Build MMseqs2 databases and run an all-vs-all search with 90% amino acid identity and 80% sequence coverage thresholds. Mark each VIRGO-90 gene as covered if matched by any VMGC-90 protein meeting both thresholds, and each VMGC-90 protein as shared if it matches any VIRGO-90 gene. Compute the two directional fractions and the relative expansion of the new catalogue over the reference (expected: 70.6% VIRGO-90 covered, 30.8% of VMGC-90 shared, 162% protein-space increase) and write them to a TSV, e.g. vmgc_virgo90_overlap_stats.tsv.

## Related tools

- **MMseqs2** (All-vs-all protein similarity search between VMGC-90 and VIRGO-90 at 90% amino acid identity and 80% sequence coverage thresholds)
- **GNU parallel** (Parallelises the search/parsing workflow steps across the large catalogue files)

## Examples

```
mmseqs search vmgc90_db virgo90_db result tmp --min-seq-id 0.9 -c 0.8
```

## Evaluation signals

- Fraction of VIRGO-90 genes covered by VMGC-90 falls within a small tolerance of the published 70.6% (Fig. 6b).
- Fraction of VMGC-90 genes shared with VIRGO-90 falls within tolerance of 30.8%.
- Relative protein-space expansion of VMGC-90 over VIRGO approximates the reported 162%.
- Every covered/shared assignment is backed by an MMseqs2 alignment meeting both thresholds (>=90% identity AND >=80% coverage), not identity alone.
- Output TSV contains exactly the three statistics in the documented schema (vmgc_virgo90_overlap_stats.tsv).

## Limitations

- Overlap fractions depend on the chosen thresholds (90% identity, 80% coverage); different cutoffs yield different coverage estimates.
- VIRGO is an earlier, smaller catalogue (~0.95M genes) and VMGC's expansion partly reflects sampling of non-western vaginal populations and uncultured taxa not captured previously.
- Coverage is asymmetric by design: a gene counts as covered only if the full threshold is met by any single cross-catalogue alignment, so divergent paralogues may be missed.

## Evidence

- [abstract] clustering at 90% identity vs VIRGO-90 comparison: "we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity"
- [abstract] VIRGO catalogue size: "the human vaginal non-redundant gene catalogue (VIRGO)10, which includes a total of 0.95 million genes"
- [discussion] expansion over VIRGO: "surpassing the VIRGO database10, VMGC offers an extensive expansion of microbial gene sequences"
- [readme] VMGC data availability: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
