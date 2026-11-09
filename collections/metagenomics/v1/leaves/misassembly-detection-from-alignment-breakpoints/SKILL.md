---
name: misassembly-detection-from-alignment-breakpoints
description: Use when you have multiple FASTA assemblies of a model organism plus
  a reference genome (e.g. NC_000913.3) and read pairs, and you need to count extensive
  misassemblies, genome fraction, duplication ratio, NGA50, and mismatches/indels
  per 100 kb.
when_to_use_negative:
- No reference genome (true or close substrain) is available for the organism — use
  QUAST's reference-free metrics (30 contiguity/completeness metrics) instead of misassembly
  detection
- Assemblies are of a metagenomic dataset — use MetaQUAST rather than single-reference
  misassembly evaluation
- The genome is very large (e.g. mammalian) — QUAST-LG is the intended extension rather
  than standard QUAST reference-based evaluation
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST
  role: Core command-line engine that aligns assemblies to the reference, splits alignments
    at misassembly breakpoints, and computes misassembly, genome fraction, NGA50,
    and mismatch/indel metrics
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web-server front end providing the same reference-based evaluation with a
    graphical interface and Icarus viewers for any platform
  repo: https://github.com/ablab/quast-website
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:5369b2e57538d41eb4f97fc7477e988749bdd56e2a17b0c89497bfe66bf927d9
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# misassembly-detection-from-alignment-breakpoints

## Summary

Detects large structural misassemblies (relocations, translocations, inversions) and per-base errors in genome assemblies by aligning contigs against a true or close reference genome with QUAST, optionally using read-pair SV detection for more precise breakpoints. Grounded in the WebQUAST/QUAST Use Case 2 evaluation of four E. coli K-12 MG1655 assemblies.

## When to use

Use when you have multiple FASTA assemblies of a model organism plus a reference genome (e.g. NC_000913.3) and read pairs, and you need to count extensive misassemblies, genome fraction, duplication ratio, NGA50, and mismatches/indels per 100 kb.

## When NOT to use

- No reference genome (true or close substrain) is available for the organism — use QUAST's reference-free metrics (30 contiguity/completeness metrics) instead of misassembly detection
- Assemblies are of a metagenomic dataset — use MetaQUAST rather than single-reference misassembly evaluation
- The genome is very large (e.g. mammalian) — QUAST-LG is the intended extension rather than standard QUAST reference-based evaluation

## Inputs

- FASTA assembly contigs for each assembler to compare (e.g. MEGAHIT, SPAdes, Velvet, ABySS outputs)
- Reference genome FASTA from NCBI (e.g. NC_000913.3 for E. coli K-12 MG1655, or AP009048.1 for W3110 as a close reference)
- Paired-end reads in FASTQ format (e.g. SRA accession ERR008613) for read-pair-based misassembly detection

## Outputs

- report.tsv (tab-separated summary table with >60 reference-based quality metrics for parsing)
- report.txt / report.pdf / report.html combined tables and plots
- icarus.html interactive Icarus viewers of alignments and misassembly breakpoints
- contigs_reports/misassemblies_report with detailed breakdown of relocations, translocations, and inversions
- contigs_reports/unaligned_report for unaligned and partially unaligned contigs

## How to apply

Download the reference genome and gene annotation from NCBI (for E. coli K-12 MG1655: NC_000913.3 / GCF_000005845.2), then run quast.py on all assemblies simultaneously, supplying the reference via -r and read pairs via -1/-2 so that QUAST performs reference-based alignment and read-pair-based misassembly detection. QUAST aligns contigs to the reference, splits alignments at misassembly breakpoints, and reports numbers of extensive misassemblies (relocations, translocations, inversions), genome fraction %, duplication ratio, NGA50, and mismatches/indels per 100 kb. Export the combined summary as report.tsv (the tab-separated version for parsing) and inspect the Icarus viewers and contigs_reports/misassemblies_report for breakpoint details. Validate results against expected values, e.g. in the WebQUAST E. coli K-12 MG1655 dataset MEGAHIT and SPAdes show 0 extensive misassemblies versus 4 each for Velvet and ABySS, with SPAdes best at ~1.0 mismatches and 0.3 indels per 100 kb. If only a close reference is available (e.g. the W3110 substrain AP009048.1), the same workflow applies but structural differences between substrains may inflate apparent misassemblies.

## Related tools

- **QUAST** (Core command-line engine that aligns assemblies to the reference, splits alignments at misassembly breakpoints, and computes misassembly, genome fraction, NGA50, and mismatch/indel metrics) — https://github.com/ablab/quast
- **WebQUAST** (Web-server front end providing the same reference-based evaluation with a graphical interface and Icarus viewers for any platform) — https://github.com/ablab/quast-website

## Examples

```
./quast.py contigs_1.fasta contigs_2.fasta -r reference.fasta.gz -1 reads1.fastq.gz -2 reads2.fastq.gz -o quast_output
```

## Evaluation signals

- report.tsv exists and its fields parse as a tab-separated table with one column per assembly
- Extensive misassembly counts are internally consistent with the misassemblies_report breakdown (relocations + translocations + inversions)
- Genome fraction % falls in a plausible range (e.g. ~98.0–98.7% for the E. coli K-12 assemblies) and duplication ratio near 1 for well-pruned assemblies
- Expected diff-style comparison reproduces Use Case 2 values: 0 extensive misassemblies for MEGAHIT and SPAdes versus 4 each for Velvet and ABySS
- Per-base error rates are per 100 kb normalized (SPAdes ≈ 1.0 mismatches, 0.3 indels per 100 kb), not raw counts

## Limitations

- Read-pair-based SV detection for more precise misassembly detection requires a full QUAST installation, Java 1.8 or later, and R (for GRIDSS)
- Reference-free mode cannot detect misassemblies; the workflow is much more informative when at least a close reference genome is provided
- Using a close but non-identical reference (e.g. W3110 instead of MG1655) changes the evaluation context and may confound true misassemblies with substrain variation
- QUAST is maintained primarily for Linux 64-bit and macOS with Python3, Perl, GCC, and zlib dependencies

## Evidence

- [readme] Numbers of misassemblies of different kinds (inversions, relocations, translocations, interspecies translocations (metaQUAST only) or local).: "Numbers of misassemblies of different kinds (inversions, relocations, translocations, interspecies translocations (metaQUAST only) or local)."
- [readme] Full installation includes SV detection tools for more precise misassembly detection.: "includes (1) tools for SV detection based on read pairs, which is used for more precise misassembly detection,"
- [results] Reference-based mode reports more than 60 quality metrics with plots and Icarus viewers.: "the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers"
- [results] Use Case 2 evaluates the same four E. coli K-12 MG1655 assemblies against the true reference genome.: "we evaluated the same four assemblies of the E. coli K-12 MG1655 dataset but selected the reference genome differently"
- [supplementary] Reference genome accessions for the E. coli K-12 substrains are available from NCBI.: "The E. coli strain K-12 reference genomes and gene annotations are available from NCBI under accession numbers NC_000913.3"
