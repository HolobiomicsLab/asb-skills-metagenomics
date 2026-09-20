---
name: assembler-output-comparison-across-assemblies
description: Use when several de novo assembly contig sets in FASTA format (e.g. ABySS,
  Velvet, SPAdes, MEGAHIT output) must be compared without a reference genome to select
  the most contiguous, complete assembly.
when_to_use_negative:
- A close or true reference genome is available and reference-aware metrics such as
  NGA50, genome fraction, duplication ratio, and misassemblies are required (>60 metrics
  instead of 30)
- The dataset is a metagenomic assemblage of genomes, where MetaQUAST's per-genome
  reference detection is the appropriate mode
- Only one assembly needs validation without cross-assembler comparison
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST
  role: Command-line engine computing the reference-free quality metrics and comparison
    table (report.tsv) for all assemblies
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web-server front end providing the same QUAST reference-free evaluation (30
    metrics, three plots) with a graphical interface
  repo: https://github.com/ablab/quast-website
- name: BUSCO
  role: Completeness cross-check based on conservative genes, used alongside QUAST
    metrics to confirm assembly completeness (98.7% complete for all four E. coli
    assemblies)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:50423fc08e25c7f72d8b9ab8f9b6ad2731fd69036992de74d512b1648b8af90d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - BUSCO
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# assembler-output-comparison-across-assemblies

## Summary

Reference-free quality assessment and comparison of multiple genome assemblies using QUAST/WebQUAST, extracting contiguity and completeness metrics (contig count, total length, largest contig, N50, N90, auN, G+C content, # N's per 100 kbp) from report.tsv to identify the best assembler.

## When to use

Use when several de novo assembly contig sets in FASTA format (e.g. ABySS, Velvet, SPAdes, MEGAHIT output) must be compared without a reference genome to select the most contiguous, complete assembly.

## When NOT to use

- A close or true reference genome is available and reference-aware metrics such as NGA50, genome fraction, duplication ratio, and misassemblies are required (>60 metrics instead of 30)
- The dataset is a metagenomic assemblage of genomes, where MetaQUAST's per-genome reference detection is the appropriate mode
- Only one assembly needs validation without cross-assembler comparison

## Inputs

- Multiple genome assembly contig files in FASTA format (e.g. outputs from ABySS, Velvet, SPAdes, MEGAHIT)

## Outputs

- report.tsv (tab-separated summary table with the four assemblies as columns and ~30 reference-free metrics as rows)
- report.txt/report.html/report.pdf with the comparison table and plots
- Three reference-free assessment plots (contiguity and completeness)

## How to apply

Install QUAST from the ablab/quast GitHub repository (Python3 3.3+, GCC 4.7+, zlib dev files; setup.py install is optional since QUAST compiles sub-parts on first use). Run quast.py on all assembly FASTA files together without a reference (-o quast_output) so ~30 reference-free quality metrics and three assessment plots are computed for all assemblies in one table. Parse the tab-separated report.tsv and extract the key metrics: number of contigs, total length, largest contig, N50, N90, auN, G+C content, and # N's per 100 kbp. Judge the best assembler by contiguity metrics (best N50/N90/auN and largest contig), fragmentation (lowest contig count at similar total length), and plausibility checks such as G+C content within the expected organism range (e.g. 50.4-50.8% for E. coli) and low uncalled-base counts.

## Related tools

- **QUAST** (Command-line engine computing the reference-free quality metrics and comparison table (report.tsv) for all assemblies) — https://github.com/ablab/quast
- **WebQUAST** (Web-server front end providing the same QUAST reference-free evaluation (30 metrics, three plots) with a graphical interface) — https://github.com/ablab/quast-website
- **BUSCO** (Completeness cross-check based on conservative genes, used alongside QUAST metrics to confirm assembly completeness (98.7% complete for all four E. coli assemblies))

## Examples

```
python3 quast.py abyss_contigs.fasta velvet_contigs.fasta spades_contigs.fasta megahit_contigs.fasta -o quast_output
```

## Evaluation signals

- report.tsv exists, is tab-separated, and contains one column per input assembly (e.g. four assemblies as columns)
- Contig counts fall in the expected ranges, e.g. 176 contigs for ABySS versus 90-95 for Velvet, SPAdes and MEGAHIT
- Total assembly lengths are consistent, e.g. 4.8 Mb (ABySS) versus 4.6 Mb for the other assemblers
- The best assembler achieves the best N50, N90 and auN values and the largest contig (e.g. SPAdes, 285 kb versus 265, 248 and 236 kb)
- G+C content lies within the expected organism range (50.4-50.8% for E. coli, observed ~50.7%) and uncalled-base counts match expectations (~94 N's per 100 kbp for Velvet, none for MEGAHIT)

## Limitations

- Without a reference, QUAST reports mainly contiguity and completeness (30 metrics); correctness-related reference-based metrics (>60 metrics, Icarus viewers) are unavailable
- QUAST results are much more informative when at least a close reference genome is provided
- PDF plots require Matplotlib (v1.1+, fully tested with v1.3.1), and optional submodules have extra dependencies (GeneMark-ES needs Time::HiRes; GRIDSS needs Java 1.8+ and R)

## Evidence

- [results] When a reference genome is unavailable, WebQUAST computes 30 quality metrics and draws three assessment plots that mainly address the contiguity and completeness: "When a reference genome is unavailable, WebQUAST computes 30 quality metrics and draws three assessment plots that mainly address the contiguity and completeness"
- [results] ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90-95 for Velvet, SPAdes and MEGAHIT): "ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT)"
- [readme] report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc): "report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc)"
- [readme] N50 (length of a contig, such that all the contigs of at least the same length together cover at least 50% of the assembly).: "N50 (length of a contig, such that all the contigs of at least the same length together cover at least 50% of the assembly)."
- [results] SPAdes assembled larger contigs on average (the best N50, N90 and auN, the area under the Nx curve, values with Velvet and MEGAHIT being close runner-ups): "SPAdes assembled larger contigs on average (the best N50, N90 and auN, the area under the Nx curve, values with Velvet and MEGAHIT being close runner-ups)"
