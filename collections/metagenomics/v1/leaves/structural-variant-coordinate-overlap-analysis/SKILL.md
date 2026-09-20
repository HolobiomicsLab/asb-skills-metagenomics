---
name: structural-variant-coordinate-overlap-analysis
description: Use when comparing multiple assemblies of one genome against a close-reference
  genome (e.g. a substrain rather than the true reference) and you need to determine
  whether reported misassemblies are shared true structural variants or assembler-specific
  errors.
when_to_use_negative:
- When only a single assembly is evaluated — breakpoint overlap across assemblers
  cannot be computed
- When the provided reference is the true reference genome rather than a close variant
  — reported misassemblies then reflect genuine assembly errors, not shared structural
  variants
- When no reference genome is available — QUAST falls back to reference-free metrics
  (30 metrics, no misassembly reporting)
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST
  role: Core tool run with -r to produce report.tsv metrics and per-assembly misassemblies_report
    used for coordinate overlap analysis
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web-server front end for running the same reference-based evaluation without
    local installation
  repo: https://github.com/ablab/quast-website
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:530f4a5ca5e8ae6693a6220070d432dce4cc538effc53d7e1a51f122776fd8fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - QUAST
  - WebQUAST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# structural-variant-coordinate-overlap-analysis

## Summary

Quantifies and cross-compares reference-based misassemblies (inversions, relocations, translocations, local) reported by QUAST/WebQUAST for multiple assemblies evaluated against a close reference genome, distinguishing true structural variants from assembly errors by checking breakpoint coordinate overlap across assemblers.

## When to use

Use when comparing multiple assemblies of one genome against a close-reference genome (e.g. a substrain rather than the true reference) and you need to determine whether reported misassemblies are shared true structural variants or assembler-specific errors.

## When NOT to use

- When only a single assembly is evaluated — breakpoint overlap across assemblers cannot be computed
- When the provided reference is the true reference genome rather than a close variant — reported misassemblies then reflect genuine assembly errors, not shared structural variants
- When no reference genome is available — QUAST falls back to reference-free metrics (30 metrics, no misassembly reporting)

## Inputs

- Multiple genome assembly FASTA files (e.g. four E. coli K-12 MG1655 assemblies from ABySS, Velvet, SPAdes, MEGAHIT)
- Close-reference genome FASTA (e.g. W3110, GenBank accession AP009048.1) supplied via the -r option

## Outputs

- QUAST report.tsv with misassembly counts, NGA50, genome fraction and duplication ratio per assembler
- Per-assembly contigs_reports/misassemblies_report with coordinates of extensive misassemblies (inversions, relocations, translocations, local)
- Cross-assembly breakpoint overlap analysis quantifying how many misassemblies are shared

## How to apply

Run QUAST (or WebQUAST) on all assemblies with the close reference provided via -r, using a close reference such as E. coli K-12 W3110 (AP009048.1) in place of the true reference MG1655 (NC_000913.3). Parse the tab-separated report.tsv to extract misassembly counts, NGA50, genome fraction and duplication ratio per assembler, then read each assembly's contigs_reports/misassemblies_report to obtain the coordinates of extensive misassemblies (inversions, relocations, translocations, local). Compute coordinate overlap of misassembly breakpoints across assemblies: breakpoints that largely coincide across all (or most) assemblies indicate true structural variants of the close reference rather than assembly errors. Validate against the expected pattern — a large count (e.g. ~49 extensive misassemblies for one assembler versus ~8 for the others) combined with shared coordinates confirms the close-reference scenario is being handled correctly.

## Related tools

- **QUAST** (Core tool run with -r to produce report.tsv metrics and per-assembly misassemblies_report used for coordinate overlap analysis) — https://github.com/ablab/quast
- **WebQUAST** (Web-server front end for running the same reference-based evaluation without local installation) — https://github.com/ablab/quast-website

## Examples

```
quast.py abyss.fasta velvet.fasta spades.fasta megahit.fasta -r W3110_AP009048.fasta -o quast_w3110
```

## Evaluation signals

- One assembler reports ~49 extensive misassemblies versus ~8 for the other three when evaluated against the close reference W3110 (vs 8 total extensive misassemblies against the true reference)
- Misassembly coordinates largely coincide across the four assemblies, matching the expected pattern of true structural variants
- report.tsv contains parseable rows for misassembly count, NGA50, genome fraction (%) and duplication ratio for every assembly
- Each assembly directory contains contigs_reports/misassemblies_report and contigs_reports/unaligned_report (generated only when a reference is provided)
- Genome fraction and duplication ratio are within expected ranges (~4.6–4.8 Mb assemblies against the ~4.6 Mb reference)

## Limitations

- QUAST is 'much more informative if at least a close reference genome is provided along with the assemblies' — reference-free runs do not report misassemblies at all
- Misassembly counts against a close reference are substantially inflated relative to true-reference evaluation (49 versus 8 extensive misassemblies in the E. coli use case), so raw counts must be interpreted together with breakpoint overlap
- More precise misassembly detection based on read pairs requires the full QUAST installation (./setup.py install_full) with GRIDSS, which needs Java 1.8+ and R

## Evidence

- [results] Here we used W3110, another E. coli K-12 substrain, as an example of a close reference: "Here we used W3110, another E. coli K-12 substrain, as an example of a close reference"
- [results] Evaluation against the close reference W3110 reports substantially increased misassemblies: "we evaluated the same four assemblies of the E. coli K-12 MG1655 dataset but selected the reference genome differently"
- [readme] README on misassembly kinds and output files: "Numbers of misassemblies of different kinds (inversions, relocations, translocations, interspecies translocations (metaQUAST only) or local)."
- [readme] README on misassemblies_report location: "contigs_reports/        [only if a reference genome is provided]
      misassemblies_report  detailed report on misassemblies"
- [readme] README on report.tsv for parsing: "report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc)"
