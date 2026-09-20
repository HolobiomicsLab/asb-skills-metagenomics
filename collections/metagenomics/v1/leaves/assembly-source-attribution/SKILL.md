---
name: assembly-source-attribution
description: Use when building or extending a non-redundant gene catalog from mixed
  inputs (e.g., vaginal metagenome assemblies plus urogenital isolate genomes) and
  you need predicted CDS counts and per-gene lengths attributed to each assembly source.
when_to_use_negative:
- Input is raw reads rather than assembled contigs/genome sequences — gene prediction
  here is run on assemblies, not FASTQ files
- You need taxonomic/functional annotation (eggNOG, KEGG, protein families) for existing
  catalog genes — use the VIRGO annotation lookup modules instead
- Your target microbes are viral or fungal — the catalog this feeds is primarily focused
  on bacteria with limited inclusion of viral and fungal gene sequences
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaGeneMark
  role: Ab initio gene-prediction tool (version 3.25, default parameters) run on metagenomic
    contigs and isolate genome assemblies to predict CDSs
- name: VIRGO
  role: The resulting non-redundant gene catalog whose 1_VIRGO/0.geneLength.txt file
    stores the predicted gene lengths produced by this step
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:36e6c15505591ead7ac732415f93f226af54c319ec7400f4b64e394e454cd7b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaGeneMark
  - VIRGO
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# assembly-source-attribution

## Summary

Predicts coding sequences (CDSs) from assembled metagenomic contigs and isolate genome assemblies using MetaGeneMark, then attributes each predicted gene to its assembly source to quantify each contribution to a gene catalog. This produced the per-gene length annotation file underlying the VIRGO human vaginal non-redundant gene catalog.

## When to use

Use when building or extending a non-redundant gene catalog from mixed inputs (e.g., vaginal metagenome assemblies plus urogenital isolate genomes) and you need predicted CDS counts and per-gene lengths attributed to each assembly source.

## When NOT to use

- Input is raw reads rather than assembled contigs/genome sequences — gene prediction here is run on assemblies, not FASTQ files
- You need taxonomic/functional annotation (eggNOG, KEGG, protein families) for existing catalog genes — use the VIRGO annotation lookup modules instead
- Your target microbes are viral or fungal — the catalog this feeds is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences

## Inputs

- Assembled vaginal metagenome contigs (FASTA)
- Urogenital bacteria isolate genome sequence assemblies (FASTA)

## Outputs

- Per-gene length annotation table (0.geneLength.txt: VIRGO gene ID and length in bp)
- Predicted CDS coordinates from MetaGeneMark output
- Per-source tally of predicted CDSs (metagenomic assemblies vs. isolate genomes)

## How to apply

Retrieve the assembled vaginal metagenome contigs and the urogenital isolate genome sequence assemblies used to build the catalog. Run MetaGeneMark version 3.25 in gene-prediction mode with default parameters separately on each set of contigs to predict CDSs. Parse the MetaGeneMark prediction output to extract CDS coordinates and compute each predicted gene's length in bp. Attribute every predicted CDS to its assembly source (metagenomic assembly vs. isolate genome) and tally counts per source. Write the per-gene length annotation table as 0.geneLength.txt, the VIRGO convention for storing gene lengths (e.g., V1000057 990) that downstream mapping and annotation modules consume.

## Related tools

- **MetaGeneMark** (Ab initio gene-prediction tool (version 3.25, default parameters) run on metagenomic contigs and isolate genome assemblies to predict CDSs)
- **VIRGO** (The resulting non-redundant gene catalog whose 1_VIRGO/0.geneLength.txt file stores the predicted gene lengths produced by this step) — github.com/ravel-lab/VIRGO

## Evaluation signals

- Total predicted CDS count is approximately 5,509,298
- Source-attribution tally shows ~80% of predicted CDSs from metagenomic assemblies and ~20% from urogenital isolate genome sequences
- 0.geneLength.txt is a two-column table of VIRGO gene ID and length in bp (e.g., 'V1000057 990'), consistent with the documented 1_VIRGO directory structure
- Every predicted gene ID in the length table traces back to a valid MetaGeneMark prediction on exactly one assembly source

## Limitations

- VIRGO acknowledges limitations of its reference-based approach and is focused on gene-level de-redundancy of the vaginal microbiome
- The catalog is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences
- MetaGeneMark predictions on metagenomic contigs may include genes not observed in isolate genome sequences for the same species, complicating one-to-one attribution

## Evidence

- [methods] CDS prediction was performed on the metagenomic assemblies and genome sequences using MetaGeneMark, yielding 5,509,298 predicted CDSs: [evidence span withheld — anchor status: not_found]
- [readme] VIRGO built from metagenomic and isolate genome sequences: "It was built using a combination of metagenomic and urogential isolate genome sequences."
- [readme] 0.geneLength contains gene length in bp: "***0.geneLength*** contains gene length in bp"
