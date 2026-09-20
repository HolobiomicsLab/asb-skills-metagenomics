---
name: reference-database-preprocessing-for-otu-annotation
description: Use when a 16S reference database FASTA (Greengenes or SILVA) must be
  prepared for CD-HIT-OTU-Miseq / CDSnake OTU assignment, or when cluster representatives
  must be matched to the most specific available taxonomy lineages.
when_to_use_negative:
- The database is already in CD-HIT-OTU-Miseq format — use the authors' pre-reformatted
  Greengenes/SILVA downloads instead of re-splicing.
- The downstream pipeline does not match cluster representatives against an ordered
  annotated database (e.g., QIIME2-DADA2 or deblur), where record ordering provides
  no benefit.
- The task is read-level preprocessing (quality cropping with Trimmomatic, chimeric
  read removal), which operates on FASTQ reads, not on the reference database.
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: CDSnake
  role: Snakemake pipeline whose Snakefile DATABASE parameter points to the reformatted
    16S reference database used to annotate OTU clusters
  repo: github.com/ydkondratenko/cdsnake
- name: CD-HIT-OTU-Miseq
  role: 'Consumer of the reformatted database: clusters reads and database fragments
    at 97% similarity so cluster representatives receive annotations from the reordered
    reference'
- name: Snakemake
  role: Workflow engine that launches the CD-HIT-OTU-Miseq steps using the formatted
    database
  repo: github.com/ydkondratenko/cdsnake
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:f3a15454ea0c42f210b8720996ae5afd9495acb6235941d9e8f37aa1668039b4
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
n_claims: 8
---

# reference-database-preprocessing-for-otu-annotation

## Summary

Reformats annotated 16S reference FASTA databases (e.g., Greengenes gg_13_8, SILVA SSU) by reordering records so that sequences carrying more specific taxonomic annotations appear at the beginning of the file, as required by CD-HIT-OTU-Miseq's cluster-annotation stage in the CDSnake pipeline.

## When to use

Use when a 16S reference database FASTA (Greengenes or SILVA) must be prepared for CD-HIT-OTU-Miseq / CDSnake OTU assignment, or when cluster representatives must be matched to the most specific available taxonomy lineages.

## When NOT to use

- The database is already in CD-HIT-OTU-Miseq format — use the authors' pre-reformatted Greengenes/SILVA downloads instead of re-splicing.
- The downstream pipeline does not match cluster representatives against an ordered annotated database (e.g., QIIME2-DADA2 or deblur), where record ordering provides no benefit.
- The task is read-level preprocessing (quality cropping with Trimmomatic, chimeric read removal), which operates on FASTQ reads, not on the reference database.

## Inputs

- Annotated 16S reference database FASTA file (e.g., Greengenes gg_13_8, SILVA SSU) with taxonomy lineage strings in FASTA headers

## Outputs

- Reformatted 16S reference database FASTA with more specifically annotated sequences placed at the beginning of the file, ready for the CD-HIT-OTU-Miseq OTU-assignment stage

## How to apply

Parse each record of the annotated microbial 16S FASTA (e.g., Greengenes gg_13_8 or SILVA SSU) and extract the taxonomic lineage string embedded in its header. Rank each record by annotation specificity (lineage depth or presence of genus/species-level names) and rewrite the FASTA so records with more specific annotations precede records annotated only at higher taxonomic levels, preserving original sequences and headers. This ordering matters because in the CDSnake pipeline, remaining reads and database fragments are clustered at 97% similarity and cluster representative sequences are annotated against this database, so specific lineages encountered first win the annotation. Alternatively, download the pre-reformatted Greengenes and SILVA files provided by the CDSnake authors and specify one via the DATABASE parameter in the Snakefile. Validate by confirming the reformatted file contains the same number of records as the source and spot-checking that specific lineages precede generic ones.

## Related tools

- **CDSnake** (Snakemake pipeline whose Snakefile DATABASE parameter points to the reformatted 16S reference database used to annotate OTU clusters) — github.com/ydkondratenko/cdsnake
- **CD-HIT-OTU-Miseq** (Consumer of the reformatted database: clusters reads and database fragments at 97% similarity so cluster representatives receive annotations from the reordered reference)
- **Snakemake** (Workflow engine that launches the CD-HIT-OTU-Miseq steps using the formatted database) — github.com/ydkondratenko/cdsnake

## Examples

```
snakemake  # after setting DATABASE=gg_13_8_reformatted.fasta in the Snakefile placed in the input folder containing paired-end .fastq reads
```

## Evaluation signals

- Reformatted FASTA contains exactly the same number of records as the source database file.
- Sequences and headers are byte-identical to the source; only record order changed.
- Spot-check confirms records with deeper lineages (genus/species-level names) precede records annotated only at higher levels.
- Downstream run of CDSnake completes without database-format errors and produces an OTU.txt table in the 'pooled' folder with database-derived annotations.
- Output matches the authors' downloadable pre-reformatted Greengenes/SILVA files in record ordering.

## Limitations

- The exact reformatting algorithm lives in step 4 of the CD-HIT-OTU-Miseq installation; CDSnake's README only references it rather than restating the procedure.
- Annotation specificity depends on the lineage strings present in Greengenes/SILVA headers; records with equally specific annotations have no defined ordering guarantee.
- No software versions or checksums are stated for the reformatted database downloads in the article or repository.

## Evidence

- [readme] Note, that database should be formatted as described in 4 step of Installation of CD-HIT-OTU-Miseq (this reformatting places sequences with more specific annotations are at the beginning of the file).: "Note, that database should be formatted as described in 4 step of Installation of CD-HIT-OTU-Miseq (this reformatting places sequences with more specific annotations are at the beginning of the file)."
- [readme] You can download reformatted versions of Greengenes and Silva databases here.: "You can download reformatted versions of Greengenes and Silva databases here."
- [methods] database of annotated microbial 16S sequences (for instance, Greengenes [5] or SILVA [6]) in form of fasta files: "database of annotated microbial 16S sequences (for instance, Greengenes [5] or SILVA [6]) in form of fasta files"
- [methods] Remaining reads and fragments from annotated database are then clustered at 97% similarity to yields annotated clusters: "Remaining reads and fragments from annotated database are then clustered at 97% similarity to yields annotated clusters"
