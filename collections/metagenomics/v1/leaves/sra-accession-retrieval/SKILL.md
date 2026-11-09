---
name: sra-accession-retrieval
description: Use when an analysis requires raw metagenomic reads (FASTQ) identified
  by specific NCBI SRA accession IDs (e.g., SRS-prefixed accessions) as starting material
  for classification or abundance profiling.
when_to_use_negative:
- The reads are already simulated or locally available (e.g., Mason-simulated simBA-5
  reads) and do not need archival retrieval.
- The required data are reference genomes from NCBI RefSeq or the NCBI taxonomy database,
  which are downloaded for database building, not SRA read accessions.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: Kraken
  role: 'Downstream consumer: classifies the retrieved FASTQ reads after trimming
    to assign taxonomic labels.'
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Used with Kraken to count k-mers when building the standard database against
    which the retrieved reads are classified.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:03ef94263e43d42b3d821dc711984956d8f7c3aa3ef78466a783c05a6fbea564
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sra-accession-retrieval

## Summary

Retrieves raw metagenomic sequencing data from the NCBI Sequence Read Archive by accession ID, as done for the HMP saliva samples (SRS019120, SRS014468, SRS015055) in the Kraken evaluation. This supplies the FASTQ inputs required for downstream trimming and taxonomic classification.

## When to use

Use when an analysis requires raw metagenomic reads (FASTQ) identified by specific NCBI SRA accession IDs (e.g., SRS-prefixed accessions) as starting material for classification or abundance profiling.

## When NOT to use

- The reads are already simulated or locally available (e.g., Mason-simulated simBA-5 reads) and do not need archival retrieval.
- The required data are reference genomes from NCBI RefSeq or the NCBI taxonomy database, which are downloaded for database building, not SRA read accessions.

## Inputs

- NCBI SRA accession IDs (e.g., SRS019120, SRS014468, SRS015055)
- NCBI Sequence Read Archive database access

## Outputs

- FASTQ files for each retrieved accession (paired-end reads where available)

## How to apply

Identify the SRA accessions named in the source protocol (here, SRS019120, SRS014468, and SRS015055 from the Human Microbiome Project saliva samples) and download their FASTQ files from the NCBI Sequence Read Archive. These downloads provide the raw read data that the Kraken workflow then trims to remove low-quality bases and adapter sequences, before classification against a Kraken database built with Jellyfish k-mer counting. Note that the article's study retrieved exactly three accessions for its real metagenome evaluation, so retrieval should be restricted to the named accessions unless extending the analysis deliberately.

## Related tools

- **Kraken** (Downstream consumer: classifies the retrieved FASTQ reads after trimming to assign taxonomic labels.) — github.com/DerrickWood/kraken
- **Jellyfish** (Used with Kraken to count k-mers when building the standard database against which the retrieved reads are classified.)

## Evaluation signals

- All named accessions (SRS019120, SRS014468, SRS015055) yield FASTQ files that open as valid FASTQ with reads and quality strings of equal length.
- Retrieved reads can be trimmed and classified by Kraken, reproducing the reported ~68.2% unclassified rate.
- Per-accession Kraken reports can be generated for each accession, confirming reads are attributable to the intended sample.
- Combined genus-level distribution among classified reads is within a few percentage points of Streptococcus 30%, Haemophilus 17%, Prevotella 13%.

## Limitations

- SRA accessions may be updated or unavailable; the article does not specify retrieval tooling or versions.
- Reads retrieved as pairs must be concatenated with 'NNNNN' before Kraken classification so each pair is classified as a single unit.
- Retrieval provides only raw reads; quality and adapter trimming is a separate prerequisite step before classification.

## Evidence

- [discussion] We retrieved the sequences of three accessions (SRS019120, SRS014468 and SRS015055) from the NCBI Sequence Read Archive: "We retrieved the sequences of three accessions (SRS019120, SRS014468 and SRS015055) from the NCBI Sequence Read Archive"
- [other] Download the FASTQ files for accessions SRS019120, SRS014468, and SRS015055 from the NCBI Sequence Read Archive.: [evidence span withheld — anchor status: not_found]
- [abstract] Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences.: "Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences."
