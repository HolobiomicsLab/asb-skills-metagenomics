---
name: genome-completeness-categorization-checkm-prokka
description: Use when final cleaned scaffold FASTAs from hybrid metagenome assemblies
  (e.g. MetaPlatanus, metaFlye, MEGAHIT, OPERA-MS on 18 human gut samples with >1
  Gbp ONT plus Illumina paired-end reads) must be categorized as over-half or near-complete
  bacterial genomes for assembler benchmarking.
when_to_use_negative:
- Input is raw ONT/Illumina FASTQ rather than cleaned, assembled scaffold FASTA —
  CheckM/Prokka categorization operates on finished assemblies, not reads
- The target elements are Swiss-Prot CDSs, biosynthetic gene clusters, or near-complete
  viral sequences rather than bacterial genome scaffold categories — these use separate
  evaluation criteria in the same benchmark
- A reference-based or strain-resolved completeness assessment is required — the study
  explicitly focuses on de novo methods and does not well examine reference information
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0193
tools:
- name: CheckM
  role: Estimates completeness and contamination of scaffolds to classify them as
    over-half or near-complete bacterial genomes
- name: Prokka
  role: Annotates CDSs on scaffolds, providing the gene content used alongside CheckM
    for scaffold categorization
- name: MetaPlatanus
  role: Produces the hybrid (Illumina paired-end + ONT) assemblies whose scaffolds
    are categorized and benchmarked against metaFlye, MEGAHIT and OPERA-MS
  repo: https://github.com/rkajitani/MetaPlatanus
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:a35d32081dbc543b382116e487f0a75e5c13c5b54ad55391eb6efd35aca757f9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/rkajitani/MetaPlatanus
  repo_url: https://github.com/rkajitani/MetaPlatanus
  tools:
  - BWA
  - CheckM
  - MEGAHIT
  - MetaBAT2
  - MetaPlatanus
  - NextPolish
  - OPERA-MS
  - Prokka
  - Racon
  - SAMtools
  - SeqKit
  - TGS-GapCloser
  - metaFlye
  - minimap2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0193
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# genome-completeness-categorization-checkm-prokka

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Categorizes scaffolds from de novo metagenome assemblies as over-half or near-complete bacterial genomes using CheckM completeness/contamination estimates and Prokka annotations, applied to benchmark assemblers such as MetaPlatanus on recovery of biologically important elements in the 18 human gut nanopore samples.

## When to use

Use when final cleaned scaffold FASTAs from hybrid metagenome assemblies (e.g. MetaPlatanus, metaFlye, MEGAHIT, OPERA-MS on 18 human gut samples with >1 Gbp ONT plus Illumina paired-end reads) must be categorized as over-half or near-complete bacterial genomes for assembler benchmarking.

## When NOT to use

- Input is raw ONT/Illumina FASTQ rather than cleaned, assembled scaffold FASTA — CheckM/Prokka categorization operates on finished assemblies, not reads
- The target elements are Swiss-Prot CDSs, biosynthetic gene clusters, or near-complete viral sequences rather than bacterial genome scaffold categories — these use separate evaluation criteria in the same benchmark
- A reference-based or strain-resolved completeness assessment is required — the study explicitly focuses on de novo methods and does not well examine reference information

## Inputs

- Cleaned final scaffold FASTA per assembler per sample (human contaminant scaffolds removed by mapping against the human reference)
- Hybrid assemblies of the 18 human gut nanopore samples (>1 Gbp ONT FASTQ plus mandatory Illumina paired-end FASTQ per sample)
- CheckM marker-gene lineage reference data
- Prokka annotation databases (for CDS identification supporting categorization)

## Outputs

- Per-scaffold CheckM completeness and contamination estimates
- Per-scaffold category assignments: over-half vs near-complete bacterial genomes, per assembler and per sample
- Prokka annotation output (CDS calls) supporting the categorization
- Summary tables of per-assembler, per-sample scaffold-category counts reproducing Figure 2 and Supplementary Tables S15–S17

## How to apply

Start from the cleaned final scaffold FASTA for each assembler and each of the 18 human gut nanopore samples, ensuring human contaminant scaffolds were removed beforehand by mapping reads against the human reference, since contaminant scaffolds would otherwise inflate categorization counts. Run CheckM on the scaffolds to estimate completeness and contamination, and run Prokka to annotate CDS content, using these signals to bin scaffolds into the paper's categories of 'over-half' (scaffolds spanning more than half of a bacterial genome) and 'near-complete' bacterial genomes. Tabulate per-assembler, per-sample category counts alongside the other element criteria (Swiss-Prot CDSs, biosynthetic gene clusters, near-complete viral sequences) compiled into the summary tables reproducing Figure 2 and Supplementary Tables S15–S17. The rationale, per the article's findings, is that MetaPlatanus assembled many biologically important elements including over-half bacterial genomes, so scaffold-completeness categories reveal contiguity differences that raw N50-type metrics can obscure. Treat a result as valid only when every assembler produced a valid final scaffold FASTA per sample and the category counts match the reported Figure 2 and Supplementary Table values.

## Related tools

- **CheckM** (Estimates completeness and contamination of scaffolds to classify them as over-half or near-complete bacterial genomes)
- **Prokka** (Annotates CDSs on scaffolds, providing the gene content used alongside CheckM for scaffold categorization)
- **MetaPlatanus** (Produces the hybrid (Illumina paired-end + ONT) assemblies whose scaffolds are categorized and benchmarked against metaFlye, MEGAHIT and OPERA-MS) — https://github.com/rkajitani/MetaPlatanus

## Evaluation signals

- Per-assembler, per-sample counts of over-half and near-complete scaffolds must match the values reported in Figure 2 and Supplementary Tables S15–S17
- Each scaffold labeled near-complete must be supported by CheckM completeness/contamination estimates and Prokka CDS content consistent with a near-complete bacterial genome
- All assemblies must be evaluated only after human contaminant scaffolds are removed by mapping against the human reference; categorization on uncleaned assemblies is invalid
- Every assembler must yield a valid final scaffold FASTA per sample before categorization can be attempted
- In the 18-sample benchmark, MetaPlatanus should show the largest number of top cases across element criteria, exceeding other tools by at least seven cases

## Limitations

- The exact category thresholds are deferred to Supplementary Tables S15–S17, which are not reproduced in the available text, so the categorization must be reconstructed from the supplementary tables
- No tool versions for CheckM, Prokka, or the compared assemblers are reported in the main text
- The study focuses on de novo methods; reference information has not been well examined, so this categorization does not cover reference-based completeness assessment

## Evidence
