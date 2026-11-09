---
name: multi-module-pipeline-output-aggregation
description: Use when several conditionally-enabled analysis modules must be executed
  per configuration flags and their heterogeneous outputs merged into one summary
  table plus interactive HTML/PDF deliverables, as in QUAST-style assembly evaluation.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled contig FASTA files — QUAST evaluates
  assemblies, it does not assemble reads
- Only a single module's output is needed (e.g. just the misassemblies_report) and
  the full bundled report would add unnecessary overhead
- A reference genome is unavailable and reference-dependent metrics (genome fraction,
  duplication ratio, NGA50, misassemblies) are the primary goal — only the 30 reference-free
  metrics will be produced
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST
  role: Core engine whose processing loop conditionally invokes modules (reference
    alignment, BUSCO, gene predictors) and merges outputs into report.txt/tsv/tex/pdf/html
    and icarus.html
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web front-end that submits assemblies with the run configuration (reference
    genome, BUSCO checkbox, gene-finding request) and bundles results into a single
    downloadable standalone report
  repo: https://github.com/ablab/quast-website
- name: BUSCO
  role: 'Conditional completeness module: runs conservative-gene screening only when
    the completeness option is selected'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:1eef9593d949eaaa830ac4420899cd27c9ca3bb3ac48667802eb1f60e2c28c3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - BUSCO
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# multi-module-pipeline-output-aggregation

## Summary

Conditionally invoke evaluation modules based on user configuration (reference alignment, BUSCO completeness, gene finding) and merge all module outputs into a single bundled QUAST/WebQUAST report with metrics, plots and Icarus viewers.

## When to use

Use when several conditionally-enabled analysis modules must be executed per configuration flags and their heterogeneous outputs merged into one summary table plus interactive HTML/PDF deliverables, as in QUAST-style assembly evaluation.

## When NOT to use

- Input is raw sequencing reads rather than assembled contig FASTA files — QUAST evaluates assemblies, it does not assemble reads
- Only a single module's output is needed (e.g. just the misassemblies_report) and the full bundled report would add unnecessary overhead
- A reference genome is unavailable and reference-dependent metrics (genome fraction, duplication ratio, NGA50, misassemblies) are the primary goal — only the 30 reference-free metrics will be produced

## Inputs

- Assembled contig FASTA files (one or more assemblies, e.g. contigs_1.fasta contigs_2.fasta)
- Optional reference genome FASTA(.gz) supplied via -r
- Optional user-provided gene positions list via -g
- Optional paired reads (FASTQ via -1/-2) and configuration toggles for BUSCO completeness and gene finding

## Outputs

- report.txt (summary table) and report.tsv (tab-separated, parseable)
- report.tex and report.pdf (PDF includes all tables and plots; requires Matplotlib)
- report.html (interactive HTML with everything)
- icarus.html (Icarus main menu with links to interactive viewers)
- contigs_reports/misassemblies_report and contigs_reports/unaligned_report (only if a reference genome is provided)
- Bundled standalone downloadable report directory

## How to apply

Load the assembled contig FASTA files and run configuration, then invoke quast.py; dispatch modules conditionally: align contigs to the reference with the reference-based module only when -r is provided (computing misassemblies, mismatches/indels, genome fraction, duplication ratio, NGA50), run BUSCO-based conservative-gene screening only when completeness is selected, and run embedded gene predictors (GeneMark.hmm for prokaryotes, GeneMark-ES/GlimmerHMM for eukaryotes) only when --gene-finding is requested. Always compute contig-only contiguity metrics (number of contigs >500 bp, largest contig, N50) and merge all module outputs into the summary table. Render the aggregated results as report.txt, report.tsv, report.tex, report.pdf (requires Matplotlib), report.html, and icarus.html plus contigs_reports (misassemblies_report, unaligned_report), and bundle everything into a single downloadable standalone report directory. Rationale: each optional module extends the metric set (30 reference-free metrics versus >60 metrics, eight plots and two Icarus viewers with a reference), so dispatch must track configuration exactly.

## Related tools

- **QUAST** (Core engine whose processing loop conditionally invokes modules (reference alignment, BUSCO, gene predictors) and merges outputs into report.txt/tsv/tex/pdf/html and icarus.html) — https://github.com/ablab/quast
- **WebQUAST** (Web front-end that submits assemblies with the run configuration (reference genome, BUSCO checkbox, gene-finding request) and bundles results into a single downloadable standalone report) — https://github.com/ablab/quast-website
- **BUSCO** (Conditional completeness module: runs conservative-gene screening only when the completeness option is selected)

## Examples

```
./quast.py test_data/contigs_1.fasta test_data/contigs_2.fasta -r test_data/reference.fasta.gz -o quast_test_output
```

## Evaluation signals

- report.html, report.tsv and icarus.html all exist in the output directory and report.html is a valid interactive HTML file
- report.tsv contains the expected metric fields (e.g. N50, genome fraction %, # misassemblies)
- contigs_reports/misassemblies_report and unaligned_report are present if and only if a reference genome was provided
- report.pdf plots are present only when Matplotlib is installed; absent Matplotlib must not break report.html generation
- Metric count matches configuration: 30 quality metrics and three plots in reference-free mode versus >60 metrics, eight plots and two Icarus viewers in reference-based mode

## Limitations

- QUAST inherited limitations of the embedded tools, which are available only on a few platforms (usually Linux) with command-line interfaces unsuitable for researchers with limited computational background — hence the WebQUAST wrapper
- PDF report output requires Matplotlib (version 1.1 or higher recommended; fully tested with v1.3.1); optional submodules need extra dependencies (Time::HiRes for GeneMark-ES, Java 1.8+ and R for GRIDSS)
- The QUAST toolkit is much more informative with at least a close reference genome; purely contig-based metrics (N50, largest contig) miss reference-aware quality signals

## Evidence

- [results] Reference-based metric scaling: "the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers"
- [readme] Report outputs in README: "report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc)"
- [readme] Conditional PDF dependency: "QUAST draws plots in two formats: HTML and PDF. If you need the PDF versions, make sure that you have installed Matplotlib"
- [abstract] Reference-free vs reference modes: "WebQUAST can handle an unlimited number of genome assemblies and evaluate them against a user-provided or pre-loaded reference genome or in a completely reference-free fashion."
