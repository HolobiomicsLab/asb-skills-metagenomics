---
name: k-mer-assembly-parameterization
description: Use when building or reproducing a body-site gene catalog from raw paired-end
  metagenomic reads that requires de novo contig assembly with a broad k-mer sweep
  and reported assembly statistics (contigs, total bp, N50).
when_to_use_negative:
- The goal is read-level taxonomic/functional profiling against an existing reference
  — use VIRGO's mapping module (bowtie-based runMapping.step1.sh) rather than re-assembling
- Reads are not yet preprocessed (adapter/quality trimmed); assembling raw FASTQ will
  inflate contig count and depress N50
- Only metatranscriptomic (RNA-seq) data are available and the goal is gene expression
  quantification; the article notes de novo assembly of metatranscriptomic datasets
  complicates integration with metagenomic data
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: IDBA-UD
  role: De novo metagenomic assembler run with k-mer values spanning 20 to 100 on
    preprocessed paired-end vaginal metagenome reads to produce the contigs used for
    VIRGO
- name: VIRGO
  role: Downstream gene catalog whose assembly step this skill reproduces; contigs
    from IDBA-UD feed the 0.95 million non-redundant gene catalog and validation statistics
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:f1ff764843d1c48d2180e2b6d98ee48c04bab47d0497ecc953c71abf87ffe457
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDBA-UD
  - VIRGO
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# k-mer-assembly-parameterization

## Summary

De novo assembly of preprocessed paired-end vaginal metagenome reads with IDBA-UD using a broad multi-k range (k 20-100), followed by computing contig count, total length, and N50 statistics to reproduce the assembly underlying the VIRGO gene catalog.

## When to use

Use when building or reproducing a body-site gene catalog from raw paired-end metagenomic reads that requires de novo contig assembly with a broad k-mer sweep and reported assembly statistics (contigs, total bp, N50).

## When NOT to use

- The goal is read-level taxonomic/functional profiling against an existing reference — use VIRGO's mapping module (bowtie-based runMapping.step1.sh) rather than re-assembling
- Reads are not yet preprocessed (adapter/quality trimmed); assembling raw FASTQ will inflate contig count and depress N50
- Only metatranscriptomic (RNA-seq) data are available and the goal is gene expression quantification; the article notes de novo assembly of metatranscriptomic datasets complicates integration with metagenomic data

## Inputs

- Preprocessed paired-end vaginal metagenome reads (FASTQ) for the assembly cohort
- IDBA-UD v1.0 executable configured for de novo assembly with k-mer range 20-100

## Outputs

- contigs.fa — final assembled contigs in FASTA format
- Assembly statistics: number of contigs (~1.2 million), total length (~2.8 billion bp), and N50 (~6.2 kbp)

## How to apply

Obtain preprocessed paired-end vaginal metagenome reads for the assembly cohort, then run IDBA-UD v1.0 in de novo assembly mode with k-mer values spanning 20 to 100 so that both low- and high-abundance organisms are captured. Collect the final assembled contigs from the IDBA-UD output and save them as contigs.fa in FASTA format. Compute assembly statistics directly from contigs.fa: total number of contigs, total assembled bases in bp, and N50 (the shortest length at which 50% of the assembled bases reside). Validate that contigs.fa is valid FASTA and that the statistics are consistent with the reported VIRGO assembly values of 1.2 million contigs, 2.8 billion bp total, and N50 of 6.2 kbp; these contigs feed downstream non-redundant gene prediction for the 0.95 million-gene VIRGO catalog.

## Related tools

- **IDBA-UD** (De novo metagenomic assembler run with k-mer values spanning 20 to 100 on preprocessed paired-end vaginal metagenome reads to produce the contigs used for VIRGO)
- **VIRGO** (Downstream gene catalog whose assembly step this skill reproduces; contigs from IDBA-UD feed the 0.95 million non-redundant gene catalog and validation statistics) — github.com/ravel-lab/VIRGO

## Examples

```
idba_ud -r reads.fa -o virgo_assembly --mink 20 --maxk 100
```

## Evaluation signals

- contigs.fa exists and parses as valid FASTA
- Computed number of contigs is ~1.2 million (order-of-magnitude match to the reported VIRGO assembly)
- Total assembled bases are ~2.8 billion bp
- N50 is ~6.2 kbp
- Assemblies were generated with the full k-mer sweep (k 20 to 100), not a single-k run

## Limitations

- De novo assembly of metagenomic or metatranscriptomic datasets typically requires rather substantial computational resources
- Reported statistics (1.2M contigs, 2.8 Gbp, N50 6.2 kbp) reflect the specific vaginal metagenome cohort used to build VIRGO; other cohorts will not match exactly
- The reference-based VIRGO approach is gene-level focused, with limited inclusion of viral and fungal gene sequences

## Evidence

- [methods] Run IDBA-UD v1.0 in de novo assembly mode with k-mer values spanning 20 to 100 on the preprocessed reads.: [evidence span withheld — anchor status: not_found]
- [abstract] we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes: "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes"
- [intro] De novo assembly of metagenomic or metatranscriptomic datasets typically requires rather substantial computational resources: "De novo assembly of metagenomic or metatranscriptomic datasets typically requires rather substantial computational resources"
- [readme] It was built using a combination of metagenomic and urogential isolate genome sequences.: "It was built using a combination of metagenomic and urogential isolate genome sequences."
