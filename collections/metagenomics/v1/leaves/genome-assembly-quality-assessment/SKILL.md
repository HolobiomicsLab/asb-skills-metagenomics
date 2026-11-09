---
name: genome-assembly-quality-assessment
description: Use when FASTA contig files from one or more genome assemblers need multifaceted
  quality assessment and comparison, with or without a reference genome, as in the
  four E. coli K-12 MG1655 assembly evaluation.
when_to_use_negative:
- When no reference is available and you need reference-aware metrics (NGA50, genome
  fraction, misassembly coordinates) — reference-free mode only yields contiguity/completeness
  metrics
- When the goal is downstream structural/functional annotation or genome mining rather
  than assembly quality comparison
- When evaluating a metagenomic dataset with plain QUAST instead of the MetaQUAST
  extension
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: QUAST
  role: Core command-line engine computing contiguity, misassembly, NGA50, genome
    fraction, duplication ratio, G+C and N-content metrics and rendering the report
    files
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web server wrapping QUAST with a graphical interface that dispatches reference
    alignment, BUSCO screening, and gene-finding modules based on user configuration
  repo: https://www.ccb.uni-saarland.de/quast/
- name: BUSCO
  role: Conservative-gene screening used to estimate assembly completeness when the
    completeness option is selected
- name: Icarus
  role: Interactive visualizer for alignment and misassembly outputs, linked from
    icarus.html
  repo: https://github.com/ablab/quast
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:5e12f5a5bbcd62e3909beef8cbfc535661af86a5aeb4e20c5600d09cd9cbf60a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - BUSCO
  - Icarus
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# genome-assembly-quality-assessment

## Summary

Evaluate and compare genome assemblies using QUAST/WebQUAST, which computes contiguity metrics (N50, N90, auN, largest contig), completeness via BUSCO, and reference-based misassembly and NGA50 statistics, producing a report.tsv summary table with assessment plots and Icarus viewers.

## When to use

Use when FASTA contig files from one or more genome assemblers need multifaceted quality assessment and comparison, with or without a reference genome, as in the four E. coli K-12 MG1655 assembly evaluation.

## When NOT to use

- When no reference is available and you need reference-aware metrics (NGA50, genome fraction, misassembly coordinates) — reference-free mode only yields contiguity/completeness metrics
- When the goal is downstream structural/functional annotation or genome mining rather than assembly quality comparison
- When evaluating a metagenomic dataset with plain QUAST instead of the MetaQUAST extension

## Inputs

- Assembly contig files in FASTA format (e.g. contigs_1.fasta, contigs_2.fasta from ABySS, Velvet, SPAdes, MEGAHIT)
- Optional reference genome in FASTA/FASTA.GZ (e.g. NC_000913.3 for MG1655 or AP009048.1 for W3110)
- Optional gene positions file (-g genes.txt) and paired reads (-1/-2 FASTQ) for advanced metrics

## Outputs

- report.tsv — tab-separated summary table with per-assembly metrics for parsing/spreadsheets
- report.txt, report.tex, report.pdf (Matplotlib required), report.html — summary tables and interactive report
- icarus.html — Icarus main menu linking interactive contig alignment viewers
- contigs_reports/misassemblies_report and contigs_reports/unaligned_report (reference-based mode only)

## How to apply

Run quast.py on all assembly FASTA files together in a single invocation; omit -r for reference-free mode (30 quality metrics, three assessment plots) or provide a reference via -r for >60 metrics including misassembly counts, genome fraction %, duplication ratio, and NGA50. Optionally enable BUSCO-based completeness screening (--conserved-genes-finding) and gene prediction with GeneMark.hmm/GlimmerHMM. Parse report.tsv (tab-separated) to extract contig counts, total length, largest contig, N50/N90/auN, G+C content, and # N's per 100 kbp for cross-assembly comparison. When using a close reference (e.g. E. coli W3110 instead of MG1655), expect substantially more reported misassemblies and inspect contigs_reports/misassemblies_report to distinguish true structural variation from errors. For large genomes consider the QUAST-LG extension and MetaQUAST for metagenomic datasets.

## Related tools

- **QUAST** (Core command-line engine computing contiguity, misassembly, NGA50, genome fraction, duplication ratio, G+C and N-content metrics and rendering the report files) — https://github.com/ablab/quast
- **WebQUAST** (Web server wrapping QUAST with a graphical interface that dispatches reference alignment, BUSCO screening, and gene-finding modules based on user configuration) — https://www.ccb.uni-saarland.de/quast/
- **BUSCO** (Conservative-gene screening used to estimate assembly completeness when the completeness option is selected)
- **Icarus** (Interactive visualizer for alignment and misassembly outputs, linked from icarus.html) — https://github.com/ablab/quast

## Examples

```
quast.py contigs_1.fasta contigs_2.fasta -r reference.fasta.gz -o quast_output
```

## Evaluation signals

- report.tsv exists, is tab-separated, and contains the assemblies as columns with expected metric fields (N50, genome fraction %, # misassemblies)
- report.html and icarus.html exist and report.html is a valid interactive HTML file
- Cross-assembly invariants reproduce Use Case 1 ranges: ABySS 176 contigs vs 90–95 for Velvet/SPAdes/MEGAHIT; total length 4.8 vs 4.6 Mb; SPAdes best N50/N90/auN with 285 kb largest contig
- G+C content falls in the expected range (50.4–50.8% for E. coli; observed 50.7%)
- Close-reference evaluation (W3110) shows ~49 extensive misassemblies for one assembler vs ~8 for the others, with misassembly breakpoints largely shared across assemblies

## Limitations

- QUAST/WebQUAST inherited limitations of embedded tools available mainly on Linux; requires Python3 (3.3+), GCC 4.7+, zlib dev files, and Perl; PDF plots need Matplotlib, structural variation detection needs Java 1.8+ and R (GRIDSS)
- Reported misassemblies depend on reference choice: a close reference (W3110) inflates misassembly counts (49 vs 8) because true substrain structural variants are flagged, requiring breakpoint-overlap inspection
- No changelog/version information for WebQUAST is provided and no quantitative benchmark of the server itself is reported in the discussion

## Evidence

- [abstract] We present WebQUAST, a web server for multifaceted quality assessment and comparison of genome assemblies based on the state-of-the-art QUAST tool.: "We present WebQUAST, a web server for multifaceted quality assessment and comparison of genome assemblies based on the state-of-the-art QUAST tool."
- [results] ABySS produced the largest assembly but most fragmented: "ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT)"
- [results] SPAdes best contiguity: "SPAdes assembled larger contigs on average (the best N50, N90 and auN, the area under the Nx curve, values with Velvet and MEGAHIT being close runner-ups)"
- [results] reference-free metrics count: "When a reference genome is unavailable, WebQUAST computes 30 quality metrics and draws three assessment plots that mainly address the contiguity and completeness"
- [readme] report.tsv purpose: "report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc)"
- [readme] duplication ratio definition: "Duplication ratio, the total number of aligned bases in the assembly divided by the total number of those in the reference."
