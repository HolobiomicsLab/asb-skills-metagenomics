---
name: interactive-html-report-rendering
description: Use when a genome/metagenome assembly evaluation pipeline (QUAST, WebQUAST,
  MetaQUAST) has computed quality metrics and plots and they must be merged into one
  interactive HTML file with an Icarus viewer menu for the user.
when_to_use_negative:
- The user only needs machine-readable metrics for downstream parsing — report.tsv
  alone suffices; the interactive HTML/Icarus rendering adds nothing for spreadsheet
  or programmatic consumption.
- The input is not a genome/metagenome assembly evaluation but another report type
  (e.g. annotation or read QC) — QUAST's report.html/icarus.html bundle is specific
  to assembly quality assessment.
- Matplotlib (v1.1 or higher) is unavailable and PDF plots are required — QUAST still
  renders HTML plots, but report.pdf cannot be produced in that environment.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: QUAST
  role: core engine that computes metrics and renders report.html, icarus.html, report.tsv/txt/tex/pdf
    from all module outputs
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: web front-end that dispatches quast.py per configuration and delivers the
    bundled interactive report directory as a single download
  repo: https://github.com/ablab/quast-website
- name: Icarus
  role: interactive visualizer whose viewer menu (icarus.html) links per-contig views
    included in the HTML bundle
  repo: https://github.com/ablab/quast
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:086c95b829fa12b882111a88237b5746d3c6424571261a07024d65de46511cbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - BUSCO
  - Icarus
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# interactive-html-report-rendering

## Summary

Bundle genome assembly evaluation results (metrics tables, plots, and Icarus contig viewers) into standalone interactive HTML reports — report.html plus an icarus.html main menu — so that a multi-module QUAST/WebQUAST run is delivered as a single downloadable, browser-readable artifact.

## When to use

Use when a genome/metagenome assembly evaluation pipeline (QUAST, WebQUAST, MetaQUAST) has computed quality metrics and plots and they must be merged into one interactive HTML file with an Icarus viewer menu for the user.

## When NOT to use

- The user only needs machine-readable metrics for downstream parsing — report.tsv alone suffices; the interactive HTML/Icarus rendering adds nothing for spreadsheet or programmatic consumption.
- The input is not a genome/metagenome assembly evaluation but another report type (e.g. annotation or read QC) — QUAST's report.html/icarus.html bundle is specific to assembly quality assessment.
- Matplotlib (v1.1 or higher) is unavailable and PDF plots are required — QUAST still renders HTML plots, but report.pdf cannot be produced in that environment.

## Inputs

- Assembled contig FASTA files (contigs_1.fasta, contigs_2.fasta, ...)
- Optional reference genome FASTA(-gz) (-r reference.fasta.gz)
- Optional user gene positions list (-g genes.txt)
- QUAST module outputs: misassembly calls, unaligned contig report, gene counts, contiguity metrics (number of contigs > 500 bp, largest contig, N50)
- Quast run configuration (reference-free vs reference-based, BUSCO/completeness toggle, --gene-finding/--eukaryote flags)

## Outputs

- report.html — everything in one interactive HTML file
- icarus.html — Icarus main menu with links to interactive viewers
- report.txt, report.tsv, report.tex — summary table (text, tab-separated, LaTeX)
- report.pdf — tables and plots (requires Matplotlib)
- contigs_reports/misassemblies_report and contigs_reports/unaligned_report (reference-based runs)
- k_mer_stats/kmers_report (with --k-mer-stats) and reads_stats/reads_report (when reads are provided)

## How to apply

After quast.py finishes the evaluation loop (reference-based module with minimap2 when -r reference.fasta.gz is given, BUSCO screening when completeness is selected, and GeneMark.hmm/GeneMark-ES/GlimmerHMM gene prediction when gene finding is requested), merge all module outputs into a single summary table of metrics (N50, genome fraction %, # misassemblies, duplication ratio, NGA50) and render it in multiple formats: report.txt, report.tsv, report.tex, report.html, and — if Matplotlib (v1.1+, tested with v1.3.1) is installed — report.pdf. Reference-based runs additionally render contigs_reports/misassemblies_report and contigs_reports/unaligned_report, and icarus.html serves as the Icarus main menu linking to interactive viewers of contigs against the reference. Validate the render by checking that report.html, report.tsv and icarus.html exist, that report.html is a valid interactive HTML file, and that report.tsv carries the expected metric fields. In the WebQUAST server this bundle becomes the single downloadable standalone report directory returned to the user.

## Related tools

- **QUAST** (core engine that computes metrics and renders report.html, icarus.html, report.tsv/txt/tex/pdf from all module outputs) — https://github.com/ablab/quast
- **WebQUAST** (web front-end that dispatches quast.py per configuration and delivers the bundled interactive report directory as a single download) — https://github.com/ablab/quast-website
- **Icarus** (interactive visualizer whose viewer menu (icarus.html) links per-contig views included in the HTML bundle) — https://github.com/ablab/quast

## Examples

```
./quast.py contigs_1.fasta contigs_2.fasta -r reference.fasta.gz -o quast_output
```

## Evaluation signals

- report.html, report.tsv and icarus.html all exist in the output directory after the run
- report.html is a valid interactive HTML file (parses/browses in a browser, not empty or malformed)
- report.tsv contains the expected metric fields (e.g. N50, genome fraction %, # misassemblies)
- Reference-based runs produce contigs_reports/misassemblies_report and contigs_reports/unaligned_report alongside icarus.html
- report.pdf with plots is present when Matplotlib (v1.1+, tested v1.3.1) was installed; reference-based reports expose >60 metrics and two Icarus viewers vs 30 metrics and three plots reference-free

## Limitations

- PDF rendering of plots requires Matplotlib (recommended v1.1 or higher; fully tested with v1.3.1) — without it only HTML plots are produced
- contigs_reports (misassemblies_report, unaligned_report) and Icarus reference viewers are only generated when a reference genome is provided; reference-free runs yield only 30 metrics and three plots
- k_mer_stats and reads_stats sub-reports appear only if --k-mer-stats is specified or reads are supplied

## Evidence

- [readme] Output formats and Icarus menu: "report.html     everything in an interactive HTML file
    icarus.html     Icarus main menu with links to interactive viewers"
- [other] Validation criteria: [evidence span withheld — anchor status: not_found]
- [readme] Matplotlib dependency for PDF: "QUAST draws plots in two formats: HTML and PDF. If you need the PDF versions, make sure that you have installed 
[Matplotlib](https://matplotlib.org/)"
- [methods] Conditional module dispatch feeding the report: [evidence span withheld — anchor status: not_found]
- [results] Reference-based report richness: "the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers"
