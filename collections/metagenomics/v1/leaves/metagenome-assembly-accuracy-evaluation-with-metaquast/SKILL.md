---
name: metagenome-assembly-accuracy-evaluation-with-metaquast
description: Use when contigs from a strain-aware metagenome assembler must be scored
  against known reference genomes to quantify reconstruction accuracy (genome fraction,
  misassemblies) across varying coverage levels or conditions.
when_to_use_negative:
- The assembly quality must be assessed without any reference genomes available —
  metaQuast accuracy evaluation requires known references (de novo-only scenarios
  cannot yield genome fraction)
- The goal is only to measure contig length statistics (e.g. N50) via plain Quast
  rather than reference-based accuracy against spiked-in strain genomes
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: metaQuast
  role: Reference-based evaluation of spiked-in strain assemblies, yielding genome
    fraction and misassembly-related accuracy metrics
- name: StrainXpress
  role: De novo strain-aware metagenome assembler producing 'stageb/final_contigs.fasta'
    from interleaved FASTQ input, whose contigs are then evaluated
  repo: https://github.com/HaploKit/StrainXpress
- name: Quast
  role: Determines the criteria listed in the paper's results tables for assembly
    quality
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:7e1b86e707d5127c2b17c6aa03bf9f9c9b5f3b975481a6b5e6fc40223a50ada8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - Quast
  - StrainXpress
  - metaQuast
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metagenome-assembly-accuracy-evaluation-with-metaquast

## Summary

Evaluates strain-aware metagenome assemblies against known reference genomes using metaQuast, extracting genome fraction and misassembly-related metrics — as done for StrainXpress contigs assembled from the Salmonella spike-in datasets (5x–40x coverage in real gut metagenomes).

## When to use

Use when contigs from a strain-aware metagenome assembler must be scored against known reference genomes to quantify reconstruction accuracy (genome fraction, misassemblies) across varying coverage levels or conditions.

## When NOT to use

- The assembly quality must be assessed without any reference genomes available — metaQuast accuracy evaluation requires known references (de novo-only scenarios cannot yield genome fraction)
- The goal is only to measure contig length statistics (e.g. N50) via plain Quast rather than reference-based accuracy against spiked-in strain genomes

## Inputs

- Assembled contigs in FASTA format (e.g. StrainXpress 'stageb/final_contigs.fasta')
- Known reference genomes for the spiked-in or targeted strains (e.g. Salmonella spike-in references)
- Interleaved paired-end FASTQ metagenome reads for the assembly step

## Outputs

- metaQuast accuracy metrics per assembly (genome fraction, misassembly-related quantities)
- CSV table with one row per condition (e.g. coverage level 5x–40x) and columns of metaQuast-derived metrics (e.g. coverage_vs_accuracy.csv)

## How to apply

Assemble the input metagenome reads de novo with StrainXpress by invoking 'python strainxpress.py -fq all_reads.fq' (adding '-fast' for large datasets, per the README recommendation); the assembly output is 'stageb/final_contigs.fasta'. Input reads must be interleaved paired-end FASTQ. Then evaluate each contig set against the known reference genomes (e.g. the 10 spiked-in Salmonella strains) with metaQuast, extracting genome fraction and misassembly-related quantities as reported in the paper's tables. Assemble results into a CSV table with one row per condition (e.g. each spike-in coverage level from 5x to 40x) and columns holding the metaQuast-derived accuracy metrics, enabling a coverage-vs-accuracy sweep against ground truth.

## Related tools

- **metaQuast** (Reference-based evaluation of spiked-in strain assemblies, yielding genome fraction and misassembly-related accuracy metrics)
- **StrainXpress** (De novo strain-aware metagenome assembler producing 'stageb/final_contigs.fasta' from interleaved FASTQ input, whose contigs are then evaluated) — https://github.com/HaploKit/StrainXpress
- **Quast** (Determines the criteria listed in the paper's results tables for assembly quality)

## Examples

```
python strainxpress.py -fq all_reads.fq -fast && metaquast.py stageb/final_contigs.fasta -r salmonella_references.fasta
```

## Evaluation signals

- coverage_vs_accuracy.csv exists and contains all eight coverage levels from 5x to 40x
- Each row of the CSV carries the metaQuast accuracy metrics for the spiked-in Salmonella strains
- Result at the lowest coverage is consistent with the paper: StrainXpress reconstructs 70.26% of the spiked-in Salmonella genomes at 5x coverage, versus 48.26% for second-best MEGAHIT
- Genome Fraction reaches 84.29% from 15x coverage onwards
- Each metaQuast run used the correct reference set (the 10 spiked-in Salmonella strains) and produced per-reference metrics

## Limitations

- The short length of Illumina-type reads constrains the contig length a strain-aware assembler can compute, preventing full-length strain-specific contigs and thus capping achievable genome fraction
- Reference-based metaQuast evaluation is only possible when ground-truth strain genomes (e.g. the Salmonella spike-in references) are known; it cannot be applied to uncharacterized communities

## Evidence

- [results] evaluated the quality of the assemblies of the 10 spiked in salmonella strains using metaQuast: "evaluated the quality of the assemblies of the 10 spiked in salmonella strains using metaQuast"
- [results] the 8 data sets varied in terms of coverage of the spiked-in strains, ranging from 5× to 40× in steps of 5×: "the 8 data sets varied in terms of coverage of the spiked-in strains, ranging from 5× to 40× in steps of 5×"
- [results] Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40): "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [readme] README: input must be interleaved FASTQ and results are in the stageb folder: "The input file must be interleaved FASTQ and format like below"
- [readme] README: fast cluster method recommendation for big data sets: "When a data set is big, we recommend to use the fast cluster method"
