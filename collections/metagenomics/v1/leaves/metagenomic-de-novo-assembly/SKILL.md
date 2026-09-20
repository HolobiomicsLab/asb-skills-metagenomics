---
name: metagenomic-de-novo-assembly
description: Use when raw or preprocessed paired-end metagenome FASTQ reads must be
  assembled into contigs de novo as the first step toward building or reproducing
  a gene catalog such as VIRGO, and assembly statistics are the deliverable.
when_to_use_negative:
- 'The goal is taxonomic/functional profiling of an existing sample: VIRGO supports
  reference-based mapping of reads directly against its non-redundant gene database,
  and de novo assembly of metagenomic datasets ''requires substantial computational
  resources and complicates integration of metagenomic and metatranscriptomic data''.'
- Input is already an assembled contig set or a feature/annotation table rather than
  sequencing reads.
- The target community is not vaginal microbiome material and a site-specific catalog
  like VIRGO is not the downstream goal.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0193
- http://edamontology.org/topic_3168
tools:
- name: IDBA-UD
  role: De novo assembler run on preprocessed vaginal metagenome reads with k-mer
    values 20–100; its output contigs form contigs.fa used to compute assembly statistics
- name: VIRGO
  role: Downstream non-redundant gene catalog (human vaginal) that the assembled contigs
    feed into; also the reference against which the assembly statistics are validated
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:62285a0375802749d629d8d49cfd90f13cc4c42549b1b57649bcc2282b8b4f4a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDBA-UD
  - VIRGO
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0193
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metagenomic-de-novo-assembly

## Summary

De novo assembly of preprocessed paired-end vaginal metagenome reads with IDBA-UD (k-mer 20–100) to produce contigs that underpin the VIRGO non-redundant gene catalog, evaluated by contig count, total assembled bp, and N50.

## When to use

Use when raw or preprocessed paired-end metagenome FASTQ reads must be assembled into contigs de novo as the first step toward building or reproducing a gene catalog such as VIRGO, and assembly statistics are the deliverable.

## When NOT to use

- The goal is taxonomic/functional profiling of an existing sample: VIRGO supports reference-based mapping of reads directly against its non-redundant gene database, and de novo assembly of metagenomic datasets 'requires substantial computational resources and complicates integration of metagenomic and metatranscriptomic data'.
- Input is already an assembled contig set or a feature/annotation table rather than sequencing reads.
- The target community is not vaginal microbiome material and a site-specific catalog like VIRGO is not the downstream goal.

## Inputs

- preprocessed paired-end vaginal metagenome reads (FASTQ)
- IDBA-UD v1.0 executable configured for de novo assembly with k-mer range 20–100

## Outputs

- contigs.fa (FASTA file of final assembled contigs from IDBA-UD output)
- assembly statistics: number of contigs, total assembled length in bp, and N50

## How to apply

Obtain the preprocessed paired-end vaginal metagenome reads for the assembly cohort and run IDBA-UD v1.0 in de novo assembly mode with k-mer values spanning 20 to 100 on those reads. Collect the final assembled contigs from the IDBA-UD output and save them as contigs.fa in FASTA format. Compute assembly statistics directly from contigs.fa: total number of contigs, total assembled bases in bp, and N50. The rationale is that VIRGO was built from such assemblies plus urogenital isolate genomes, and the reference result for the vaginal metagenomes is a combined set of 1.2 million contigs totaling 2.8 billion bp with an N50 of 6.2 kbp.

## Related tools

- **IDBA-UD** (De novo assembler run on preprocessed vaginal metagenome reads with k-mer values 20–100; its output contigs form contigs.fa used to compute assembly statistics)
- **VIRGO** (Downstream non-redundant gene catalog (human vaginal) that the assembled contigs feed into; also the reference against which the assembly statistics are validated) — github.com/ravel-lab/VIRGO

## Examples

```
idba_ud -r preprocessed_reads.fa -o vaginal_assembly; grep '>' vaginal_assembly/contigs.fa | wc -l
```

## Evaluation signals

- contigs.fa exists and is valid FASTA (parseable headers and nucleotide sequences)
- combined assembly statistics match the reported VIRGO values: 1.2 million contigs, 2.8 billion bp total, N50 of 6.2 kbp
- k-mer sweep actually covered 20 to 100 as specified for IDBA-UD v1.0
- computed N50 and total length are internally consistent with the contig length distribution in contigs.fa

## Limitations

- De novo assembly of metagenomic or metatranscriptomic datasets requires rather substantial computational resources and complicates integration of metagenomic and metatranscriptomic data.
- VIRGO takes a reference-based approach afterward, and that approach has acknowledged limitations; the catalog is primarily bacterial with limited inclusion of viral and fungal gene sequences.
- README lists mapping-stage dependencies (BLAST 2.2.3+, Bowtie 1.1.2+, GNU Awk 3.1+, seqtk 1.0+) but does not document the assembly toolchain or parameters, so reproduction relies on the reported IDBA-UD settings.

## Evidence

- [other] assembly statistics and VIRGO reference values: [evidence span withheld — anchor status: not_found]
- [other] IDBA-UD run parameters from the workflow: [evidence span withheld — anchor status: not_found]
- [readme] what the assembled contigs feed into: "VIRGO is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic and urogential isolate genome sequences."
- [intro] cost of de novo assembly (why reference-based VIRGO exists): "De novo assembly of metagenomic or metatranscriptomic datasets typically requires rather substantial computational resources and complicates integration of metagenomic and metatranscriptomic data."
- [abstract] the catalog built from the assembly: "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO)"
