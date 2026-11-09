---
name: paired-end-read-assembly
description: Use when raw paired-end Illumina reads (e.g. SRA FASTQ like ERR008613)
  must be converted into genome assembly contig FASTAs for comparison with QUAST/WebQUAST.
when_to_use_negative:
- Input is long reads or a hybrid dataset — the shortlisted tools here are short-read
  assemblers (ABySS, MEGAHIT, SPAdes, Velvet).
- The goal is only to evaluate existing assemblies — use QUAST/WebQUAST directly instead
  of re-running assembly.
- A finished reference-quality assembly is required — assembled contigs (90–176 contigs,
  ~4.6–4.8 Mb) are drafts suitable for comparison, not finished genomes.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: SRA Toolkit
  role: fasterq-dump downloads raw paired-end reads for ERR008613 as FASTQ
- name: FastQC
  role: pre-assembly quality check producing fastqc_report.html
- name: Trimmomatic
  role: PE-mode trimming of low-quality read ends before assembly
- name: ABySS
  role: assembler run with the GAGE-B recipe (multiple k-mer sizes) because default
    assembly was very poor
- name: MEGAHIT
  role: assembler run with default parameters on trimmed reads
- name: SPAdes
  role: assembler run with default/recommended parameters; yielded best N50, N90 and
    auN
- name: Velvet
  role: assembler run as velveth + velvetg with recommended parameters
- name: WebQUAST
  role: downstream evaluation of the four assemblies (30 metrics reference-free; >60
    metrics with reference)
  repo: https://github.com/ablab/quast-website
- name: QUAST
  role: core command-line quality assessment of the produced contig FASTAs
  repo: https://github.com/ablab/quast
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:00d0777b30f7fbcc4dad188cbf735f8f3a915d34929366a2202b92285f892d17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast-website
  tools:
  - ABySS
  - FastQC
  - MEGAHIT
  - QUAST
  - SPAdes
  - SRA Toolkit
  - Trimmomatic
  - Velvet
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# paired-end-read-assembly

## Summary

Generates draft genome assemblies from paired-end short reads by quality-trimming with Trimmomatic and assembling with multiple short-read assemblers (ABySS, MEGAHIT, SPAdes, Velvet), producing contig FASTAs later assessed with WebQUAST/QUAST metrics.

## When to use

Use when raw paired-end Illumina reads (e.g. SRA FASTQ like ERR008613) must be converted into genome assembly contig FASTAs for comparison with QUAST/WebQUAST.

## When NOT to use

- Input is long reads or a hybrid dataset — the shortlisted tools here are short-read assemblers (ABySS, MEGAHIT, SPAdes, Velvet).
- The goal is only to evaluate existing assemblies — use QUAST/WebQUAST directly instead of re-running assembly.
- A finished reference-quality assembly is required — assembled contigs (90–176 contigs, ~4.6–4.8 Mb) are drafts suitable for comparison, not finished genomes.

## Inputs

- Raw paired-end FASTQ reads from SRA (e.g. accession ERR008613, E. coli K-12 MG1655)
- fastqc_report.html from FastQC quality inspection
- Trimmed paired reads: trimmed_1P.fastq.gz and trimmed_2P.fastq.gz

## Outputs

- Four contig FASTA assemblies (one each from ABySS, MEGAHIT, SPAdes, Velvet)
- QUAST/WebQUAST evaluation reports (report.tsv, report.html, icarus.html) with metrics such as total length, contig count, N50, N90, auN

## How to apply

First download raw reads with fasterq-dump (SRA Toolkit) and inspect quality with FastQC; trim low-quality read ends with Trimmomatic in paired-end (PE) mode, keeping surviving pairs in trimmed_1P.fastq.gz and trimmed_2P.fastq.gz and discarding orphaned mates. Install the four most-cited short-read assemblers (ABySS, MEGAHIT, SPAdes, Velvet) via Bioconda and run each on the trimmed reads: MEGAHIT and SPAdes with default or documentation-recommended parameters, Velvet as velveth followed by velvetg with recommended parameters, and ABySS with the GAGE-B parameter recipe because its default assembly was of very poor quality. Each assembler yields a contig FASTA, and producing four assemblies from the same trimmed input enables fair comparison of contiguity metrics (N50, N90, auN, largest contig) in WebQUAST. Validate that every assembly FASTA exists and that total lengths and contig counts fall in the expected ranges for the organism.

## Related tools

- **SRA Toolkit** (fasterq-dump downloads raw paired-end reads for ERR008613 as FASTQ)
- **FastQC** (pre-assembly quality check producing fastqc_report.html)
- **Trimmomatic** (PE-mode trimming of low-quality read ends before assembly)
- **ABySS** (assembler run with the GAGE-B recipe (multiple k-mer sizes) because default assembly was very poor)
- **MEGAHIT** (assembler run with default parameters on trimmed reads)
- **SPAdes** (assembler run with default/recommended parameters; yielded best N50, N90 and auN)
- **Velvet** (assembler run as velveth + velvetg with recommended parameters)
- **WebQUAST** (downstream evaluation of the four assemblies (30 metrics reference-free; >60 metrics with reference)) — https://github.com/ablab/quast-website
- **QUAST** (core command-line quality assessment of the produced contig FASTAs) — https://github.com/ablab/quast

## Examples

```
trimmomatic PE reads_1.fq.gz reads_2.fq.gz trimmed_1P.fastq.gz /dev/null trimmed_2P.fastq.gz /dev/null && megahit -1 trimmed_1P.fastq.gz -2 trimmed_2P.fastq.gz -o megahit_out
```

## Evaluation signals

- trimmed_1P.fastq.gz and trimmed_2P.fastq.gz both exist, decompress to valid FASTQ, and contain equal numbers of reads.
- Four assembly FASTA files exist, one per assembler.
- Total assembly lengths fall in the ~4.6–4.8 Mb range for E. coli K-12 MG1655.
- Contig counts fall within 90–176: Velvet, SPAdes and MEGAHIT ~90–95 contigs; ABySS most fragmented at 176 contigs.
- FastQC and Trimmomatic runs completed without errors; SPAdes shows the largest contig (~285 kb vs 265/248/236 kb for Velvet/ABySS/MEGAHIT).

## Limitations

- ABySS's default assembly was of very poor quality, so it must be run with the GAGE-B recipe evaluating multiple k-mer sizes.
- Assemblers rely on different heuristics; even different settings of the same tool may produce substantially diverging assemblies.
- QUAST/assembly tools are mainly Linux command-line software with interfaces unsuitable for researchers with limited computational background (motivating WebQUAST).

## Evidence

- [methods] The four E. coli assemblies were produced with five most-cited short-read assemblers shortlisted to four tools, all installed via Bioconda: [evidence span withheld — anchor status: not_found]
- [methods] while ABySS used the GAGE-B recipe because its default assembly was of very poor quality: [evidence span withheld — anchor status: not_found]
- [results] ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT): "ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT)"
- [supplementary] The sequencing data for E. coli K-12 MG1655 dataset is available from the National Center for Biotechnology Information (NCBI) Sequence Read Archive under accession number ERR008613: "The sequencing data for E. coli K-12 MG1655 dataset is available from the National Center for Biotechnology Information (NCBI) Sequence Read Archive under accession number ERR008613"
- [intro] These tools rely on different heuristics that greatly vary their output. Moreover, even different settings of the same tool may result in substantially diverging assemblies.: "These tools rely on different heuristics that greatly vary their output. Moreover, even different settings of the same tool may result in substantially diverging assemblies."
