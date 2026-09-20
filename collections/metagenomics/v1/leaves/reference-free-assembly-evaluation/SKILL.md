---
name: reference-free-assembly-evaluation
description: Use when no reference genome is available and multiple assemblies (e.g.,
  FASTA contigs from ABySS, Velvet, SPAdes, MEGAHIT) must be compared on contiguity
  and completeness metrics.
when_to_use_negative:
- A reference genome is available — reference-based QUAST reports >60 metrics, eight
  assessment plots and two Icarus viewers, so use reference-based evaluation instead
- The input is already a QUAST report or quality metrics table rather than raw assembly
  contigs
- The dataset is metagenomic or very large (mammalian-scale), where MetaQUAST or QUAST-LG
  extensions are the appropriate tools
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: QUAST
  role: Command-line engine that computes the 30 reference-free quality metrics and
    generates report.tsv and three assessment plots
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web server providing the same QUAST functionality with a graphical interface
    on any platform, freely available at ccb.uni-saarland.de/quast
  repo: https://github.com/ablab/quast-website
- name: BUSCO
  role: Conserved-gene-based completeness estimate used to complement contiguity metrics
    (98.7% complete for all four E. coli assemblies)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:2b0d23d225b471b6beb97e162dd68fb530779964e18984c94209fc73f60536d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - BUSCO
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# reference-free-assembly-evaluation

## Summary

Evaluates and compares genome assemblies without a reference genome using QUAST/WebQUAST, which computes 30 reference-free quality metrics (contig counts, total length, largest contig, N50/N90/auN, G+C content, # N's per 100 kbp) and three assessment plots focused on contiguity and completeness.

## When to use

Use when no reference genome is available and multiple assemblies (e.g., FASTA contigs from ABySS, Velvet, SPAdes, MEGAHIT) must be compared on contiguity and completeness metrics.

## When NOT to use

- A reference genome is available — reference-based QUAST reports >60 metrics, eight assessment plots and two Icarus viewers, so use reference-based evaluation instead
- The input is already a QUAST report or quality metrics table rather than raw assembly contigs
- The dataset is metagenomic or very large (mammalian-scale), where MetaQUAST or QUAST-LG extensions are the appropriate tools

## Inputs

- FASTA files of genome assembly contigs from each assembler (e.g., ABySS, Velvet, SPAdes, MEGAHIT outputs)
- Multiple assemblies passed together in a single QUAST run for comparison

## Outputs

- report.tsv (tab-separated summary table with ~30 quality metrics, one column per assembly)
- report.txt / report.tex / report.pdf / report.html summary tables
- Three reference-free assessment plots addressing contiguity and completeness

## How to apply

Install QUAST from the ablab/quast GitHub repo (Python3 3.3+, GCC 4.7+, zlib dev files; setup.py install is optional since QUAST compiles sub-parts on first use) or submit the assemblies to the WebQUAST server at ccb.uni-saarland.de/quast. Run quast.py on all assembly FASTA files together without the -r reference option, directing output to a directory (e.g., -o quast_output), so the 30 reference-free metrics and three assessment plots are computed uniformly. Parse the generated report.tsv (tab-separated summary table) and extract number of contigs, total length, largest contig, N50, N90, auN, G+C content, and # N's per 100 kbp to rank assemblers — e.g., in the E. coli K-12 MG1655 use case SPAdes achieved the best N50/N90/auN and the largest contig (285 kb) while ABySS was largest (4.8 Mb) but most fragmented (176 contigs vs 90–95). Optionally complement contiguity assessment with BUSCO to estimate completeness (all four E. coli assemblies were 98.7% BUSCO-complete) and sanity-check G+C against expected organism range (50.4–50.8% for E. coli).

## Related tools

- **QUAST** (Command-line engine that computes the 30 reference-free quality metrics and generates report.tsv and three assessment plots) — https://github.com/ablab/quast
- **WebQUAST** (Web server providing the same QUAST functionality with a graphical interface on any platform, freely available at ccb.uni-saarland.de/quast) — https://github.com/ablab/quast-website
- **BUSCO** (Conserved-gene-based completeness estimate used to complement contiguity metrics (98.7% complete for all four E. coli assemblies))

## Examples

```
python quast.py abyss_contigs.fasta velvet_contigs.fasta spades_contigs.fasta megahit_contigs.fasta -o quast_output
```

## Evaluation signals

- report.tsv exists, is tab-separated, and contains one column per input assembly
- Contig count matches reported ranges (e.g., 176 contigs for ABySS vs 90–95 for Velvet, SPAdes and MEGAHIT)
- Total length falls in expected range (4.8 Mb vs 4.6 Mb for the E. coli use case)
- G+C content lies within the expected organism range (50.4–50.8% for E. coli; ~50.7% observed)
- # N's per 100 kbp pattern is reproducible (e.g., ~94 uncalled bases per 100 kb for Velvet vs none for MEGAHIT)

## Limitations

- Reference-free mode is mainly informative about contiguity and completeness; metrics like NGA50, genome fraction, misassemblies and duplication ratio require a reference genome
- QUAST inherited limitations of embedded tools available mainly on Linux with a command-line interface, which WebQUAST addresses but local runs still require Python3, GCC 4.7+, zlib dev files
- Reference-free metrics alone may be one-sided: quality assessment should consider that even a close reference makes the evaluation much more informative

## Evidence

- [results] When a reference genome is unavailable, WebQUAST computes 30 quality metrics and draws three assessment plots that mainly address the contiguity and completeness: "When a reference genome is unavailable, WebQUAST computes 30 quality metrics and draws three assessment plots that mainly address the contiguity and completeness"
- [results] SPAdes assembled larger contigs on average (the best N50, N90 and auN, the area under the Nx curve, values with Velvet and MEGAHIT being close runner-ups): "SPAdes assembled larger contigs on average (the best N50, N90 and auN, the area under the Nx curve, values with Velvet and MEGAHIT being close runner-ups)"
- [readme] report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc): "report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc)"
- [readme] QUAST automatically compiles all its sub-parts when needed (on the first use). Thus, installation is not required.: "QUAST automatically compiles all its sub-parts when needed (on the first use). Thus, installation is not required."
- [intro] or look for conservative genes to estimate the assembly completeness, such as BUSCO (7,8) and CEGMA (9): "or look for conservative genes to estimate the assembly completeness, such as BUSCO (7,8) and CEGMA (9)"
