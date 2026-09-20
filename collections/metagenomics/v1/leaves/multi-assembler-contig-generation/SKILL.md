---
name: multi-assembler-contig-generation
description: Use when input is paired-end short-read FASTQ from a mixed microbial/viral
  community (e.g., E. coli K12 + phage Lambda test reads) and you want contigs assembled
  by SPAdes, metaSPAdes, and MEGAHIT together to maximize recovery of viral genomes
  before gene-prediction-based viral scoring.
when_to_use_negative:
- Input is already assembled long-read sequences or assembled contigs — use the '-A'
  flag instead of the '-a all3' short-read assembly route.
- Only single-end reads are available — the '-p' paired-end input path this skill
  depends on does not apply (use '-s' instead).
- The task is contig filtering rather than assembly — use virMine's filter parameters
  (-m, -M, -c, -cov) on existing contigs rather than re-running all three assemblers.
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: virMine 2.0
  role: Pipeline wrapper that orchestrates assembly via the '-a all3' flag and performs
    downstream gene prediction, BLAST annotation, and viral classification
  repo: https://github.com/putonti/virmine
- name: SPAdes 3.15.3
  role: One of three assemblers invoked by the all3 option for short-read assembly
- name: metaSPAdes
  role: Metagenomic assembly variant of SPAdes invoked by the all3 option
- name: MEGAHIT
  role: Third assembler invoked by the all3 option
- name: BBMap 38.94
  role: Computes coverage statistics for the coverage filter after assembly
- name: Docker
  role: Container runtime; only prerequisite, handles all other dependencies via the
    Dockerfile
  repo: https://hub.docker.com/repository/docker/genevievej16/virmine
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:34abab76e6b7a12cd29e9289e9b1f3ec42d9ae3a70601c9451eb6fe2d458efbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/putonti/virmine
  tools:
  - BBMap
  - BBMap 38.94
  - Docker
  - MEGAHIT
  - Python
  - SPAdes
  - SPAdes 3.15.3
  - metaSPAdes
  - virMine 2.0
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multi-assembler-contig-generation

## Summary

Generate a consensus set of assembled contigs from paired-end metagenomic reads by running virMine 2.0's 'all3' option, which invokes SPAdes, metaSPAdes, and MEGAHIT in a single pipeline, then consolidates assemblies into final_contigs.fasta for downstream viral classification.

## When to use

Use when input is paired-end short-read FASTQ from a mixed microbial/viral community (e.g., E. coli K12 + phage Lambda test reads) and you want contigs assembled by SPAdes, metaSPAdes, and MEGAHIT together to maximize recovery of viral genomes before gene-prediction-based viral scoring.

## When NOT to use

- Input is already assembled long-read sequences or assembled contigs — use the '-A' flag instead of the '-a all3' short-read assembly route.
- Only single-end reads are available — the '-p' paired-end input path this skill depends on does not apply (use '-s' instead).
- The task is contig filtering rather than assembly — use virMine's filter parameters (-m, -M, -c, -cov) on existing contigs rather than re-running all three assemblers.

## Inputs

- Paired-end short-read FASTQ files (e.g., test_data_R1.fastq, test_data_R2.fastq)
- Viral protein database FASTA (viral_aa.fasta)
- Nonviral protein database FASTA (nonviral_aa.fasta)
- Docker environment (Docker image virmine:latest or genevievej16/virmine:latest)

## Outputs

- final_contigs.fasta (consolidated contigs from SPAdes, metaSPAdes, and MEGAHIT assemblies)
- viral_contigs.fasta (contigs classified as viral by gene-content scoring)
- Predicted ORF files (all ORFs and viral ORFs only)
- BLAST output files (ORF annotation against viral and nonviral databases)
- Coverage statistics computed with BBMap 38.94

## How to apply

Run virMine.py inside the Docker image (genevievej16/virmine or a locally built virmine:latest image) with the '-a all3' assembler flag, providing paired-end FASTQ files via '-p', the viral and nonviral protein databases via '-v' and '-nv', and an output path via '-o'. The pipeline then performs read quality control, assembly with all three assemblers, coverage computation with BBMap 38.94, gene prediction, and BLAST-based annotation of predicted ORFs against the supplied viral and nonviral databases. Contigs are subsequently scored by the cellular/viral/unknown origin of their gene content, with viral contigs separated into viral_contigs.fasta. Correctness on the reference test data is verified by final_contigs.fasta containing exactly 3 contigs with 1 classified as viral. Adjust optional filters (-m minimum contig size, -M maximum contig size, -c minimum coverage, -cov minimum SPAdes coverage) to refine the contig set for the study system.

## Related tools

- **virMine 2.0** (Pipeline wrapper that orchestrates assembly via the '-a all3' flag and performs downstream gene prediction, BLAST annotation, and viral classification) — https://github.com/putonti/virmine
- **SPAdes 3.15.3** (One of three assemblers invoked by the all3 option for short-read assembly)
- **metaSPAdes** (Metagenomic assembly variant of SPAdes invoked by the all3 option)
- **MEGAHIT** (Third assembler invoked by the all3 option)
- **BBMap 38.94** (Computes coverage statistics for the coverage filter after assembly)
- **Docker** (Container runtime; only prerequisite, handles all other dependencies via the Dockerfile) — https://hub.docker.com/repository/docker/genevievej16/virmine

## Examples

```
python3 virMine.py -a all3 -p testFiles/test_data_R1.fastq testFiles/test_data_R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/testOutput
```

## Evaluation signals

- final_contigs.fasta contains exactly 3 contigs when run on the reference test data (artificial reads of E. coli K12 NC_000913.3 and phage Lambda J02459.1).
- viral_contigs.fasta contains exactly 1 contig classified as viral on the reference test data.
- Predicted ORF files and BLAST output files are present in the output folder.
- All three assemblers (SPAdes, metaSPAdes, MEGAHIT) completed without error — check Docker memory allocation if any step fails.
- Coverage statistics were produced by BBMap 38.94 as part of the coverage filter step.

## Limitations

- Docker is the only prerequisite, but all dependencies are handled by the Dockerfile — if any section of the program errors, check that the Docker container has enough memory resources allocated.
- The expected output (3 contigs, 1 viral) is calibrated to the specific artificial E. coli K12 + phage Lambda test reads; other datasets will yield different contig counts.
- Contig scoring depends on gene content origin (cellular, viral, or unknown) relative to the supplied viral and nonviral databases, so database currency and completeness affect classification.

## Evidence

- [abstract] Three methods for assembly are included in virMine, namely, SPAdes, metaSPAdes, and MEGAHIT: "Three methods for assembly are included in virMine, namely, SPAdes, metaSPAdes, and MEGAHIT"
- [abstract] read QC, assembly, and coverage computation: "read quality control is conducted, followed by assembly"
- [methods] expected test output of 3 contigs with 1 viral: "Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral"
- [methods] test data composition: [evidence span withheld — anchor status: not_found]
- [readme] example test-data invocation with all3: "python3 virMine.py -a all3 -p testFiles/test_data_R1.fastq testFiles/test_data_R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/testOutput"
- [readme] other output files: "Other output files include all predicted ORFs, the viral ORFs only, and all BLAST output files."
