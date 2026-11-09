---
name: protein-annotation-filtering
description: Use when building a reference protein database from multi-source fungal
  coding genes (NCBI, JGI, UniProt) where unannotated proteins must be removed and
  redundancy reduced before functional profiling.
when_to_use_negative:
- Constructing the taxonomic marker-gene database (FunOMIC-T), which requires 99%
  identity CD-HIT clustering of single-copy marker genes, not annotated-protein filtering
- Input proteins are already functionally annotated and de-redundicated, or the analysis
  requires retaining unannotated hypothetical proteins
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CD-HIT
  role: Performs the 95% identity clustering step that removes redundant annotated
    fungal proteins
- name: FunOMIC
  role: Pipeline whose built-in FunOMIC-P functional database is produced by this
    filtering and clustering workflow; scripts link accessions to EC/KEGG terms
  repo: https://github.com/ManichanhLab/FunOMIC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:c98cea93bdcf5ce7567b2d6e9c616924f4fdf2d673ef3ab412479bbfcaa0286c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - CD-HIT
  - FunOMIC
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# protein-annotation-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a curated non-redundant functional protein database (FunOMIC-P) by discarding fungal proteins lacking explicit functional annotation and clustering the annotated set with CD-HIT at a 95% identity threshold.

## When to use

Use when building a reference protein database from multi-source fungal coding genes (NCBI, JGI, UniProt) where unannotated proteins must be removed and redundancy reduced before functional profiling.

## When NOT to use

- Constructing the taxonomic marker-gene database (FunOMIC-T), which requires 99% identity CD-HIT clustering of single-copy marker genes, not annotated-protein filtering
- Input proteins are already functionally annotated and de-redundicated, or the analysis requires retaining unannotated hypothetical proteins

## Inputs

- Combined FASTA of fungal protein sequences from NCBI and JGI genome coding genes (2,967 quality-filtered genomes)
- 35,360 reviewed fungal proteins from the UniProt database
- Functional annotations (EC numbers, KEGG pathways) associated with each protein accession

## Outputs

- FunOMIC-P FASTA containing 3,413,239 non-redundant annotated fungal protein sequences
- Cluster summary table reporting reduction from ~4.9 million input genes after 95% identity clustering
- Protein accession-to-EC-number and KEGG pathway mapping for functional annotation

## How to apply

Combine amino-acid sequences from quality-filtered fungal genomes (2,967 of 4,331 genomes used for the functional database) with reviewed fungal proteins from UniProt to form a ~4.9 million gene protein set. First discard every protein lacking an explicit functional annotation (~1.5 million proteins removed), since the downstream purpose is EC-number and KEGG pathway linkage. Then run CD-HIT with a 95% identity clustering threshold to remove near-duplicate sequences arising from overlapping genome sources. Emit the surviving 3,413,239 non-redundant fungal proteins as the FunOMIC-P FASTA and link protein accessions to EC numbers and KEGG pathways. Note that 95% identity is deliberately stricter (lower threshold) than the 99% threshold used for the taxonomic marker-gene database FunOMIC-T.

## Related tools

- **CD-HIT** (Performs the 95% identity clustering step that removes redundant annotated fungal proteins)
- **FunOMIC** (Pipeline whose built-in FunOMIC-P functional database is produced by this filtering and clustering workflow; scripts link accessions to EC/KEGG terms) — https://github.com/ManichanhLab/FunOMIC

## Examples

```
cd-hit -i combined_annotated_fungal_proteins.fasta -o funomic_p.fasta -c 0.95
```

## Evaluation signals

- Output FASTA contains exactly 3,413,239 non-redundant fungal protein sequences
- Cluster summary table confirms reduction from the 4.9 million input genes after 95% identity clustering
- Approximately 1.5 million proteins without explicit annotation were discarded before clustering
- Every retained protein accession (from JGI, NCBI, UniProt) resolves to an EC number and/or KEGG pathway entry
- CD-HIT was invoked with a 95% (0.95) identity threshold, not the 99% threshold reserved for FunOMIC-T marker genes

## Limitations

- Discarding all proteins without explicit annotation excludes uncharacterized/hypothetical proteins from the functional database, reducing coverage
- Redundancy is only removed at 95% identity, so more divergent homologs (including possible bacterial-like proteins) remain; the article notes bacterial contamination was not totally preventable at the functional-annotation stage even with strict DIAMOND thresholds (coverage >95%, identity >99%, e-value < 10e-10)
- The 4.9 million gene pool depends on the quality of collected fungal genomes from NCBI and JGI MycoCosm

## Evidence
