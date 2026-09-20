---
name: assembly-metric-interpretation
description: Use when comparing genome assemblies with QUAST/WebQUAST against a true
  or close reference genome (e.g., E. coli K-12 MG1655 vs W3110) and you must interpret
  misassemblies, NGA50, genome fraction, or duplication ratio to select the assembly.
when_to_use_negative:
- No reference genome is available and only contiguity/completeness is needed — use
  the reference-free QUAST mode (30 metrics, three plots) instead of reference-based
  misassembly interpretation
- The reference is distant from the assembled organism rather than a close substrain
  — misassembly counts will conflate true divergence with errors and the close-reference
  interpretation logic does not apply
- Evaluation of metagenomic or very large (mammalian) genomes, where MetaQUAST or
  QUAST-LG extensions are the appropriate toolkit
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: WebQUAST
  role: Web server providing QUAST-based reference-based and reference-free assembly
    evaluation with >60 metrics, eight assessment plots and two Icarus viewers
  repo: https://github.com/ablab/quast-website
- name: QUAST
  role: Core command-line evaluation tool computing misassemblies, NGA50, genome fraction,
    and duplication ratio when a reference is given via -r
  repo: https://github.com/ablab/quast
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:3dd520813a7f47843846eabb80362c71e060a107c7e43e8f858f6b91efe8bdfd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast-website
  tools:
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# assembly-metric-interpretation

## Summary

Interpret QUAST/WebQUAST evaluation metrics (misassembly counts, NGA50, genome fraction, duplication ratio, N50/auN) to judge genome assembly quality and to distinguish true structural variants from assembly errors when a close reference genome is used.

## When to use

Use when comparing genome assemblies with QUAST/WebQUAST against a true or close reference genome (e.g., E. coli K-12 MG1655 vs W3110) and you must interpret misassemblies, NGA50, genome fraction, or duplication ratio to select the best assembly.

## When NOT to use

- No reference genome is available and only contiguity/completeness is needed — use the reference-free QUAST mode (30 metrics, three plots) instead of reference-based misassembly interpretation
- The reference is distant from the assembled organism rather than a close substrain — misassembly counts will conflate true divergence with errors and the close-reference interpretation logic does not apply
- Evaluation of metagenomic or very large (mammalian) genomes, where MetaQUAST or QUAST-LG extensions are the appropriate toolkit

## Inputs

- FASTA assemblies from multiple assemblers (e.g., Velvet, SPAdes, ABySS, MEGAHIT contigs)
- Reference genome in FASTA/GenBank format (e.g., NC_000913.3 for MG1655, AP009048.1 for W3110) passed via -r
- QUAST output files: report.tsv and contigs_reports/misassemblies_report

## Outputs

- Per-assembler table of misassembly counts (extensive and local), NGA50, genome fraction %, and duplication ratio
- Coordinates of extensive misassemblies (inversions, relocations, translocations, local) per assembly
- Overlap analysis of misassembly breakpoints across assemblies to identify shared (true structural variant) breakpoints

## How to apply

Run QUAST (or WebQUAST) on the assemblies with the reference provided via -r, then parse the tab-separated report.tsv to extract misassembly counts, NGA50, genome fraction, and duplication ratio per assembler; with a reference, WebQUAST reports >60 metrics versus 30 in reference-free mode. Read the contigs_reports/misassemblies_report to obtain coordinates of extensive misassemblies (inversions, relocations, translocations, local). When misassemblies spike against a close reference (e.g., 49 versus 8 extensive misassemblies when evaluating MG1655 assemblies against W3110), check whether breakpoint coordinates coincide across assemblies: shared coordinates indicate true structural variants between substrains rather than assembler errors. Interpret contiguity metrics in context — e.g., SPAdes had the best N50, N90 and auN while ABySS produced the largest (4.8 Mb versus 4.6 Mb) but most fragmented assembly (176 contigs versus 90–95).

## Related tools

- **WebQUAST** (Web server providing QUAST-based reference-based and reference-free assembly evaluation with >60 metrics, eight assessment plots and two Icarus viewers) — https://github.com/ablab/quast-website
- **QUAST** (Core command-line evaluation tool computing misassemblies, NGA50, genome fraction, and duplication ratio when a reference is given via -r) — https://github.com/ablab/quast

## Examples

```
quast.py velvet_contigs.fasta spades_contigs.fasta abyss_contigs.fasta megahit_contigs.fasta -r w3110_AP009048.1.fasta -o quast_close_ref
```

## Evaluation signals

- Validation pattern from Use Case 3: one assembler shows ~49 extensive misassemblies versus ~8 for the other three when evaluated against the close reference W3110
- Misassembly breakpoint coordinates largely coincide across the four assemblies, indicating true structural variants rather than assembler-specific errors
- report.tsv is machine-parseable tab-separated with one column per assembly containing misassembly, NGA50, genome fraction, and duplication ratio rows
- Duplication ratio near 1 indicates no massively over-represented contigs; significant excess signals repeat multiplicity overestimation
- Contiguity cross-check: best N50/N90/auN assembler should not automatically be the one with the largest total length (ABySS: 4.8 Mb, 176 contigs vs SPAdes: best Nx metrics)

## Limitations

- Misassembly counts increase substantially when a close reference (W3110) replaces the true reference (49 vs 8 extensive misassemblies), so raw misassembly numbers against a divergent reference overstate assembly errors
- QUAST intrinsically inherited limitations of embedded tools available mainly on Linux with command-line interfaces unsuitable for researchers with limited computational background (addressed by WebQUAST)
- Duplication ratio > 1 can arise from multiple reasons, including overestimating repeat multiplicities and overlaps between contigs

## Evidence

- [results] Evaluation against the close reference W3110 reports a substantially increased number of misassemblies compared to the true-reference evaluation: 49 versus 8 extensive misassemblies in total.: [evidence span withheld — anchor status: not_found]
- [readme] When a reference is given, QUAST reports misassembly counts, NGA50, genome fraction and duplication ratio.: "Numbers of misassemblies of different kinds (inversions, relocations, translocations, interspecies translocations (metaQUAST only) or local)."
- [readme] NGA50 is a reference-aware version of N50 calculated using aligned blocks instead of contigs.: "NGA50, a reference-aware version of N50 metric. It is calculated using aligned blocks instead of contigs."
- [results] the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers: "the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers"
- [results] ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT): "ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT)"
