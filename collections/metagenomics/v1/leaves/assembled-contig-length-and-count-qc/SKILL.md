---
name: assembled-contig-length-and-count-qc
description: Use when you have newly generated contig FASTA files from one or more
  short-read assemblers and must verify they fall within expected total-length and
  contig-count ranges for a known-size genome before deeper evaluation.
when_to_use_negative:
- Input is raw sequencing reads (FASTQ) rather than assembled contig FASTA files —
  run the assemblers first
- Assemblies are metagenomic datasets where no single expected genome size exists
  — use MetaQUAST-style reference detection instead of fixed-length range checks
- Assemblies target large genomes (e.g. mammalian) where the small-genome metric set
  is uninformative — QUAST-LG is the appropriate extension
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: QUAST
  role: Computes the contig-level metrics (total length, contig count, N50, N90, auN,
    largest contig) and emits report.txt/report.tsv/plots for the assemblies
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web server wrapping QUAST; computes 30 reference-free quality metrics and
    three contiguity/completeness assessment plots for the submitted assemblies
  repo: https://github.com/ablab/quast-website
- name: ABySS
  role: Source of the most fragmented benchmark assembly (176 contigs, 4.8 Mb), run
    with the GAGE-B parameter recipe rather than defaults
- name: MEGAHIT
  role: Source of a benchmark assembly (~4.6 Mb, ~90–95 contigs), run with default
    parameters on trimmed reads
- name: SPAdes
  role: Source of the benchmark assembly with best N50/N90/auN and largest contig
    (285 kb)
- name: Velvet
  role: Source of a benchmark assembly (~4.6 Mb, ~90–95 contigs), run as velveth followed
    by velvetg
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:8d91455bf28a902b84d40b7b9d22751ea9cb12d7f2430042395b12ffbfca2d1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - ABySS
  - MEGAHIT
  - QUAST
  - SPAdes
  - Velvet
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Assembled-contig-length-and-count QC

## Summary

Sanity-check de novo genome assemblies at the contig level — total length, contig count, and contiguity metrics (N50, N90, auN, largest contig) — before downstream evaluation, using the expected ranges established in the WebQUAST E. coli K-12 MG1655 four-assembler benchmark (ABySS, MEGAHIT, SPAdes, Velvet).

## When to use

Use when you have newly generated contig FASTA files from one or more short-read assemblers and must verify they fall within expected total-length and contig-count ranges for a known-size genome before deeper evaluation.

## When NOT to use

- Input is raw sequencing reads (FASTQ) rather than assembled contig FASTA files — run the assemblers first
- Assemblies are metagenomic datasets where no single expected genome size exists — use MetaQUAST-style reference detection instead of fixed-length range checks
- Assemblies target large genomes (e.g. mammalian) where the small-genome metric set is uninformative — QUAST-LG is the appropriate extension

## Inputs

- Contig FASTA files, one per assembler (e.g. ABySS, MEGAHIT, SPAdes, Velvet outputs of E. coli K-12 MG1655 trimmed paired-end reads)

## Outputs

- Per-assembly total length and contig count values validated against expected ranges (~4.6–4.8 Mb total; 90–176 contigs)
- Contiguity metric table (N50, N90, auN, largest contig length) per assembly, e.g. QUAST report.txt/report.tsv
- Nx-curve and contiguity/completeness assessment plots

## How to apply

First confirm each assembly FASTA exists and is a parseable contig file. Compute total assembly length and contig count per assembler; for the E. coli K-12 MG1655 benchmark, valid outputs fall in the ~4.6–4.8 Mb total-length range with contig counts of 90–176, where Velvet, SPAdes and MEGAHIT are tightly clustered at ~90–95 contigs and ABySS (run with the GAGE-B recipe because its default assembly is very poor) is the most fragmented at 176 contigs. Then compute contiguity metrics — N50, N90, auN (area under the Nx curve) and largest-contig length — to rank the assemblies; in the benchmark SPAdes had the best N50/N90/auN and the largest contig (285 kb versus 265, 248 and 236 kb for Velvet, ABySS and MEGAHIT). Use QUAST/WebQUAST to generate these metrics in a report (report.txt / report.tsv) plus contiguity plots, so the raw length/count checks are backed by a standardized metric table comparable across assemblers.

