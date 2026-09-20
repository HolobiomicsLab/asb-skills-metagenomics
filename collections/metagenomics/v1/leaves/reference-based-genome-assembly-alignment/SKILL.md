---
name: reference-based-genome-assembly-alignment
description: Use when a gold-standard or close reference genome (e.g., E. coli K-12
  MG1655, NC_000913.3) is available and multiple assemblies must be compared on structural
  errors and per-base accuracy.
when_to_use_negative:
- No reference genome (true or close) is available for the organism — use reference-free
  evaluation instead, which computes 30 quality metrics
- Assemblies are of a large (e.g., mammalian) genome where standard QUAST is inappropriate
  — QUAST-LG is the intended extension
- The dataset is metagenomic — MetaQUAST, not plain reference-based QUAST, is the
  appropriate extension
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: QUAST
  role: Core engine that aligns contigs to the reference, splits alignments at misassembly
    breakpoints, and computes reference-based metrics; run via -r with optional -1/-2
    reads
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web server providing the same reference-based QUAST functionality through
    a graphical interface, reporting >60 quality metrics, eight assessment plots and
    two Icarus viewers
  repo: https://github.com/ablab/quast-website
- name: Icarus
  role: Interactive visualizer for inspecting reference-based evaluation results (contig
    alignment and misassembly viewers)
  repo: https://github.com/ablab/quast
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:472a12a1fa01a0bc1ad332d9a40553dd8ddf67eb390a6a90c8e41d98edaa6620
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - Icarus
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# reference-based-genome-assembly-alignment

## Summary

Evaluate genome assemblies by aligning contigs to a known reference genome with QUAST (via WebQUAST or the command line), yielding >60 reference-based metrics such as extensive misassemblies, genome fraction %, duplication ratio, NGA50, and mismatches/indels per 100 kb.

## When to use

Use when a gold-standard or close reference genome (e.g., E. coli K-12 MG1655, NC_000913.3) is available and multiple assemblies must be compared on structural errors and per-base accuracy.

## When NOT to use

- No reference genome (true or close) is available for the organism — use reference-free evaluation instead, which computes 30 quality metrics
- Assemblies are of a large (e.g., mammalian) genome where standard QUAST is inappropriate — QUAST-LG is the intended extension
- The dataset is metagenomic — MetaQUAST, not plain reference-based QUAST, is the appropriate extension

## Inputs

- FASTA assemblies (contigs) from multiple assemblers (e.g., MEGAHIT, SPAdes, Velvet, ABySS)
- Reference genome FASTA (e.g., NC_000913.3 for E. coli K-12 MG1655) supplied via -r
- Paired-end sequencing reads (FASTQ, e.g., SRA accession ERR008613) supplied via -1/-2
- Optional gene annotation from NCBI supplied via -g

## Outputs

- report.tsv (tab-separated summary of >60 quality metrics, for parsing/spreadsheets)
- report.txt / report.pdf / report.html summary tables and plots
- contigs_reports/misassemblies_report (detailed misassembly report)
- contigs_reports/unaligned_report (unaligned and partially unaligned contigs)
- icarus.html interactive visualizer

## How to apply

Download the reference genome and gene annotation from NCBI (e.g., NC_000913.3 / GCF_000005845.2 for E. coli K-12 MG1655) and run quast.py on all assemblies with the reference supplied via -r and read pairs via -1/-2 so that read-pair-based misassembly detection is enabled. QUAST aligns the contigs to the reference, splits alignments at misassembly breakpoints, and computes reference-based metrics: numbers of extensive misassemblies (relocations, translocations, inversions), genome fraction %, duplication ratio, NGA50, and mismatches/indels per 100 kb. Export the combined summary as report.tsv (the tab-separated version for parsing) and inspect contigs_reports/misassemblies_report for detailed structural errors. Rationale: reference-based evaluation is much more informative than reference-free mode, catching structural errors invisible to contiguity metrics like N50.

## Related tools

- **QUAST** (Core engine that aligns contigs to the reference, splits alignments at misassembly breakpoints, and computes reference-based metrics; run via -r with optional -1/-2 reads) — https://github.com/ablab/quast
- **WebQUAST** (Web server providing the same reference-based QUAST functionality through a graphical interface, reporting >60 quality metrics, eight assessment plots and two Icarus viewers) — https://github.com/ablab/quast-website
- **Icarus** (Interactive visualizer for inspecting reference-based evaluation results (contig alignment and misassembly viewers)) — https://github.com/ablab/quast

## Examples

```
quast.py megahit.fasta spades.fasta velvet.fasta abyss.fasta -r NC_000913.3.fasta -1 reads1.fastq.gz -2 reads2.fastq.gz -o quast_out
```

## Evaluation signals

- report.tsv fields match known Use Case 2 values: 0 extensive misassemblies for MEGAHIT and SPAdes versus 4 each for Velvet and ABySS
- ABySS genome fraction of 98.7% versus 98.0–98.4% for the other three assemblers
- Per-base accuracy expected around 1.0 mismatches and 0.3 indels per 100 kb for the best assembler (SPAdes)
- contigs_reports/misassemblies_report exists and details relocations/translocations/inversions whenever a reference was supplied via -r
- Duplication ratio is close to 1 for well-assembled data; large deviations indicate overestimated repeat multiplicities or duplicated contigs

## Limitations

- QUAST is much more informative with a reference, but evaluating against a close reference (e.g., W3110 for MG1655) rather than the true reference changes structural-error interpretation
- Misassembly detection without reads is alignment-based only; full read-pair-based misassembly detection requires supplying reads via -1/-2
- Structural variation detection based on read pairs (GRIDSS) requires Java 1.8+ and R; GeneMark-ES gene finding requires the Time::HiRes perl module
- The command-line tool is Linux-oriented with a CLI unsuitable for researchers with limited computational background — WebQUAST addresses this

## Evidence

- [other] Run quast.py on the four assemblies ... with the reference supplied via the -r flag and reads supplied via -1/-2 to enable full reference-based and read-pair-based misassembly detection.: [evidence span withheld — anchor status: not_found]
- [results] the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers: "the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers"
- [readme] Numbers of mismatches and indels, over the assembly and per 100 kb.: "Numbers of mismatches and indels, over the assembly and per 100 kb."
- [readme] it is much more informative if at least a close reference genome is provided along with the assemblies: "it is much more informative if at least a close reference genome is provided along with the assemblies"
- [results] Here we used W3110, another E. coli K-12 substrain, as an example of a close reference: "Here we used W3110, another E. coli K-12 substrain, as an example of a close reference"
