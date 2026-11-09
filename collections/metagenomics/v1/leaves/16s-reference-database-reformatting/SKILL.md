---
name: 16s-reference-database-reformatting
description: Use when preparing an annotated microbial 16S FASTA database (Greengenes
  or SILVA) as input to the CDSnake/CD-HIT-OTU-Miseq pipeline, whose DATABASE parameter
  expects the reordered format.
when_to_use_negative:
- The database is already a pre-reformatted Greengenes or SILVA file downloaded from
  the CDSnake authors — re-reordering is unnecessary
- The target pipeline does not use CD-HIT-OTU-Miseq annotation assignment (e.g., DADA2
  or deblur workflows with QIIME2 classifiers), where this specific reordering provides
  no benefit
- The input FASTA records lack taxonomic annotations in their headers, so no specificity-based
  sort key can be computed
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: CD-HIT-OTU-Miseq
  role: Defines the required database formatting (step 4 of its installation) and
    consumes the reordered FASTA when matching cluster representative sequences to
    annotations
- name: CDSnake
  role: Pipeline that takes the reformatted database via its DATABASE parameter in
    the Snakefile
  repo: github.com/ydkondratenko/cdsnake
- name: Snakemake
  role: Workflow engine in which the reformatted database is wired into the CDSnake
    Snakefile
  repo: github.com/ydkondratenko/cdsnake
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:6630eaf1fb0171c336d35872ddda40090db3bdef3bfc95cecec60312d9f896dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - Snakemake
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# 16s-reference-database-reformatting

## Summary

Reformats 16S reference databases (e.g., Greengenes gg_13_8, SILVA SSU) so that FASTA records carrying more specific taxonomic annotations are placed at the beginning of the file, as required by CD-HIT-OTU-Miseq for annotated OTU assignment in the CDSnake pipeline.

## When to use

Use when preparing an annotated microbial 16S FASTA database (Greengenes or SILVA) as input to the CDSnake/CD-HIT-OTU-Miseq pipeline, whose DATABASE parameter expects the reordered format.

## When NOT to use

- The database is already a pre-reformatted Greengenes or SILVA file downloaded from the CDSnake authors — re-reordering is unnecessary
- The target pipeline does not use CD-HIT-OTU-Miseq annotation assignment (e.g., DADA2 or deblur workflows with QIIME2 classifiers), where this specific reordering provides no benefit
- The input FASTA records lack taxonomic annotations in their headers, so no specificity-based sort key can be computed

## Inputs

- Annotated microbial 16S reference database in FASTA format (e.g., Greengenes gg_13_8 or SILVA SSU) with taxonomy embedded in record headers
- 16S-ref-db-PE-splice.pl reordering script (step 4 of CD-HIT-OTU-Miseq installation)

## Outputs

- Reordered FASTA file containing the same records and headers, with more-specific taxonomic annotations placed at the beginning of the file, ready as the CDSnake DATABASE parameter

## How to apply

Load the annotated 16S FASTA (e.g., Greengenes or SILVA SSU) and parse each record's header to extract the embedded taxonomic lineage string. Rank records by annotation specificity (lineage depth, presence of genus- or species-level names) as the sort key. Reorder records so that sequences with more specific annotations precede generic, higher-level-only ones, preserving original sequences and headers. This ordering matters because at the 97% clustering step, matched OTU clusters receive database annotations from the representative sequence they first match; more-specific annotations earlier in the file take precedence. Alternatively, use the pre-reformatted Greengenes and SILVA databases provided by the CDSnake authors, and set the reformatted file name in the Snakefile DATABASE parameter.

## Related tools

- **CD-HIT-OTU-Miseq** (Defines the required database formatting (step 4 of its installation) and consumes the reordered FASTA when matching cluster representative sequences to annotations)
- **CDSnake** (Pipeline that takes the reformatted database via its DATABASE parameter in the Snakefile) — github.com/ydkondratenko/cdsnake
- **Snakemake** (Workflow engine in which the reformatted database is wired into the CDSnake Snakefile) — github.com/ydkondratenko/cdsnake

## Examples

```
perl 16S-ref-db-PE-splice.pl gg_13_8_otus.fasta gg_13_8_otus_formatted.fasta  # then set DATABASE=gg_13_8_otus_formatted.fasta in Snakefile; snakemake
```

## Evaluation signals

- Reformatted FASTA contains exactly the same number of records as the source file
- Record sequences and headers are byte-identical to the originals — only the order changed
- Spot-check confirms records with genus- or species-level lineages appear before records annotated only at higher taxonomic levels
- Pipeline runs past the DATABASE-loading step without CD-HIT-OTU-Miseq complaining about database format
- Resulting OTU.txt annotations reflect the specific lineages expected from the reordered database

## Limitations

- The ordering heuristic only affects which annotation a 97%-similarity cluster receives when multiple database sequences match; genuinely novel sequences still may remain unannotated
- Only Greengenes and SILVA pre-reformatted versions are offered for download; other databases must be reformatted by the user following CD-HIT-OTU-Miseq installation step 4

## Evidence

- [readme] database should be formatted as described in 4 step of Installation of CD-HIT-OTU-Miseq (this reformatting places sequences with more specific annotations are at the beginning of the file): "Note, that database should be formatted as described in 4 step of Installation of CD-HIT-OTU-Miseq (this reformatting places sequences with more specific annotations are at the beginning of the file)"
- [intro] reformatted versions of Greengenes and Silva databases are downloadable: "You can download reformatted versions of Greengenes and Silva databases here."
- [methods] pipeline takes an annotated microbial 16S sequence FASTA database as input: "Pipeline takes as input a folder with paired-end reads in form of fastq files and database of annotated microbial 16S sequences"
- [methods] clusters are matched against the annotated database to yield annotated OTUs at 97% similarity: "Remaining reads and fragments from annotated database are then clustered at 97% similarity to yields annotated clusters commonly named Operational Taxonomic Units, or OTU."
- [readme] Specify the name of your 16S database in DATABASE parameter: "Specify the name of your 16S database in DATABASE parameter."
