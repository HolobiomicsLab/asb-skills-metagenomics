---
name: mmseqs2-homology-search
description: Use when you need to quantify overlap between two protein/gene catalogues
  (e.g. VMGC-90 vs VIRGO-90) by all-vs-all similarity search at defined amino acid
  identity and coverage thresholds.
when_to_use_negative:
- Input is a nucleotide-level comparison or read-level taxonomic profiling task (use
  Kraken2/Bracken custom SGB databases instead)
- You need genome-level clustering of MAGs at 95% nucleotide similarity (species-level
  genome binning), not protein catalogue overlap
- The target catalogue lacks protein-coding gene predictions — this skill operates
  on protein sequences at amino acid identity thresholds
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: MMseqs2
  role: Builds the VIRGO-90 sequence database and performs the all-vs-all protein
    search of VMGC-90 against it at 90% identity / 80% coverage
- name: GNU parallel 20201122
  role: Parallelises the parsing and per-file processing steps of the homology comparison
    workflow
  repo: https://github.com/RChGO/VMGC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:dd3a579b2cf5f3db36b13b3402ff9d6d0fefcf97a9a4f3ad49078679379d7278
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RChGO/VMGC
  tools:
  - GNU parallel 20201122
  - MMseqs2
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# mmseqs2-homology-search

## Summary

Run large-scale protein homology searches between gene catalogues with MMseqs2 to quantify mutual coverage, as used to compare the VMGC-90 vaginal gene catalogue against the published VIRGO-90 catalogue (0.95 million genes; VIRGO-90 subset 0.62M genes).

## When to use

Use when you need to quantify overlap between two protein/gene catalogues (e.g. VMGC-90 vs VIRGO-90) by all-vs-all similarity search at defined amino acid identity and coverage thresholds.

## When NOT to use

- Input is a nucleotide-level comparison or read-level taxonomic profiling task (use Kraken2/Bracken custom SGB databases instead)
- You need genome-level clustering of MAGs at 95% nucleotide similarity (species-level genome binning), not protein catalogue overlap
- The target catalogue lacks protein-coding gene predictions — this skill operates on protein sequences at amino acid identity thresholds

## Inputs

- VMGC-90 protein catalogue (from VMGC microbial sequence data, Zenodo record 10457006)
- VIRGO-90 human vaginal non-redundant gene catalogue (0.62M genes clustered at 90% amino acid identity)
- Threshold parameters: 90% amino acid identity, 80% sequence coverage

## Outputs

- MMseqs2 all-vs-all alignment output (hits passing 90% identity / 80% coverage)
- vmgc_virgo90_overlap_stats.tsv with VIRGO-90 coverage %, VMGC-90 shared %, and protein-space expansion %
- Per-gene coverage/shared flags for both catalogues

## How to apply

Download the VMGC microbial sequence data containing the VMGC-90 protein catalogue from Zenodo record 10457006 and obtain the VIRGO human vaginal non-redundant gene catalogue. Build an MMseqs2 database for the VIRGO-90 gene set (0.62M genes) and run an all-vs-all search of VMGC-90 proteins against it, using 90% amino acid identity and 80% sequence coverage thresholds to call a hit. Parse the MMseqs2 alignment output, marking each VIRGO-90 gene as covered if matched by any VMGC-90 protein, and each VMGC-90 protein as shared if it matches a VIRGO-90 gene. Compute the overlap fractions — VIRGO-90 coverage (expected 70.6%), VMGC-90 shared (expected 30.8%) — and derive the protein-space expansion of VMGC-90 over VIRGO (expected 162%), writing the statistics to vmgc_virgo90_overlap_stats.tsv. The 90% identity clustering mirrors how the paper built VMGC-90 from ~26 million protein-coding genes clustered at 50%, 90% and 95% amino acid identity.

## Related tools

- **MMseqs2** (Builds the VIRGO-90 sequence database and performs the all-vs-all protein search of VMGC-90 against it at 90% identity / 80% coverage)
- **GNU parallel 20201122** (Parallelises the parsing and per-file processing steps of the homology comparison workflow) — https://github.com/RChGO/VMGC

## Examples

```
mmseqs createdb VIRGO90.faa virgo90_db; mmseqs createdb VMGC90.faa vmgc90_db; mmseqs search vmgc90_db virgo90_db results_db tmp --min-seq-id 0.9 -c 0.8; mmseqs convertalis vmgc90_db virgo90_db results_db vmgc90_vs_virgo90.m8
```

## Evaluation signals

- Percentage of VIRGO-90 genes covered by VMGC-90 matches ~70.6% (Fig. 6b), within a small tolerance of the published figure
- Percentage of VMGC-90 genes also present in VIRGO-90 ~30.8%, within tolerance of the published figure
- Relative protein-space expansion of VMGC-90 over VIRGO ~162%
- vmgc_virgo90_overlap_stats.tsv contains exactly the three statistics with plausible fractions (each between 0 and 100, expansion consistent with the two coverage fractions)
- Hit counts consistent with catalogue sizes (VIRGO total 0.95 million genes; VIRGO-90 subset 0.62M genes)

## Limitations

- VMGC and its gene catalogues are incomplete, especially for vaginal samples from non-western populations, so coverage fractions are lower bounds on true overlap
- Overlap statistics depend sensitively on the identity/coverage cutoffs (90% identity, 80% coverage); different thresholds will yield different fractions
- Many genes and phages in VMGC are unknown and functionally uncharacterised, so matched vs unmatched genes cannot be functionally distinguished

## Evidence

- [other] VMGC-90 covered about 70.6% of the genes in VIRGO-90, while only 30.8% of VMGC-90 genes were also present in VIRGO-90: [evidence span withheld — anchor status: not_found]
- [abstract] Protein-coding gene clustering at 50/90/95% amino acid identity: "we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity"
- [abstract] VIRGO catalogue size: "the human vaginal non-redundant gene catalogue (VIRGO)10, which includes a total of 0.95 million genes"
- [intro] Data availability for VMGC: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
- [readme] GNU parallel dependency in VMGC pipelines: "> GNU parallel 20201122<br>"
