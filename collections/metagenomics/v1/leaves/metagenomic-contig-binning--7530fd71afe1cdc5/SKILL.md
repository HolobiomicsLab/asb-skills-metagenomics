---
name: metagenomic-contig-binning
description: Use when starting from a metagenomic de Bruijn-graph assembly (FASTA
  contigs >=500 bp) plus the trimmed reads, and the goal is a bin-quality table of
  completeness and strain heterogeneity per bin.
when_to_use_negative:
- Input is raw FASTQ reads rather than an assembled, length-filtered contig set —
  assemble first with MetaCortex SW before binning.
- The sample is an isolate or low-diversity assembly where strain-level binning metrics
  (completeness, strain heterogeneity) are not meaningful.
- No read-mapping coverage is available — MetaBAT2 requires per-contig coverage depth
  from mapping reads back to the contigs.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaCortex
  role: Produces the contigs to bin via its two-stage CTX de Bruijn graph workflow
    (SW mode, k=63, min coverage 2, delta 0.8) before the 500 bp length filter
  repo: https://github.com/SR-Martin/metacortex.git
- name: Bowtie2
  role: Maps the trimmed reads back to the filtered contigs to supply coverage depth
    for binning
- name: MetaBAT2
  role: Clusters contigs into metagenomic bins using the Bowtie2-derived coverage
- name: CheckM
  role: Assesses each bin for completeness and strain heterogeneity in the bin-quality
    table
- name: trim-galore
  role: Trims adapters and low-quality bases from reads before assembly and read-back
    mapping
  repo: https://github.com/FelixKrueger/TrimGalore
- name: SRA toolkit
  role: Downloads the WGS reads (fastq-dump) used as input to the assembly-binning
    workflow
  repo: https://github.com/ncbi/sra-tools/wiki
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:a9cfcfc4009977d10e414738be153315c6d6d0dd168df049c50c525461ab40c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - Bowtie2
  - CheckM
  - Conda
  - GCC
  - MetaBAT2
  - MetaCortex
  - SRA toolkit
  - trim-galore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metagenomic-contig-binning

## Summary

Clusters metagenomic assembly contigs (e.g. MetaCortex SW contigs filtered to >=500 bp) into metagenomic bins by mapping trimmed reads back with Bowtie2 to derive coverage depth, binning with MetaBAT2, and assessing bin quality (completeness, strain heterogeneity) with CheckM.

## When to use

Use when starting from a metagenomic de Bruijn-graph assembly (FASTA contigs >=500 bp) plus the trimmed reads, and the goal is a bin-quality table of completeness and strain heterogeneity per bin.

## When NOT to use

- Input is raw FASTQ reads rather than an assembled, length-filtered contig set — assemble first with MetaCortex SW before binning.
- The sample is an isolate or low-diversity assembly where strain-level binning metrics (completeness, strain heterogeneity) are not meaningful.
- No read-mapping coverage is available — MetaBAT2 requires per-contig coverage depth from mapping reads back to the contigs.

## Inputs

- MetaCortex SW assembly contigs (FASTA) filtered to length >=500 bp
- Trimmed WGS reads (FASTQ, e.g. ENA ERR2099157 / SRR13060942 human gut sample) trimmed with trim-galore
- Bowtie2 index of the filtered contigs for read-mapping coverage

## Outputs

- MetaBAT2 metagenomic bins (per-bin FASTA; target 155 bins for the human gut sample)
- CheckM bin-quality table with completeness and strain heterogeneity per bin (target: 23 bins >50% complete, 11 bins >50% strain heterogeneity)
- Tabular assembly summary (target: 123720 contigs, N50 5399, max contig 879838 bp)

## How to apply

First produce the contig set to bin: assemble trimmed reads with MetaCortex SW (k=63, minimum coverage 2, delta 0.8) via the two-stage CTX workflow, then discard contigs shorter than 500 bp. Map the trimmed reads back to the filtered contigs with Bowtie2 so MetaBAT2 receives per-contig coverage depth, then cluster the contigs into metagenomic bins with MetaBAT2. Assess each resulting bin with CheckM for completeness and strain heterogeneity, and compile the bin-quality table (target for the human gut sample SRR13060942: 155 bins, 23 bins >50% complete, 11 bins with >50% strain heterogeneity) into tabular output files. MetaCortex can be installed via 'conda install -c bioconda metacortex' or compiled from source with 'make MAXK=63 metacortex'; LLVM will not compile it, so use GCC.

## Related tools

- **MetaCortex** (Produces the contigs to bin via its two-stage CTX de Bruijn graph workflow (SW mode, k=63, min coverage 2, delta 0.8) before the 500 bp length filter) — https://github.com/SR-Martin/metacortex.git
- **Bowtie2** (Maps the trimmed reads back to the filtered contigs to supply coverage depth for binning)
- **MetaBAT2** (Clusters contigs into metagenomic bins using the Bowtie2-derived coverage)
- **CheckM** (Assesses each bin for completeness and strain heterogeneity in the bin-quality table)
- **trim-galore** (Trims adapters and low-quality bases from reads before assembly and read-back mapping) — https://github.com/FelixKrueger/TrimGalore
- **SRA toolkit** (Downloads the WGS reads (fastq-dump) used as input to the assembly-binning workflow) — https://github.com/ncbi/sra-tools/wiki

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git && make MAXK=63 metacortex
```

## Evaluation signals

- Contig count and N50 match the assembly summary target (123720 contigs, N50 5399, max 879838 bp) after the >=500 bp filter.
- Bin count matches the target of 155 bins for the human gut sample.
- CheckM reports 23 bins with >50% completeness.
- CheckM reports 11 bins with >50% strain heterogeneity.
- Every contig sent to MetaBAT2 is >=500 bp and has a Bowtie2 mapping-derived coverage depth from the trimmed reads.

## Limitations

- LLVM will not compile MetaCortex — Mac users must install GCC and set CC to the GCC binary.
- Bins with high strain heterogeneity (11 bins >50% in the reference run) indicate mixed strains that CheckM flags and that should not be treated as clean single-genome bins.
- The binning targets (155 bins, 23 >50% complete) are specific to the human gut WGS sample assembled with k=63, min coverage 2, delta 0.8; other samples will yield different counts.

## Evidence

- [intro] MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file.: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file."
- [methods] Install via Conda: "conda install -c bioconda metacortex"
- [other] Binning workflow: [evidence span withheld — anchor status: not_found]
- [other] CheckM assessment: [evidence span withheld — anchor status: not_found]
- [other] Trimming prerequisite: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
