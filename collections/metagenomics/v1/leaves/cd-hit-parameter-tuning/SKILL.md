---
name: cd-hit-parameter-tuning
description: Use when building a non-redundant gene catalog from a large set of predicted
  coding sequences (FASTA) and a near-duplicate collapse at a defined nucleotide identity/coverage
  threshold is required.
when_to_use_negative:
- The goal is protein-level orthologous grouping rather than nucleotide-level de-redundancy
  — VIRGO builds VOGs via all-vs-all BLASTP plus Jaccard clustering (linkscore 0.6,
  80% identity, 60% coverage) instead
- Sequences are already a curated non-redundant reference (e.g. the distributed VIRGO
  database being mapped against); re-clustering would alter the V-prefixed gene IDs
  used by downstream VIRGO modules
- Reads or abundance tables are the input — de-redundancy applies to predicted CDS
  nucleotide sequences prior to mapping
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CD-HIT-EST
  role: Greedy incremental nucleotide sequence clustering at -c 0.95 identity and
    -aS 0.90 shorter-sequence coverage to collapse redundant CDSs
- name: VIRGO
  role: Pipeline whose gene catalog construction uses this de-redundancy step; output
    V-prefixed gene IDs feed VIRGO annotation and mapping modules
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:701cce9bd454969e802124bf9cb2f2b3f1bd754c8a573d892d284a4f0b8bada8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-EST
  - VIRGO
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cd-hit-parameter-tuning

## Summary

Greedy nucleotide-level de-redundancy of predicted CDSs with CD-HIT-EST (>=95% identity over >=90% of the shorter gene) to build a non-redundant gene catalog, as used to reduce ~5.5 million vaginal metagenome CDSs to the 948,158-gene VIRGO catalog.

## When to use

Use when building a non-redundant gene catalog from a large set of predicted coding sequences (FASTA) and a near-duplicate collapse at a defined nucleotide identity/coverage threshold is required.

## When NOT to use

- The goal is protein-level orthologous grouping rather than nucleotide-level de-redundancy — VIRGO builds VOGs via all-vs-all BLASTP plus Jaccard clustering (linkscore 0.6, 80% identity, 60% coverage) instead
- Sequences are already a curated non-redundant reference (e.g. the distributed VIRGO database being mapped against); re-clustering would alter the V-prefixed gene IDs used by downstream VIRGO modules
- Reads or abundance tables are the input — de-redundancy applies to predicted CDS nucleotide sequences prior to mapping

## Inputs

- Predicted CDS nucleotide FASTA (~5.5 million coding sequences from assembled vaginal metagenomes)
- Minimum CDS length cutoff of 99 bp (sequences shorter are discarded before clustering)

## Outputs

- nr_gene_catalog.fasta containing 948,158 cluster-representative (longest) non-redundant CDS sequences with V-prefixed gene IDs
- CD-HIT-EST cluster file mapping redundant CDSs to representatives

## How to apply

Load the predicted CDS nucleotide FASTA (~5.5 million coding sequences assembled from 1,507 vaginal metagenomes plus urogenital isolate genomes) and first discard CDSs shorter than 99 bp to avoid spurious clusters of very short fragments. Run CD-HIT-EST v4.6 in greedy incremental mode with the sequence identity threshold set to -c 0.95 and the alignment coverage of the shorter sequence set to -aS 0.90, so clusters require >=95% identity over >=90% of the shorter gene length. Write the cluster representative (longest) sequences to nr_gene_catalog.fasta; this greedy pairwise de-redundancy yielded 948,158 non-redundant CDSs, ~17.2% of the original 5.5 million. Preserve the V-prefixed gene IDs so the catalog remains compatible with downstream VIRGO annotation and read-mapping modules.

## Related tools

- **CD-HIT-EST** (Greedy incremental nucleotide sequence clustering at -c 0.95 identity and -aS 0.90 shorter-sequence coverage to collapse redundant CDSs)
- **VIRGO** (Pipeline whose gene catalog construction uses this de-redundancy step; output V-prefixed gene IDs feed VIRGO annotation and mapping modules) — github.com/ravel-lab/VIRGO

## Examples

```
cd-hit-est -i predicted_cds.fasta -o nr_gene_catalog.fasta -c 0.95 -aS 0.90 -g 1 -T 8
```

## Evaluation signals

- Output FASTA contains exactly 948,158 sequences when run on the full VIRGO CDS input (~17.2% of the original 5.5 million predicted CDSs)
- All retained sequences are >=99 bp after the minimum-length prefilter
- Every cluster representative is the longest member of its cluster (CD-HIT-EST greedy incremental behavior)
- Gene IDs retain the V-prefixed format (e.g. V1593031) so VIRGO annotation files (0.geneLength, 1.taxon.table, proteinFamily, EC, KEGG files) resolve against the catalog
- No two representatives in the output share >=95% nucleotide identity over >=90% of the shorter sequence

## Limitations

- Gene-level de-redundancy only: this version of VIRGO is focused on gene-level de-redundancy and does not yet identify nucleotide variants within a gene
- The reference-based approach has acknowledged limitations for characterizing metagenomic subspecies and accessory-gene diversity beyond the catalog
- Discarding CDSs <99 bp removes very short fragments, which may exclude some genuine short genes from the catalog

## Evidence

- [results] The greedy pairwise nucleotide-level de-redundancy procedure (95% identity over 90% of the shorter gene length) yielded a catalog of 948,158 non-redundant CDSs: [evidence span withheld — anchor status: not_found]
- [abstract] we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes: "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes"
- [readme] VIRGO is a non-redundant gene catalog for the microbial communities that inhabit the human vagina.: "VIRGO is a non-redundant gene catalog for the microbial communities that inhabit the human vagina."
- [methods] This module demonstrates how to map reads from a metagenome or metatranscriptome sample to VIRGO: "This module demonstrates how to map reads from a metagenome or metatranscriptome sample to VIRGO and present the results."