## Related tools

- **QUAST** (Computes the contig-level metrics (total length, contig count, N50, N90, auN, largest contig) and emits report.txt/report.tsv/plots for the assemblies) — https://github.com/ablab/quast
- **WebQUAST** (Web server wrapping QUAST; computes 30 reference-free quality metrics and three contiguity/completeness assessment plots for the submitted assemblies) — https://github.com/ablab/quast-website
- **ABySS** (Source of the most fragmented benchmark assembly (176 contigs, 4.8 Mb), run with the GAGE-B parameter recipe rather than defaults)
- **MEGAHIT** (Source of a benchmark assembly (~4.6 Mb, ~90–95 contigs), run with default parameters on trimmed reads)
- **SPAdes** (Source of the benchmark assembly with best N50/N90/auN and largest contig (285 kb))
- **Velvet** (Source of a benchmark assembly (~4.6 Mb, ~90–95 contigs), run as velveth followed by velvetg)

## Examples

```
./quast.py contigs_abyss.fasta contigs_megahit.fasta contigs_spades.fasta contigs_velvet.fasta -o quast_test_output
```

## Evaluation signals

- Each assembly FASTA file exists and parses, and total assembly length falls in the ~4.6–4.8 Mb range for the E. coli K-12 MG1655 dataset
- Contig counts fall within the reported 90–176 range, with Velvet/SPAdes/MEGAHIT near ~90–95 and ABySS at 176 (most fragmented)
- The N50/N90/auN table ranks SPAdes best and identifies the largest contig (285 kb) — a cross-check that metrics were computed on the correct assemblies
- QUAST output directory contains report.txt, report.tsv, report.pdf/report.html and Icarus/icarus.html entry points as expected from the README
- Metric values are reproducible across WebQUAST and command-line QUAST runs on the same FASTA inputs

## Limitations

- ABySS with default parameters produced a very poor quality assembly; the GAGE-B recipe (which evaluates multiple k-mer sizes and selects the best assembly) was required, so default runs of some assemblers may fail the length/count sanity checks
- Reference-free contig metrics give only a one-sided view (contiguity and completeness); the article notes existing tools that 'provide just a one-sided view on the assembly quality' — a reference-based evaluation reports >60 metrics plus eight plots and two Icarus viewers
- The QUAST core pipeline requires Python 3.3+, Perl 5.6.0+, GCC 4.7+, GNU make/ar, and zlib dev files; PDF plots additionally require Matplotlib, so missing dependencies can block report generation

## Evidence

- [results] ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT): "ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT)"
- [results] SPAdes assembled larger contigs on average (the best N50, N90 and auN, the area under the Nx curve, values with Velvet and MEGAHIT being close runner-ups): "SPAdes assembled larger contigs on average (the best N50, N90 and auN, the area under the Nx curve, values with Velvet and MEGAHIT being close runner-ups)"
- [results] WebQUAST computes 30 quality metrics and draws three assessment plots that mainly address the contiguity and completeness: "When a reference genome is unavailable, WebQUAST computes 30 quality metrics and draws three assessment plots that mainly address the contiguity and completeness"
- [readme] N50 (length of a contig, such that all the contigs of at least the same length together cover at least 50% of the assembly).: "N50 (length of a contig, such that all the contigs of at least the same length together cover at least 50% of the assembly)."
- [readme] It evaluates genome/metagenome assemblies by computing various metrics.: "It evaluates genome/metagenome assemblies by computing various metrics."
- [other] ABySS used the GAGE-B recipe because its default assembly was of very poor quality.: [evidence span withheld — anchor status: not_found]
