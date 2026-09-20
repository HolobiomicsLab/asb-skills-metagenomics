---
name: krona-interactive-html-report-generation
description: Use when kraken2 taxonomic classifications from an nf-rnaSeqMetagen run
  need to be visualized as interactive hierarchical charts for a sample's raw reads
  and assembled contigs.
when_to_use_negative:
- Input is an existing taxonomic abundance table or set-based intersection data —
  use UpSet/UpSetR visualization instead, as in the pipeline's UpSet plots
- No kraken2 classification has been run yet (exogenous reads not classified against
  the prep.KrakenDB database) — the Krona report depends on classification output
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: nf-rnaSeqMetagen
  role: Pipeline whose run.FilterClassify mode produces the kraken2 classifications
    rendered as Krona reports
  repo: https://github.com/phelelani/nf-rnaSeqMetagen
- name: Nextflow
  role: Workflow management system orchestrating the pipeline steps that generate
    the Krona report inputs
  repo: https://github.com/phelelani/nf-rnaSeqMetagen
- name: Singularity
  role: Containerization used to run the pipeline steps portably and reproducibly
  repo: https://github.com/phelelani/nf-rnaSeqMetagen
- name: kraken2
  role: Classifies remaining exogenous reads against a database of bacterial, archaeal,
    and viral genomes, providing the taxonomic profile visualized by Krona
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.medmic.2020.100011
    title: 'nf-rnaSeqMetagen: A nextflow metagenomics pipeline for identifying and
      characterizing microbial sequences from RNA-seq data'
  source_content_hash: sha256:1ba25b3a14e5007aba0bd247d9df199f374244d0a80234d1baf5523a9591ec2e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/phelelani/nf-rnaSeqMetagen
  repo_url: https://github.com/phelelani/nf-rnaSeqMetagen
  tools:
  - Nextflow
  - Singularity
  - kraken2
  - nf-rnaSeqMetagen
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# krona-interactive-html-report-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render taxonomic classifications produced by the nf-rnaSeqMetagen pipeline (kraken2 over bacterial, archaeal, and viral genomes) as self-contained interactive Krona HTML charts, one per sample, for raw reads and assembled FASTA reads as demonstrated for sample SRR5074528.

## When to use

Use when kraken2 taxonomic classifications from an nf-rnaSeqMetagen run need to be visualized as interactive hierarchical charts for a sample's raw reads and assembled contigs.

## When NOT to use

- Input is an existing taxonomic abundance table or set-based intersection data — use UpSet/UpSetR visualization instead, as in the pipeline's UpSet plots
- No kraken2 classification has been run yet (exogenous reads not classified against the prep.KrakenDB database) — the Krona report depends on classification output

## Inputs

- kraken2 taxonomic classification of raw reads (per sample, e.g. SRR5074528)
- kraken2 taxonomic classification of assembled reads (per-sample FASTA contigs)

## Outputs

- Interactive Krona HTML chart for raw reads: <output_directory>/<sample>/<sample>_reads.html (e.g. examples/output/SRR5074528/SRR5074528_reads.html)
- Interactive Krona HTML chart for assembled reads: <output_directory>/<sample>/<sample>_fasta.html (e.g. examples/output/SRR5074528/SRR5074528_fasta.html)

## How to apply

After running the nf-rnaSeqMetagen pipeline in run.FilterClassify mode with Singularity-containerized steps (nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config), take the kraken2 classification results for each sample. Render the raw-read classification into an interactive Krona HTML chart saved as <sample>_reads.html inside the sample analysis directory (e.g. <output_directory>/SRR5074528/SRR5074528_reads.html). Separately render the assembled-reads (FASTA) classification as <sample>_fasta.html in the same directory. Each chart is self-contained HTML that can be opened in a browser to interactively explore the taxonomic hierarchy; for the tutorial sample SRR5074528 the reference reports are linked in the results section for comparison.

## Related tools

- **nf-rnaSeqMetagen** (Pipeline whose run.FilterClassify mode produces the kraken2 classifications rendered as Krona reports) — https://github.com/phelelani/nf-rnaSeqMetagen
- **Nextflow** (Workflow management system orchestrating the pipeline steps that generate the Krona report inputs) — https://github.com/phelelani/nf-rnaSeqMetagen
- **Singularity** (Containerization used to run the pipeline steps portably and reproducibly) — https://github.com/phelelani/nf-rnaSeqMetagen
- **kraken2** (Classifies remaining exogenous reads against a database of bacterial, archaeal, and viral genomes, providing the taxonomic profile visualized by Krona)

## Examples

```
nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config
```

## Evaluation signals

- Both Krona HTML files exist for each sample under the sample output directory (e.g. examples/output/SRR5074528/SRR5074528_reads.html and SRR5074528_fasta.html)
- Each HTML file opens as a self-contained interactive hierarchical taxonomic chart in a browser without external dependencies
- The chart's taxonomic hierarchy matches the kraken2 classification against the K2DB database (bacterial, archaeal, viral genomes)
- Raw-reads chart and assembled-reads chart are distinguishable and correspond to the correct input (reads vs. FASTA) per the <sample>_reads.html / <sample>_fasta.html naming convention
- Generated reports visually match the reference Krona reports linked in the article's results section for SRR5074528

## Limitations

- The pipeline is designed for RNA-seq metagenomics input after host-read filtering against a reference (e.g. Mus_musculus.GRCm38 STAR index); Krona reports only reflect exogenous reads that survive filtering
- Taxonomic resolution is bounded by the kraken2 database (bacterial, archaeal, and viral genomes only) used in prep.KrakenDB
- Article's discussion section is absent from source ("_No changelog found._"), so no documented limitations or failure modes for report rendering are extractable

## Evidence

- [methods] Krona report: raw reads (SRR5074528): "#### 3.2.1. Krona report: raw reads (SRR5074528)"
- [abstract] kraken2 database constructed from bacterial, archaeal, and viral genomes: "The remaining exogenous reads are then characterized using the kraken2 database constructed from bacterial, archaeal, and viral genomes"
- [methods] run.FilterClassify command: "To perform filtering of host reads and classification of exogeneous reads, use this command"
- [readme] Download RNA-seq test dataset from H3ABioNet script: "Download RNA-seq test dataset from H3ABioNet: <a href="examples/data/get_data.sh" target="_blank">script</a>."
