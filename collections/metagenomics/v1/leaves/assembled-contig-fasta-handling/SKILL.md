---
name: assembled-contig-fasta-handling
description: Use when input is long reads or already-assembled contigs to supply via
  -A, or when short reads must be assembled by SPAdes, metaSPAdes, or MEGAHIT with
  the assembly chosen by N50.
when_to_use_negative:
- Input is raw short reads and you want virMine's own read QC followed by assembly
  — supply them with -p/-s instead of importing a contig FASTA via -A.
- The FASTA is a protein reference database (viral_aa.fasta or nonviral_aa.fasta)
  — these are scoring inputs via -v/-nv, not contig assemblies to handle with -A.
- The task is long-read assembly/polishing itself — virMine accepts supplied long-read/contig
  files but delegates assembly of short reads to SPAdes/metaSPAdes/MEGAHIT; it does
  not assemble long reads.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: virMine 2.0
  role: 'Pipeline driver: accepts contig FASTA via -A, orchestrates the three assemblers
    via -a, and applies the -m/-M/-c/-cov contig filters'
  repo: https://github.com/putonti/virmine
- name: SPAdes 3.15.3
  role: Assembler invoked with '-a spades'; its coverage feeds the '-cov' minimum
    SPAdes coverage filter
- name: metaSPAdes
  role: Assembler invoked with '-a metaspades' as one of the three candidate assemblies
- name: MEGAHIT
  role: Assembler invoked with '-a megahit' as one of the three candidate assemblies
- name: Docker
  role: Container runtime used to build or pull the virMine image; all other dependencies
    are handled by the Dockerfile
  repo: https://hub.docker.com/repository/docker/genevievej16/virmine
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:7984267bdad6f375ce2453602e6a9b335873840a75a432c2d4b2752a29e1bbaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/putonti/virmine
  tools:
  - Docker
  - MEGAHIT
  - Python
  - SPAdes
  - SPAdes 3.15.3
  - metaSPAdes
  - virMine 2.0
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# assembled-contig-fasta-handling

## Summary

Handling assembled contig FASTA within virMine 2.0: either supplying pre-assembled contigs/long reads via the -A option, or generating contigs from short reads using one of the three bundled assemblers (SPAdes 3.15.3, metaSPAdes, MEGAHIT) and carrying the greatest-N50 assembly into downstream gene prediction and viral scoring.

## When to use

Use when input is long reads or already-assembled contigs to supply via -A, or when short reads must be assembled by SPAdes, metaSPAdes, or MEGAHIT with the best assembly chosen by N50.

## When NOT to use

- Input is raw short reads and you want virMine's own read QC followed by assembly — supply them with -p/-s instead of importing a contig FASTA via -A.
- The FASTA is a protein reference database (viral_aa.fasta or nonviral_aa.fasta) — these are scoring inputs via -v/-nv, not contig assemblies to handle with -A.
- The task is long-read assembly/polishing itself — virMine accepts supplied long-read/contig files but delegates assembly of short reads to SPAdes/metaSPAdes/MEGAHIT; it does not assemble long reads.

## Inputs

- Pre-assembled contig or long-read FASTA file (inputFiles/assembled_contigs.fasta, supplied via -A)
- Paired-end short-read FASTQ files (e.g., testFiles/test_data_R1.fastq and testFiles/test_data_R2.fastq, supplied via -p)
- Viral protein database FASTA (inputFiles/viral_aa.fasta, -v)
- Nonviral protein database FASTA (inputFiles/nonviral_aa.fasta, -nv)

## Outputs

- final_contigs.fasta — assembled contigs (3 contigs expected for the all-3 test-data run)
- viral_contigs.fasta — contigs classified as viral (1 expected for the test data)
- Predicted ORF files (all predicted ORFs and viral ORFs only)
- BLAST output files
- n50_comparison.tsv — per-assembler N50 values and the selected assembly

## How to apply

