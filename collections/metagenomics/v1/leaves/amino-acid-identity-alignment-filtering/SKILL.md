---
name: amino-acid-identity-alignment-filtering
description: Use when two non-redundant protein/gene catalogues (e.g. VMGC-90 vs VIRGO-90)
  must be compared to compute mutual gene coverage and protein-space expansion fractions.
when_to_use_negative:
- The catalogues were clustered at a different amino-acid identity threshold (e.g.
  VMGC-50 or VMGC-95), since 90%/80% cutoffs and expected fractions only apply to
  90%-identity catalogues
- The inputs are nucleotide contigs or genomes rather than protein-coding gene catalogue
  FASTA files
- The goal is functional annotation overlap (e.g. KEGG/CAZy categories) rather than
  sequence-level gene coverage
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MMseqs2
  role: All-vs-all protein alignment of VMGC-90 against the VIRGO-90 gene database
    at 90% amino acid identity and 80% sequence coverage thresholds
- name: GNU parallel 20201122
  role: Parallelization of the catalogue comparison and downstream parsing steps
  repo: https://github.com/RChGO/VMGC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:909f8f70c2ed4a8bd91eca853a26c30e905e33ce5db56448ad2fb29df4c59045
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RChGO/VMGC
  tools:
  - GNU parallel 20201122
  - MMseqs2
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# amino-acid-identity-alignment-filtering

## Summary

Quantifies overlap between large protein-coding gene catalogues by running an MMseqs2 all-vs-all alignment under strict amino-acid identity and coverage cutoffs (e.g. 90% identity, 80% coverage), as done to compare the VMGC-90 vaginal gene catalogue with VIRGO-90.

## When to use

Use when two non-redundant protein/gene catalogues (e.g. VMGC-90 vs VIRGO-90) must be compared to compute mutual gene coverage and protein-space expansion fractions.

## When NOT to use

- The catalogues were clustered at a different amino-acid identity threshold (e.g. VMGC-50 or VMGC-95), since 90%/80% cutoffs and expected fractions only apply to 90%-identity catalogues
- The inputs are nucleotide contigs or genomes rather than protein-coding gene catalogue FASTA files
- The goal is functional annotation overlap (e.g. KEGG/CAZy categories) rather than sequence-level gene coverage

## Inputs

- VMGC-90 protein catalogue FASTA (from Zenodo record 10457006)
- VIRGO human vaginal non-redundant gene catalogue (0.95M genes; VIRGO-90 subset ~0.62M genes)
- MMseqs2 database built from the reference gene set

## Outputs

- MMseqs2 alignment table (m8 format) of VMGC-90 proteins vs VIRGO-90 genes
- Per-gene coverage/shared flags for both catalogues
- vmgc_virgo90_overlap_stats.tsv with VIRGO-90 coverage (%), VMGC-90 shared (%) and protein-space expansion (%)

## How to apply

Download both gene catalogues (VMGC-90 proteins from Zenodo record 10457006 and the VIRGO human vaginal non-redundant gene catalogue of ~0.95M genes, with ~0.62M genes in the VIRGO-90 clustered set). Build an MMseqs2 database for the reference catalogue and run an all-vs-all search of the query catalogue against it using 90% amino acid identity and 80% sequence coverage thresholds. Parse the MMseqs2 alignment output (m8/BLAST-tabular), marking each reference gene as covered if matched by any query protein and each query protein as shared if it matches any reference gene. Compute the overlap fractions — in the VMGC vs VIRGO case this yields ~70.6% of VIRGO-90 genes covered, ~30.8% of VMGC-90 genes shared, and a 162% relative protein-space expansion of VMGC-90 over VIRGO (Fig. 6b). The choice of 90% identity mirrors the VMGC-90 clustering threshold (genes clustered at 90% amino acid identity from ~26 million protein-coding genes), so results are comparable at that redundancy level.

## Related tools

- **MMseqs2** (All-vs-all protein alignment of VMGC-90 against the VIRGO-90 gene database at 90% amino acid identity and 80% sequence coverage thresholds)
- **GNU parallel 20201122** (Parallelization of the catalogue comparison and downstream parsing steps) — https://github.com/RChGO/VMGC

## Examples

```
mmseqs easy-search VMGC-90.faa VIRGO-90_db overlap.m8 tmp --min-seq-id 0.9 -c 0.8 --cov-mode 0
```

## Evaluation signals

- Percentage of VIRGO-90 genes covered by VMGC-90 matches ≈ 70.6% (within a small tolerance of the published figure, Fig. 6b)
- Percentage of VMGC-90 genes also present in VIRGO-90 ≈ 30.8%
- Relative protein-space expansion of VMGC-90 over VIRGO-90 ≈ 162%
- Every VIRGO-90 gene flagged as covered is backed by at least one MMseqs2 alignment row satisfying both identity ≥90% and coverage ≥80%
- Output TSV contains exactly the three statistics with consistent denominators (VIRGO-90 ≈ 0.62M genes)

## Limitations

- Overlap estimates are sensitive to the identity and coverage thresholds; changing 90%/80% changes the covered/shared fractions
- VMGC captures incomplete microbial diversity, especially for vaginal samples from non-western populations, which limits interpretation of catalogue expansion
- A 'shared' call depends on the completeness of both catalogues; genes missing from either source are unobservable

## Evidence

- [abstract] we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity: "we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity"
- [other] VMGC-90 covered about 70.6% of the genes in VIRGO-90, while only 30.8% of VMGC-90 genes were also present in VIRGO-90: [evidence span withheld — anchor status: not_found]
- [other] run an all-vs-all search of VMGC-90 proteins against it with 90% amino acid identity and 80% sequence coverage thresholds: [evidence span withheld — anchor status: not_found]
- [readme] The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006, including the data presented in the table below."
