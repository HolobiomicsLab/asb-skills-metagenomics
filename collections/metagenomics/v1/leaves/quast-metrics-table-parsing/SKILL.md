---
name: quast-metrics-table-parsing
description: Use when FASTA contig files from multiple genome assemblers (e.g. ABySS,
  Velvet, SPAdes, MEGAHIT) must be compared reference-free, and the QUAST report.tsv
  metrics table must be extracted and validated.
when_to_use_negative:
- A reference genome is available — QUAST then reports >60 quality metrics plus eight
  assessment plots and two Icarus viewers, so the reference-free 30-metric subset
  in report.tsv is not the right comparison basis
- The dataset is metagenomic or very large (mammalian-scale) — MetaQUAST or QUAST-LG
  extensions, not plain reference-free QUAST, are the appropriate toolkits
- The input is reads (FASTQ) or already-computed quality statistics rather than assembled
  contigs in FASTA format
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: QUAST
  role: Command-line engine that runs reference-free evaluation of the four assemblies
    and emits report.tsv for parsing
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web-server front end providing the same reference-free evaluation (30 metrics,
    three plots) on any platform
  repo: https://github.com/ablab/quast-website
- name: BUSCO
  role: Conservative-gene completeness estimate reported alongside the QUAST metrics
    (98.7% complete for all four E. coli assemblies)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:723f1e84cb0d7366d3d1a25c125a0770096859d64214f51f653ad4633d763a09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - BUSCO
  - QUAST
  - WebQUAST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# quast-metrics-table-parsing

## Summary

Parses the tab-separated report.tsv emitted by QUAST/WebQUAST after reference-free evaluation of genome assemblies, extracting contig-count, contiguity (N50/N90/auN, largest contig), length, G+C content and # N's per 100 kbp metrics into a comparison table across assemblers.

## When to use

Use when FASTA contig files from multiple genome assemblers (e.g. ABySS, Velvet, SPAdes, MEGAHIT) must be compared reference-free, and the QUAST report.tsv metrics table must be extracted and validated.

## When NOT to use

- A reference genome is available — QUAST then reports >60 quality metrics plus eight assessment plots and two Icarus viewers, so the reference-free 30-metric subset in report.tsv is not the right comparison basis
- The dataset is metagenomic or very large (mammalian-scale) — MetaQUAST or QUAST-LG extensions, not plain reference-free QUAST, are the appropriate toolkits
- The input is reads (FASTQ) or already-computed quality statistics rather than assembled contigs in FASTA format

## Inputs

- FASTA contig files from multiple assemblers (e.g. ABySS, Velvet, SPAdes, MEGAHIT outputs for E. coli K-12 MG1655)
- QUAST report.tsv (tab-separated summary table with metrics as rows and assemblies as columns)

## Outputs

- Parsed metrics table (report.tsv) with number of contigs, total length, largest contig, N50, N90, auN, G+C content and # N's per 100 kbp per assembly
- Validation verdict per metric against expected reference ranges

## How to apply

Run quast.py on all assembler FASTA files together without a reference (-o quast_output) so that 30 reference-free quality metrics and three assessment plots addressing contiguity and completeness are computed. Open the generated report.tsv, the tab-separated version intended for parsing, with the four assemblies as columns and metric names as rows. Extract the key metrics: number of contigs, total length, largest contig, N50, N90, auN, G+C content, and # N's per 100 kbp. Judge the result against expected patterns: for the four E. coli K-12 MG1655 assemblies, ABySS gives the largest (4.8 Mb vs 4.6 Mb) but most fragmented assembly (176 contigs vs 90-95), SPAdes has the best N50/N90/auN and the largest contig overall (285 kb), G+C falls in the expected E. coli range (50.4-50.8%), and uncalled bases differ by assembler (e.g. ~94 N's per 100 kb for Velvet, none for MEGAHIT). Cross-check every extracted value against these reported ranges before writing the final table.

## Related tools

- **QUAST** (Command-line engine that runs reference-free evaluation of the four assemblies and emits report.tsv for parsing) — https://github.com/ablab/quast
- **WebQUAST** (Web-server front end providing the same reference-free evaluation (30 metrics, three plots) on any platform) — https://github.com/ablab/quast-website
- **BUSCO** (Conservative-gene completeness estimate reported alongside the QUAST metrics (98.7% complete for all four E. coli assemblies))

## Examples

```
./quast.py abyss_contigs.fasta velvet_contigs.fasta spades_contigs.fasta megahit_contigs.fasta -o quast_output
```

## Evaluation signals

- report.tsv exists, is tab-separated, and contains the four assemblies as columns
- Contig count for ABySS is 176 while Velvet, SPAdes and MEGAHIT fall in the 90-95 range
- Total lengths are approximately 4.8 Mb (ABySS) versus 4.6 Mb for the other three assemblies
- SPAdes shows the best N50, N90 and auN values and the largest contig (285 kb vs 265, 248 and 236 kb)
- G+C content for all assemblies lies within the expected E. coli range 50.4-50.8% (around 50.7%)

## Limitations

- Reference-free evaluation yields only 30 metrics and three plots; more informative reference-based metrics (NGA50, genome fraction, misassemblies) require providing a reference genome
- QUAST output detail depends on options: k-mer stats and read statistics appear only with --k-mer-stats or when reads are supplied, and contigs_reports only when a reference is given
- Existing online evaluation tools may be limited to specific taxa or provide just a one-sided view of assembly quality, so single metrics should not be over-interpreted

## Evidence

- [results] reference-free 30 metrics: "When a reference genome is unavailable, WebQUAST computes 30 quality metrics and draws three assessment plots"
- [results] ABySS fragmentation finding: [evidence span withheld — anchor status: not_found]
- [results] SPAdes contiguity finding: "SPAdes assembled larger contigs on average (the best N50, N90 and auN, the area under the Nx curve, values with Velvet and MEGAHIT being close runner-ups)"
- [readme] report.tsv purpose: "report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc)"
- [readme] N50 definition: "N50 (length of a contig, such that all the contigs of at least the same length together cover at least 50% of the assembly)."
