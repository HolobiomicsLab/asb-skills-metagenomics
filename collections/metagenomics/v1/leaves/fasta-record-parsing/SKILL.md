---
name: fasta-record-parsing
description: Use when a FASTA reference database of annotated microbial 16S sequences
  must be reformatted for CD-HIT-OTU-Miseq/CDSnake, i.e., when headers embed taxonomy
  lineages and record order must reflect annotation specificity.
when_to_use_negative:
- The database is already a pre-reformatted Greengenes or SILVA file downloaded from
  the CDSnake repository — no re-parsing needed
- Input is paired-end FASTQ reads; this skill only applies to annotated reference
  database FASTA files, not read data
- The downstream pipeline does not scan the database in file order for annotation
  (e.g., QIIME2-DADA2 or deblur assignment), so the specificity-based reordering has
  no effect
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: CD-HIT-OTU-Miseq
  role: Consumer of the reformatted database; its Installation step 4 defines the
    required reformatting that places more-specific annotations first
- name: CDSnake
  role: Snakemake pipeline whose DATABASE parameter points to the reformatted 16S
    database used for OTU annotation
  repo: github.com/ydkondratenko/cdsnake
- name: Snakemake
  role: Workflow engine that orchestrates the pipeline step consuming the reformatted
    database
  repo: github.com/ydkondratenko/cdsnake
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:e5cd488f1ed3e491d9bf9f5b3c1aaec08f9ee44428e8c1adfb1a9f636ac767c8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# fasta-record-parsing

## Summary

Parse FASTA records of an annotated 16S reference database (Greengenes or SILVA SSU), extract the taxonomic lineage from each header, and reorder records so sequences with more specific annotations precede generic ones, as required by CD-HIT-OTU-Miseq's database-annotation stage.

## When to use

Use when a FASTA reference database of annotated microbial 16S sequences must be reformatted for CD-HIT-OTU-Miseq/CDSnake, i.e., when headers embed taxonomy lineages and record order must reflect annotation specificity.

## When NOT to use

- The database is already a pre-reformatted Greengenes or SILVA file downloaded from the CDSnake repository — no re-parsing needed
- Input is paired-end FASTQ reads; this skill only applies to annotated reference database FASTA files, not read data
- The downstream pipeline does not scan the database in file order for annotation (e.g., QIIME2-DADA2 or deblur assignment), so the specificity-based reordering has no effect

## Inputs

- FASTA file of annotated microbial 16S reference sequences (Greengenes, e.g., gg_13_8)
- FASTA file of annotated SILVA SSU 16S reference sequences
- Taxonomic lineage/annotation strings embedded in FASTA headers

## Outputs

- Reformatted database FASTA file with sequences carrying more specific taxonomic annotations placed at the beginning of the file
- Validation report confirming record-count parity with the source file and correct ordering of specific vs. generic lineages

## How to apply

Load the Greengenes (e.g., gg_13_8) or SILVA SSU FASTA file of annotated microbial 16S sequences and iterate over each FASTA record, extracting the taxonomic lineage/annotation string embedded in the header. Rank each record by annotation specificity (e.g., lineage depth or presence of genus- and species-level names) to build a sort key. Write the records in reordered form — more-specific annotations at the beginning of the file, generic higher-level-only annotations later — preserving the original sequences and headers verbatim. This ordering is mandated because CD-HIT-OTU-Miseq assigns database annotations to 97%-similarity OTU clusters by scanning the file, so earlier, more specific records win the annotation. Validate by checking record counts match the source file and spot-checking that specific lineages precede generic ones.

## Related tools

- **CD-HIT-OTU-Miseq** (Consumer of the reformatted database; its Installation step 4 defines the required reformatting that places more-specific annotations first)
- **CDSnake** (Snakemake pipeline whose DATABASE parameter points to the reformatted 16S database used for OTU annotation) — github.com/ydkondratenko/cdsnake
- **Snakemake** (Workflow engine that orchestrates the pipeline step consuming the reformatted database) — github.com/ydkondratenko/cdsnake

## Evaluation signals

- Reformatted FASTA contains exactly the same number of records as the source Greengenes/SILVA file
- Spot-check confirms records with genus- or species-level lineage annotations precede records annotated only at higher taxonomic levels
- Original sequences and headers are preserved byte-for-byte; only record order changes
- CDSnake run with DATABASE set to the reformatted file completes and produces an OTU.txt annotation table in the 'pooled' folder
- Output matches the pre-reformatted Greengenes and SILVA databases offered for download alongside CDSnake

## Limitations

- The reformatting is only meaningful for databases whose headers embed taxonomic annotations; unannotated FASTA files cannot be ranked by lineage specificity
- CDSnake reads must be unzipped '.fastq' with '_R1_'/'_R2_' (or '_1'/'_2') naming, but no checksums or exact versions of the reformatted databases are provided
- The ordering rationale is specific to CD-HIT-OTU-Miseq's cluster-annotation scan; other pipelines may ignore record order

## Evidence

- [readme] database should be formatted as described in 4 step of Installation of CD-HIT-OTU-Miseq (this reformatting places sequences with more specific annotations are at the beginning of the file): "Note, that database should be formatted as described in 4 step of Installation of CD-HIT-OTU-Miseq (this reformatting places sequences with more specific annotations are at the beginning of the file)."
- [methods] database of annotated microbial 16S sequences (for instance, Greengenes [5] or SILVA [6]) in form of fasta files: "Pipeline takes as input a folder with paired-end reads in form of fastq files and database of annotated microbial 16S sequences (for instance, Greengenes [5] or SILVA [6]) in form of fasta files"
- [readme] You can download reformatted versions of Greengenes and Silva databases here.: "You can download reformatted versions of Greengenes and Silva databases here."
- [intro] matching the resulting clusters’ representative sequences with annotated database: "which commonly includes clustering of sequences and matching the resulting clusters’ representative sequences with annotated database"
