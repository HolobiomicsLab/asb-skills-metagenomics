---
name: reference-based-misassembly-detection
description: Use when assembled contig FASTA files and a reference genome (true or
  close, e.g. W3110 for E. coli K-12) are available and you need misassembly counts,
  NGA50, genome fraction or duplication ratio to compare assemblies.
when_to_use_negative:
- No reference genome is available or relevant — use WebQUAST/QUAST in reference-free
  mode, which computes 30 contiguity/completeness metrics instead of >60 reference-based
  ones
- Input is raw sequencing reads rather than assembled contigs — QUAST evaluates assemblies,
  not reads
- Comparing against a very distant or wrong reference where misassembly counts reflect
  true structural differences rather than assembly errors
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: QUAST
  role: Core engine invoked via quast.py; its reference-based module aligns contigs
    with minimap2-style alignment, detects misassemblies, and computes NGA50, genome
    fraction and duplication ratio
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web server wrapping QUAST's reference-based evaluation; conditionally invokes
    the reference module when a reference genome is uploaded and bundles the report
    for download
  repo: https://github.com/ablab/quast-website
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:3e023ee45963e6603f6fc80ab9b2b736aa4e6dc23e48ddf9afc55ecb6d147040
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - BUSCO
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# reference-based-misassembly-detection

## Summary

Detects and characterizes structural assembly errors (inversions, relocations, translocations, local misassemblies) by aligning contigs to a provided reference genome with QUAST/WebQUAST, yielding misassembly counts, NGA50, genome fraction and duplication ratio in report.tsv and per-assembly contigs_reports/misassemblies_report files.

## When to use

Use when assembled contig FASTA files and a reference genome (true or close, e.g. W3110 for E. coli K-12) are available and you need misassembly counts, NGA50, genome fraction or duplication ratio to compare assemblies.

## When NOT to use

- No reference genome is available or relevant — use WebQUAST/QUAST in reference-free mode, which computes 30 contiguity/completeness metrics instead of >60 reference-based ones
- Input is raw sequencing reads rather than assembled contigs — QUAST evaluates assemblies, not reads
- Comparing against a very distant or wrong reference where misassembly counts reflect true structural differences rather than assembly errors

## Inputs

- Assembled contig FASTA files (one or more assemblies to compare)
- Reference genome FASTA (e.g. GenBank accession AP009048.1 for E. coli K-12 W3110, NC_000913.3 for MG1655), optionally gzipped

## Outputs

- report.tsv (tab-separated metrics: # misassemblies, NGA50, genome fraction %, duplication ratio, mismatches, indels)
- contigs_reports/misassemblies_report (coordinates of inversions, relocations, translocations, local misassemblies per assembly)
- report.html and icarus.html (interactive report and Icarus viewer menu)
- report.txt, report.tex, report.pdf (if Matplotlib installed)

## How to apply

Run quast.py on the contig FASTA files with the reference supplied via the -r option and output directory set with -o (e.g. 'quast.py contigs_1.fasta contigs_2.fasta -r reference.fasta.gz -o quast_output'). QUAST's reference-based module aligns contigs to the reference to detect extensive misassemblies (inversions, relocations, translocations) and local misassemblies, counts mismatches and indels, and computes genome fraction %, duplication ratio and the reference-aware NGA50. Parse report.tsv for per-assembly metric values and read contigs_reports/misassemblies_report for misassembly coordinates. To compare across assemblers, extract misassembly, NGA50, genome fraction and duplication ratio per assembler, and optionally compute coordinate overlap of misassembly breakpoints across assemblies to quantify shared breakpoints. Distinguish true structural variants from errors when a close reference (a different substrain) is used, since a close reference inflates misassembly counts substantially (49 vs 8 extensive misassemblies in the E. coli W3110 use case).

## Related tools

- **QUAST** (Core engine invoked via quast.py; its reference-based module aligns contigs with minimap2-style alignment, detects misassemblies, and computes NGA50, genome fraction and duplication ratio) — https://github.com/ablab/quast
- **WebQUAST** (Web server wrapping QUAST's reference-based evaluation; conditionally invokes the reference module when a reference genome is uploaded and bundles the report for download) — https://github.com/ablab/quast-website

## Examples

```
quast.py contigs_1.fasta contigs_2.fasta -r reference.fasta.gz -o quast_output
```

## Evaluation signals

- report.tsv exists and contains expected metric fields (N50, genome fraction %, # misassemblies, NGA50, duplication ratio)
- contigs_reports/misassemblies_report exists and lists misassembly coordinates per assembly
- report.html is a valid interactive HTML file and icarus.html exists with links to Icarus viewers
- Number of misassembly categories reported covers inversions, relocations, translocations and local misassemblies
- When using a close reference, expect substantially inflated misassembly counts (e.g. ~49 vs ~8 extensive misassemblies) with breakpoints largely coinciding across assemblies

## Limitations

- Evaluation against a close reference (different substrain) reports substantially more misassemblies than against the true reference (49 vs 8 extensive), so counts conflate true structural variants with assembly errors
- Reference-based metrics require the reference to be close to the assembled organism; the package 'is much more informative if at least a close reference genome is provided'
- PDF report rendering requires Matplotlib (v1.1+ recommended; fully tested with v1.3.1)

## Evidence

- [readme] Numbers of misassemblies of different kinds (inversions, relocations, translocations, interspecies translocations (metaQUAST only) or local).: "Numbers of misassemblies of different kinds (inversions, relocations, translocations, interspecies translocations (metaQUAST only) or local)."
- [results] Here we used W3110, another E. coli K-12 substrain, as an example of a close reference: "Here we used W3110, another E. coli K-12 substrain, as an example of a close reference"
- [results] Evaluation against the close reference W3110 reports a substantially increased number of misassemblies compared to the true-reference evaluation: 49 versus 8 extensive misassemblies in total.: [evidence span withheld — anchor status: not_found]
- [readme] NGA50, a reference-aware version of N50 metric. It is calculated using aligned blocks instead of contigs.: "NGA50, a reference-aware version of N50 metric. It is calculated using aligned blocks instead of contigs."
- [readme] The QUAST package works both with and without reference genomes. However, it is much more informative if at least a close reference genome is provided along with the assemblies.: "The QUAST package works both with and without reference genomes. However, it is much more informative if at least a close reference genome is provided along with the assemblies."
