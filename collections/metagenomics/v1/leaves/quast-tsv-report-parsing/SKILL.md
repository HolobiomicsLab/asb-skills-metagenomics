---
name: quast-tsv-report-parsing
description: Use when you have one or more genome assemblies (FASTA contigs) to evaluate
  against a reference genome (e.g., E. coli K-12 MG1655 NC_000913.3) and need machine-parseable
  metrics to compare assemblers.
when_to_use_negative:
- When no reference genome exists and only contiguity metrics are needed — reference-free
  QUAST computes 30 metrics, but reference-based claims (genome fraction, NGA50, misassemblies)
  require a reference via -r
- When evaluating metagenomic assemblies — MetaQUAST, not plain QUAST with a single
  -r reference, is the appropriate extension
- When assemblies are from large genomes (e.g., mammalian) — QUAST-LG is the intended
  extension
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: QUAST
  role: Command-line tool that aligns assemblies to a reference, detects misassemblies,
    computes quality metrics, and writes report.tsv for parsing
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web server providing the same QUAST evaluation functionality with a graphical
    interface, used to evaluate the four E. coli assemblies
  repo: https://www.ccb.uni-saarland.de/quast/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:d67da5c50e3b5e2a665465dba43830be8ba6cd1ee2fedb46dce6276c43368212
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# quast-tsv-report-parsing

## Summary

Runs QUAST (or WebQUAST) on genome assemblies with an optional reference genome and reads, then parses the tab-separated report.tsv summary table to extract reference-based quality metrics such as misassemblies, genome fraction %, duplication ratio, NGA50, and mismatches/indels per 100 kb for assembler comparison.

## When to use

Use when you have one or more genome assemblies (FASTA contigs) to evaluate against a reference genome (e.g., E. coli K-12 MG1655 NC_000913.3) and need machine-parseable metrics to compare assemblers.

## When NOT to use

- When no reference genome exists and only contiguity metrics are needed — reference-free QUAST computes 30 metrics, but reference-based claims (genome fraction, NGA50, misassemblies) require a reference via -r
- When evaluating metagenomic assemblies — MetaQUAST, not plain QUAST with a single -r reference, is the appropriate extension
- When assemblies are from large genomes (e.g., mammalian) — QUAST-LG is the intended extension

## Inputs

- Genome assembly contigs in FASTA format (e.g., MEGAHIT, SPAdes, Velvet, ABySS outputs)
- Reference genome FASTA/FASTA.gz (e.g., NC_000913.3, GCF_000005845.2) supplied via -r
- Paired-end reads in FASTQ/FASTQ.gz (e.g., SRA accession ERR008613) supplied via -1/-2

## Outputs

- report.tsv — tab-separated summary table for parsing
- report.txt / report.pdf / report.html — human-readable summary tables and plots
- contigs_reports/misassemblies_report — detailed report on misassemblies (when a reference is provided)
- icarus.html — Icarus interactive viewer main menu

## How to apply

Supply the reference genome via the -r flag and read pairs via -1/-2 so QUAST performs full reference-based and read-pair-based misassembly detection. Let QUAST align contigs to the reference, split alignments at misassembly breakpoints, and compute metrics including extensive misassemblies (relocations, translocations, inversions), genome fraction %, duplication ratio, NGA50, and mismatches/indels per 100 kb. Export the combined summary and use report.tsv, the tab-separated version written specifically for parsing or spreadsheets, rather than report.txt or report.pdf. Validate the parsed fields against expected values before downstream use (e.g., 0 extensive misassemblies for MEGAHIT and SPAdes versus 4 each for Velvet and ABySS on the E. coli K-12 MG1655 dataset).

## Related tools

- **QUAST** (Command-line tool that aligns assemblies to a reference, detects misassemblies, computes quality metrics, and writes report.tsv for parsing) — https://github.com/ablab/quast
- **WebQUAST** (Web server providing the same QUAST evaluation functionality with a graphical interface, used to evaluate the four E. coli assemblies) — https://www.ccb.uni-saarland.de/quast/

## Examples

```
quast.py megahit.fasta spades.fasta velvet.fasta abyss.fasta -r reference.fasta.gz -1 reads1.fastq.gz -2 reads2.fastq.gz -o quast_output
```

## Evaluation signals

- report.tsv exists in the QUAST output directory and its fields are tab-separated and parseable
- Reference-based fields are present (misassemblies, genome fraction %, duplication ratio, NGA50, # mismatches per 100 kbp, # indels per 100 kbp) only when -r was supplied
- Parsed values match the Use Case 2 expectations: 0 extensive misassemblies for MEGAHIT and SPAdes versus 4 each for Velvet and ABySS
- ABySS genome fraction of 98.7% versus 98.0–98.4% for the other three assemblers on the E. coli dataset
- SPAdes per-base accuracy of approximately 1.0 mismatches and 0.3 indels per 100 kb

## Limitations

- QUAST is most informative only when at least a close reference genome is provided along with the assemblies
- Optional submodules require extra dependencies: Java 1.8+ and R for GRIDSS (structural variation detection), Time::HiRes for GeneMark-ES (--gene-finding --eukaryote)
- PDF plots require Matplotlib (version 1.1 or higher recommended, fully tested with v1.3.1)

## Evidence

- [readme] report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc): "report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc)"
- [readme] Numbers of misassemblies of different kinds (inversions, relocations, translocations...): "Numbers of misassemblies of different kinds (inversions, relocations, translocations, interspecies translocations (metaQUAST only) or local)."
- [readme] NGA50, a reference-aware version of N50 metric: "NGA50, a reference-aware version of N50 metric. It is calculated using aligned blocks instead of contigs."
- [results] the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers: "the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers"
- [other] Run quast.py on the four assemblies from task_002 (MEGAHIT, SPAdes, Velvet, ABySS) with the reference supplied via the -r flag: [evidence span withheld — anchor status: not_found]