Contig FASTA enters virMine 2.0 in two ways: pre-assembled contigs or long reads are passed with the -A flag (e.g., -A inputFiles/assembled_contigs.fasta), while raw paired-end FASTQ (-p) or single-end FASTQ (-s) are assembled by one of the three included methods — SPAdes v3.15.3 (-a spades), metaSPAdes (-a metaspades), or MEGAHIT (-a megahit); '-a all3' runs all three and selects the assembly with the greatest N50 for further analysis. Compute the N50 of each contig set from its contig length distribution and record the per-assembler comparison (e.g., n50_comparison.tsv), verifying the selected assembly is the maximum-N50 row. Resulting contigs then pass through optional filters — -m minimum contig size, -M maximum contig size, -c minimum coverage, -cov minimum SPAdes coverage — before gene prediction and scoring against the -v viral and -nv nonviral amino-acid databases. For Docker runs, place the FASTA/FASTQ files plus viral_aa.fasta and nonviral_aa.fasta in inputFiles before building the image (Option 1), or in the mounted virmineDockerOutputFolder when pulling genevievej16/virmine from Docker Hub (Option 2).

## Related tools

- **virMine 2.0** (Pipeline driver: accepts contig FASTA via -A, orchestrates the three assemblers via -a, and applies the -m/-M/-c/-cov contig filters) — https://github.com/putonti/virmine
- **SPAdes 3.15.3** (Assembler invoked with '-a spades'; its coverage feeds the '-cov' minimum SPAdes coverage filter)
- **metaSPAdes** (Assembler invoked with '-a metaspades' as one of the three candidate assemblies)
- **MEGAHIT** (Assembler invoked with '-a megahit' as one of the three candidate assemblies)
- **Docker** (Container runtime used to build or pull the virMine image; all other dependencies are handled by the Dockerfile) — https://hub.docker.com/repository/docker/genevievej16/virmine

## Examples

```
python3 virMine.py -a all3 -p testFiles/test_data_R1.fastq testFiles/test_data_R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/testOutput
```

## Evaluation signals

- The all-3 test-data run yields 3 contigs in final_contigs.fasta, with 1 of those contigs classified as viral in viral_contigs.fasta.
- The assembly carried forward is the row with the maximum N50 in the per-assembler N50 comparison table.
- N50 values are computed from each assembly's contig length distribution before selection, not from read-level statistics.
- Contigs surviving the pipeline respect any applied -m/-M/-c thresholds (minimum/maximum contig size, minimum coverage).
- A '-A' run consumes the supplied contig FASTA directly and skips read QC/assembly, producing the same downstream ORF, BLAST, and scoring outputs.

## Limitations

- Docker resource limits can cause failures — 'If any section of the program causes an error or is unable to run, check that you have enough memory in your Docker resources.'
- Input placement is rigid: with Option 1, FASTQ and viral/nonviral databases must be moved to inputFiles before building the image; with Option 2, they must be in the mounted virmineDockerOutputFolder.
- Contig filtering is optional; without -m/-M/-c/-cov thresholds, every assembled contig proceeds to gene prediction and viral scoring.

## Evidence

- [abstract] Three methods for assembly are included in virMine, namely, SPAdes, metaSPAdes, and MEGAHIT: "Three methods for assembly are included in virMine, namely, SPAdes, metaSPAdes, and MEGAHIT"
- [readme] Input can be either: (1) raw sequencing reads from short-read sequencing technologies or (2) long-read sequences or assembled contigs.: "Input can be either: (1) raw sequencing reads from short-read sequencing technologies or (2) long-read sequences or assembled contigs."
- [readme] -A : list your long-read/contig assembly file: "-A : list your long-read/contig assembly file"
- [methods] Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral: "Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral"
- [abstract] virMine filters include minimum and/or maximum contig length, minimum contig coverage, and the presence of sequences of interest: "virMine filters include minimum and/or maximum contig length, minimum contig coverage, and the presence of sequences of interest"
- [readme] If any section of the program causes an error or is unable to run, check that you have enough memory in your Docker resources.: "If any section of the program causes an error or is unable to run, check that you have enough memory in your Docker resources."
