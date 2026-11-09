---
name: gene-catalogue-expansion-quantification
description: Use when comparing a newly built non-redundant gene catalogue against
  a published one (e.g. VMGC-90 vs VIRGO-90) to report bidirectional coverage fractions
  and protein-space expansion.
when_to_use_negative:
- The catalogues being compared use different amino acid identity clustering thresholds
  than 90%, so the shared/covered fractions are not comparable at the reported 70.6%/30.8%
  benchmarks.
- The goal is functional annotation of genes (KEGG, CAZy) rather than measuring catalogue-to-catalogue
  sequence overlap.
- Input sequences are nucleotide contigs or genomes rather than predicted protein-coding
  gene catalogues.
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: MMseqs2
  role: Builds a database of the VIRGO-90 gene set and runs the all-vs-all protein
    search with 90% identity / 80% coverage thresholds used to call covered and shared
    genes
- name: GNU parallel 20201122
  role: Parallelizes the search/parsing steps over the large VMGC-90 vs VIRGO-90 comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:6815d993ffb25126ea18b3070ff6d82e72d8a485594ea5f4654a9a21eabaa0ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNU parallel 20201122
  - MMseqs2
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-catalogue-expansion-quantification

## Summary

Quantifies mutual gene coverage between two protein-coding gene catalogues (here VMGC-90, ~26M clustered genes, and the 0.95M-gene VIRGO catalogue) via all-vs-all protein search at 90% amino acid identity and 80% sequence coverage, to measure how much one catalogue expands the protein space of the other.

## When to use

Use when comparing a newly built non-redundant gene catalogue against a published one (e.g. VMGC-90 vs VIRGO-90) to report bidirectional coverage fractions and protein-space expansion.

## When NOT to use

- The catalogues being compared use different amino acid identity clustering thresholds than 90%, so the shared/covered fractions are not comparable at the reported 70.6%/30.8% benchmarks.
- The goal is functional annotation of genes (KEGG, CAZy) rather than measuring catalogue-to-catalogue sequence overlap.
- Input sequences are nucleotide contigs or genomes rather than predicted protein-coding gene catalogues.

## Inputs

- VMGC-90 protein catalogue FASTA (from Zenodo record 10457006)
- VIRGO human vaginal non-redundant gene catalogue (0.95M genes; VIRGO-90 subset, 0.62M genes)

## Outputs

- MMseqs2 all-vs-all alignment table (m8/BLAST-tab format)
- vmgc_virgo90_overlap_stats.tsv with VIRGO-90 covered fraction (70.6%), VMGC-90 shared fraction (30.8%), and protein-space increase (162%)

## How to apply

Download the new catalogue's protein sequences (VMGC-90 from Zenodo record 10457006) and the reference catalogue (VIRGO, 0.95 million genes; 0.62M genes at the VIRGO-90 clustering). Build an MMseqs2 database for the reference gene set and run an all-vs-all search of the new catalogue's proteins against it using 90% amino acid identity and 80% sequence coverage thresholds. Parse the alignment output, marking each VIRGO-90 gene as covered if matched by any VMGC-90 protein at those thresholds, and each VMGC-90 protein as shared if it matches a VIRGO-90 gene. Compute the overlap fractions (VIRGO-90 covered ~70.6%, VMGC-90 shared ~30.8%) and the relative expansion of the new catalogue ((1 - shared fraction)/covered basis; expected 162% increase), writing the statistics to a TSV such as vmgc_virgo90_overlap_stats.tsv (Fig. 6b of the paper).

## Related tools

- **MMseqs2** (Builds a database of the VIRGO-90 gene set and runs the all-vs-all protein search with 90% identity / 80% coverage thresholds used to call covered and shared genes)
- **GNU parallel 20201122** (Parallelizes the search/parsing steps over the large VMGC-90 vs VIRGO-90 comparison)

## Examples

```
mmseqs easy-search VMGC-90.faa VIRGO-90.faa vmgc_vs_virgo90.m8 tmp --min-seq-id 0.9 -c 0.8
```

## Evaluation signals

- Percentage of VIRGO-90 genes covered by VMGC-90 matches ~70.6% within a small tolerance of the published figure (Fig. 6b)
- Percentage of VMGC-90 genes also present in VIRGO-90 ~30.8% within a small tolerance
- Computed relative protein-space expansion of VMGC-90 over VIRGO ~162%
- Output file vmgc_virgo90_overlap_stats.tsv contains exactly the three statistics with plausible denominators (0.62M VIRGO-90 genes)
- Every covered/shared call is traceable to at least one alignment row meeting both the 90% identity and 80% coverage thresholds

## Limitations

- Coverage fractions are threshold-dependent: results change if the 90% amino acid identity or 80% sequence coverage cutoffs are altered.
- Many VMGC phages are unknown and functional exploration against existing databases is limited, so overlap does not imply functional equivalence.
- VMGC sequences are incomplete, especially for vaginal samples from non-western populations, which may understate overlap with other catalogues.

## Evidence

- [abstract] Protein-coding gene clustering at 50%, 90% and 95% amino acid identity to build VMGC-50/90/95 catalogues: "we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity"
- [abstract] VIRGO reference catalogue size: "the human vaginal non-redundant gene catalogue (VIRGO)10, which includes a total of 0.95 million genes"
- [discussion] Expansion claim over VIRGO: "surpassing the VIRGO database10, VMGC offers an extensive expansion of microbial gene sequences"
- [readme] VMGC data availability for the comparison inputs: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
