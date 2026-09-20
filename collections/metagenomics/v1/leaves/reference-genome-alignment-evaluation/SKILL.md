---
name: reference-genome-alignment-evaluation
description: Use when one or more genome assemblies (FASTA contigs) need reference-based
  quality assessment, or when quantifying how evaluation metrics change when a close-reference
  genome is substituted for the true reference.
when_to_use_negative:
- No reference genome is available and only contiguity/completeness metrics are needed
  — use reference-free QUAST evaluation instead (30 metrics, three plots).
- The dataset is metagenomic — MetaQUAST with interspecies translocation handling
  is the appropriate extension, not plain reference-based QUAST.
- Evaluating large (e.g., mammalian) genomes — QUAST-LG is designed for that scale.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: QUAST
  role: Core command-line evaluation engine; run with -r reference and -o output to
    produce report.tsv and contigs_reports/misassemblies_report
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web server providing the same reference-based evaluation (>60 metrics, eight
    plots, two Icarus viewers) through a graphical interface
  repo: https://www.ccb.uni-saarland.de/quast/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:ae08732423ec2e32bbaa9d07b73dc8990715f86de713e9a5ff94828be790ad76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
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
yes_rate: 0.6153846153846154
n_claims: 13
---

# reference-genome-alignment-evaluation

## Summary

Evaluate genome assemblies against a user-provided reference genome using QUAST/WebQUAST, which reports >60 quality metrics (misassemblies, NGA50, genome fraction, duplication ratio) plus misassembly coordinates, and use a close reference (e.g., E. coli K-12 W3110, AP009048.1) to distinguish true structural variants from assembler errors.

## When to use

Use when one or more genome assemblies (FASTA contigs) need reference-based quality assessment, or when quantifying how evaluation metrics change when a close-reference genome is substituted for the true reference.

## When NOT to use

- No reference genome is available and only contiguity/completeness metrics are needed — use reference-free QUAST evaluation instead (30 metrics, three plots).
- The dataset is metagenomic — MetaQUAST with interspecies translocation handling is the appropriate extension, not plain reference-based QUAST.
- Evaluating large (e.g., mammalian) genomes — QUAST-LG is designed for that scale.

## Inputs

- Genome assembly contigs in FASTA format (e.g., four E. coli K-12 MG1655 assemblies from ABySS, Velvet, SPAdes, MEGAHIT)
- Reference genome FASTA (e.g., E. coli K-12 W3110, GenBank accession AP009048.1, from NCBI)

## Outputs

- report.tsv (tab-separated summary with misassembly counts, NGA50, genome fraction %, duplication ratio per assembler)
- contigs_reports/misassemblies_report with coordinates of extensive misassemblies (inversions, relocations, translocations, local)
- report.html / icarus.html with interactive assessment plots and Icarus viewers
- Cross-assembly misassembly breakpoint overlap quantifying shared structural variants

## How to apply

Run QUAST (or the WebQUAST server at https://www.ccb.uni-saarland.de/quast/) on all assemblies simultaneously, supplying the reference genome via the -r option (e.g., W3110 AP009048.1 downloaded from NCBI) and an output directory via -o. Parse the tab-separated report.tsv to extract per-assembler misassembly counts, NGA50, genome fraction, and duplication ratio. Read contigs_reports/misassemblies_report to obtain coordinates of extensive misassemblies (inversions, relocations, translocations, local) per assembly. Compute coordinate overlap of misassembly breakpoints across assemblies: if the same assembler shows ~49 extensive misassemblies versus ~8 for others under the close reference, and breakpoints largely coincide across assemblies, the events reflect true structural variants between substrains rather than errors. Interpretation point: a substantially inflated misassembly count under a close reference (49 vs 8 extensive misassemblies here) signals reference divergence, not necessarily worse assemblies.

## Related tools

- **QUAST** (Core command-line evaluation engine; run with -r reference and -o output to produce report.tsv and contigs_reports/misassemblies_report) — https://github.com/ablab/quast
- **WebQUAST** (Web server providing the same reference-based evaluation (>60 metrics, eight plots, two Icarus viewers) through a graphical interface) — https://www.ccb.uni-saarland.de/quast/

## Examples

```
./quast.py contigs_abyss.fasta contigs_velvet.fasta contigs_spades.fasta contigs_megahit.fasta -r w3110_AP009048.fasta -o quast_close_ref_output
```

## Evaluation signals

- report.tsv exists and parses as tab-separated with per-assembler rows for # misassemblies, NGA50, genome fraction (%), and duplication ratio.
- contigs_reports/misassemblies_report is present and lists coordinates classified as inversions, relocations, translocations, or local misassemblies.
- Under the close reference W3110, total extensive misassemblies rise from ~8 (true reference) to ~49 for the affected assembler, matching the Use Case 3 pattern.
- Misassembly breakpoint coordinates largely coincide across the four assemblies, indicating shared true structural variants rather than assembler-specific errors.
- Duplication ratio stays near 1 and genome fraction is consistent across assemblers, confirming alignment of contigs to the reference is sane.

## Limitations

- A close reference inflates misassembly counts (49 vs 8 extensive misassemblies) even for correct assemblies; misassemblies must be cross-checked for shared coordinates to distinguish true structural variants from errors.
- The QUAST package is 'much more informative if at least a close reference genome is provided' — reference-free runs yield only contiguity/completeness-oriented metrics.
- Command-line QUAST is limited to a few platforms (usually Linux) and has a CLI unsuitable for researchers with limited computational background; WebQUAST mitigates this.
- Full installation (setup.py install_full) is needed for SV-detection-based more precise misassembly detection (GRIDSS, requiring Java 1.8+ and R).

## Evidence

- [results] the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers: "the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers"
- [results] close reference W3110: "Here we used W3110, another E. coli K-12 substrain, as an example of a close reference"
- [supplementary] reference accessions: "and AP009048.1 for substrains MG1655 and W3110, respectively."
- [readme] much more informative with reference: "However, it is much more informative if at least a close reference genome is provided along with the assemblies."
- [readme] misassembly metrics: "* Numbers of misassemblies of different kinds (inversions, relocations, translocations, interspecies translocations (metaQUAST only) or local)."
