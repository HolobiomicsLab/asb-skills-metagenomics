---
name: assembly-contiguity-metric-computation
description: Use when comparing multiple assemblies of the same genome (e.g. outputs
  from different assemblers such as SPAdes, Velvet, ABySS, MEGAHIT) and you need contiguity
  statistics like N50, N90, auN, or largest contig length to rank them.
when_to_use_negative:
- The goal is functional completeness (conserved-gene screening) rather than contiguity
  — use BUSCO-style evaluation instead of relying on N50 alone
- Assemblies are of different genomes/species, so contiguity metrics are not comparable
- Only a single FASTA needs a quick size check and no comparative Nx statistics are
  required
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST
  role: Core engine that computes contiguity metrics (number of large contigs >500
    bp, largest contig, N50, NGA50, auN) from contig FASTA files and emits report.tsv/report.html
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web server providing QUAST's contiguity metric computation through a graphical
    interface, e.g. reference-free evaluation producing 30 quality metrics and three
    assessment plots
  repo: https://www.ccb.uni-saarland.de/quast/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:d583f49e3869a5c6f9fdef65e7a5117699083f0514f982fa0bab46e727aee84e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - BUSCO
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# assembly-contiguity-metric-computation

## Summary

Computes contiguity metrics for genome assemblies using QUAST-style calculations: number of large contigs (>500 bp), largest contig length, N50, N90, auN (area under the Nx curve), and, when a reference is provided, the reference-aware NGA50. This enables quantitative comparison of how fragmented different assemblies of the same genome are.

## When to use

Use when comparing multiple assemblies of the same genome (e.g. outputs from different assemblers such as SPAdes, Velvet, ABySS, MEGAHIT) and you need contiguity statistics like N50, N90, auN, or largest contig length to rank them.

## When NOT to use

- The goal is functional completeness (conserved-gene screening) rather than contiguity — use BUSCO-style evaluation instead of relying on N50 alone
- Assemblies are of different genomes/species, so contiguity metrics are not comparable
- Only a single FASTA needs a quick size check and no comparative Nx statistics are required

## Inputs

- Assembled contig FASTA files (one per assembly, multiple accepted for comparison)
- Optional reference genome FASTA/FASTA.gz (e.g. -r reference.fasta.gz)

## Outputs

- report.txt (summary table)
- report.tsv (tab-separated metrics including N50, NGA50, genome fraction %, # misassemblies)
- report.html / report.pdf (tables and Nx plots)
- icarus.html (interactive viewer menu)
- contigs_reports/ (misassemblies_report, unaligned_report; only when a reference is provided)

## How to apply

Load the assembled contig FASTA files and run quast.py on them, optionally with a reference genome via the -r flag (e.g. 'quast.py contigs_1.fasta contigs_2.fasta -r reference.fasta.gz -o quast_output'). Compute contiguity metrics based only on the contigs: number of large contigs (longer than 500 bp) and their total length, length of the largest contig, and N50 (the contig length such that all contigs of at least that length together cover at least 50% of the assembly). When a reference genome is supplied, additionally compute NGA50, the reference-aware version of N50 calculated using aligned blocks instead of contigs, plus genome fraction and duplication ratio. Interpret auN as the area under the Nx curve to capture contiguity beyond the single N50/N90 breakpoints. Merge these contiguity metrics with any module outputs (misassemblies, gene counts) into the summary table and inspect report.tsv for the expected fields.

## Related tools

- **QUAST** (Core engine that computes contiguity metrics (number of large contigs >500 bp, largest contig, N50, NGA50, auN) from contig FASTA files and emits report.tsv/report.html) — https://github.com/ablab/quast
- **WebQUAST** (Web server providing QUAST's contiguity metric computation through a graphical interface, e.g. reference-free evaluation producing 30 quality metrics and three assessment plots) — https://www.ccb.uni-saarland.de/quast/

## Examples

```
quast.py contigs_1.fasta contigs_2.fasta -r reference.fasta.gz -o quast_output
```

## Evaluation signals

- report.html, report.tsv and icarus.html all exist in the output directory
- report.tsv contains the expected contiguity metric fields (e.g. N50, genome fraction %, # misassemblies)
- report.html is a valid interactive HTML file
- For a reference-based run, NGA50 appears in the report; for reference-free runs, 30 quality metrics and three assessment plots are reported
- Ranking sanity check: in the article's E. coli example, SPAdes had the best N50, N90 and auN and the largest contig (285 kb) while ABySS produced the most fragmented assembly (176 contigs versus 90–95)

## Limitations

- N50-type metrics assess contiguity only, not accuracy; QUAST is much more informative if at least a close reference genome is provided
- PDF reports require Matplotlib (version 1.1 or higher recommended; QUAST is fully tested with Matplotlib v.1.3.1)
- Reference-based extras (NGA50, misassemblies, contigs_reports/) are only produced when a reference genome is provided
- A high duplication ratio (>1) from overestimated repeat multiplicities or contig overlaps can distort length-based metrics

## Evidence

- [readme] Number of large contigs (i.e., longer than 500 bp) and total length of them.: "Number of large contigs (i.e., longer than 500 bp) and total length of them."
- [readme] N50 definition: "N50 (length of a contig, such that all the contigs of at least the same length together cover at least 50% of the assembly)."
- [readme] NGA50 reference-aware metric: "NGA50, a reference-aware version of N50 metric. It is calculated using aligned blocks instead of contigs."
- [readme] Documented usage example: "quast.py test_data/contigs_1.fasta test_data/contigs_2.fasta -r test_data/reference.fasta.gz"
- [results] Reference-free evaluation scope: "When a reference genome is unavailable, WebQUAST computes 30 quality metrics and draws three assessment plots that mainly address the contiguity and completeness"
- [results] SPAdes contiguity result: "SPAdes assembled larger contigs on average (the best N50, N90 and auN, the area under the Nx curve, values with Velvet and MEGAHIT being close runner-ups)"
