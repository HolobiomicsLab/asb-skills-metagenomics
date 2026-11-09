---
name: gene-length-annotation
description: Use when building or extending the VIRGO non-redundant gene catalog and
  you need per-gene CDS predictions and gene lengths in bp from assembled metagenomic
  contigs or isolate genome sequences.
when_to_use_negative:
- Input is raw metagenomic/metatranscriptomic reads to be profiled against an existing
  VIRGO database — use the read-mapping module (runMapping.step1.sh) instead of re-predicting
  genes
- You need functional or taxonomic annotation for existing VIRGO genes (e.g. eggNOG,
  KEGG, Pfam) — pull from 1_VIRGO annotation files rather than re-running gene prediction
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaGeneMark
  role: CDS prediction tool (version 3.25, default parameters) run on metagenomic
    contigs and isolate genomes to generate predicted CDSs and coordinates for length
    computation
- name: VIRGO
  role: Non-redundant vaginal gene catalog whose annotation directory (1_VIRGO/0.geneLength.txt)
    stores the per-gene bp lengths produced by this skill
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:d77443b80cb502f80971fb9c6f97aeb37b5a51c3435835b1ab77b475729a13f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaGeneMark
  - VIRGO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gene-length-annotation

## Summary

Predicts coding sequences on the vaginal metagenome contig assemblies and urogenital isolate genomes used to build the VIRGO gene catalog with MetaGeneMark, then compiles each predicted gene's length in bp into the VIRGO annotation file 0.geneLength.txt, keyed by VIRGO gene ID.

## When to use

Use when building or extending the VIRGO non-redundant gene catalog and you need per-gene CDS predictions and gene lengths in bp from assembled metagenomic contigs or isolate genome sequences.

## When NOT to use

- Input is raw metagenomic/metatranscriptomic reads to be profiled against an existing VIRGO database — use the read-mapping module (runMapping.step1.sh) instead of re-predicting genes
- You need functional or taxonomic annotation for existing VIRGO genes (e.g. eggNOG, KEGG, Pfam) — pull from 1_VIRGO annotation files rather than re-running gene prediction

## Inputs

- Assembled vaginal metagenome contigs (FASTA)
- Urogenital bacteria isolate genome sequence assemblies (FASTA)

## Outputs

- 0.geneLength.txt: per-gene length table mapping each VIRGO gene ID to its length in bp
- CDS coordinates parsed from MetaGeneMark predictions, attributed to metagenomic assembly vs. isolate genome source

## How to apply

Retrieve the assembled vaginal metagenome contigs and the urogenital isolate genome assemblies that feed the gene catalog, then run MetaGeneMark version 3.25 in gene-prediction mode with default parameters on each contig set. Parse the MetaGeneMark prediction output to extract CDS coordinates and compute each predicted gene's length in base pairs. Attribute each predicted CDS to its assembly source (metagenomic assembly vs. isolate genome) and tally counts per source; in the published catalog this yielded 5,509,298 CDSs, with roughly 80% from metagenomic assemblies and ~20% from isolate genomes. Write the per-gene length annotation table as 0.geneLength.txt, formatted as VIRGO gene ID followed by length (e.g. 'V1000057 990'), consistent with the 1_VIRGO annotation directory layout so downstream modules (taxonomy, eggNOG, KEGG annotations) can join on gene ID.

## Related tools

- **MetaGeneMark** (CDS prediction tool (version 3.25, default parameters) run on metagenomic contigs and isolate genomes to generate predicted CDSs and coordinates for length computation)
- **VIRGO** (Non-redundant vaginal gene catalog whose annotation directory (1_VIRGO/0.geneLength.txt) stores the per-gene bp lengths produced by this skill) — github.com/ravel-lab/VIRGO

## Examples

```
grep -w "V1000057" 1_VIRGO/*
```

## Evaluation signals

- Total predicted CDS count across all assemblies is approximately 5,509,298
- Source attribution split is roughly 80% of CDSs from metagenomic assemblies and ~20% from urogenital isolate genomes
- 0.geneLength.txt rows follow the VIRGO ID + length format (e.g. 'V1000057 990') and join cleanly with other 1_VIRGO files such as 1.taxon.tbl.txt on gene ID
- Every reported gene length is a positive integer consistent with the CDS start/stop coordinates in the MetaGeneMark output
- Every predicted CDS is attributed to exactly one assembly source (metagenomic assembly or isolate genome) with no unassigned entries

## Limitations

- VIRGO's reference-based approach is acknowledged as a limitation; this version focuses on gene-level de-redundancy of the vaginal microbiome
- The database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences (fungal/phage genes are annotated separately in 2_fungal_phage)
- Gene lengths are for predicted CDSs only; nucleotide-variant identification within genes was not yet included in VIRGO

## Evidence

- [methods] CDS prediction was performed on the metagenomic assemblies and genome sequences using MetaGeneMark, yielding 5,509,298 predicted CDSs: [evidence span withheld — anchor status: not_found]
- [readme] 0.geneLength contains gene length in bp: "***0.geneLength*** contains gene length in bp"
- [readme] It is easy to quickly pull annotations for your favorite gene, using a single VIRGO geneID!: "It is easy to quickly pull annotations for your favorite gene, using a single VIRGO geneID!"
