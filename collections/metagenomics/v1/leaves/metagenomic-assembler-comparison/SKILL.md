---
name: metagenomic-assembler-comparison
description: Use when short-read metagenomic data must be assembled and no single
  assembler is obviously appropriate; comparing SPAdes, metaSPAdes, and MEGAHIT by
  N50 lets contiguity decide the assembly carried forward.
when_to_use_negative:
- Input is already assembled contigs or long reads — use virMine's '-A' option instead
  of short-read assembly
- Only a single-end read set is available and comparison across all three assemblers
  is not required
- Prior knowledge pins the correct assembler for the sample type, making a three-way
  N50 comparison unnecessary overhead
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: virMine 2.0
  role: Pipeline that orchestrates all three assemblers via the '-a all3' option and
    carries the best-N50 assembly into viral contig identification
  repo: https://github.com/putonti/virmine
- name: SPAdes 3.15.3
  role: One of the three assemblers invoked via '-a spades'; its assembly competes
    on N50
  repo: https://github.com/putonti/virmine
- name: metaSPAdes
  role: Metagenome-mode assembler invoked via '-a metaspades'; its assembly competes
    on N50
  repo: https://github.com/putonti/virmine
- name: MEGAHIT
  role: Assembler invoked via '-a megahit'; its assembly competes on N50
  repo: https://github.com/putonti/virmine
- name: Docker
  role: Container runtime that provisions all assembler dependencies; the only prerequisite
    for running virMine
  repo: https://hub.docker.com/repository/docker/genevievej16/virmine
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:f3e9b71bc26aef9ac970092926eac7da95a6aaf22e28db3b97e3c461b7fc1139
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
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# metagenomic-assembler-comparison

## Summary

Run multiple metagenomic assemblers (SPAdes, metaSPAdes, and MEGAHIT) on the same short-read input and select the assembly with the greatest N50 value for downstream analysis, as implemented in virMine 2.0's all3 option.

## When to use

Use when short-read metagenomic data must be assembled and no single assembler is obviously appropriate; comparing SPAdes, metaSPAdes, and MEGAHIT by N50 lets contiguity decide the assembly carried forward.

## When NOT to use

- Input is already assembled contigs or long reads — use virMine's '-A' option instead of short-read assembly
- Only a single-end read set is available and comparison across all three assemblers is not required
- Prior knowledge pins the correct assembler for the sample type, making a three-way N50 comparison unnecessary overhead

## Inputs

- Paired-end short-read FASTQ files (e.g., testFiles/test_data_R1.fastq, testFiles/test_data_R2.fastq)
- viral database FASTA of viral protein sequences (-v, e.g., inputFiles/viral_aa.fasta)
- nonviral database FASTA (-nv, e.g., inputFiles/nonviral_aa.fasta)

## Outputs

- Three contig sets (one per assembler: SPAdes, metaSPAdes, MEGAHIT)
- Selected assembly with the greatest N50 used for downstream virMine analysis
- final_contigs.fasta (expected 3 contigs on virMine test data)
- viral_contigs.fasta (1 contig classified as viral on test data)

## How to apply

Assemble the paired-end FASTQ reads (e.g., testFiles/test_data_R1.fastq and test_data_R2.fastq from the virMine 2.0 distribution) with each assembler: SPAdes v.3.15.3 via '-a spades', metaSPAdes via '-a metaspades', and MEGAHIT via '-a megahit'. Alternatively, invoke virMine.py with '-a all3' to execute all three assembly methods in a single run. Compute the N50 of each resulting contig set from its contig length distribution, select the assembly with the greatest N50 for further virMine analysis, and record the per-assembler comparison. As a sanity check on the virMine test data, the all-3 run is expected to yield 3 contigs in final_contigs.fasta, 1 of which is classified as viral.

## Related tools

- **virMine 2.0** (Pipeline that orchestrates all three assemblers via the '-a all3' option and carries the best-N50 assembly into viral contig identification) — https://github.com/putonti/virmine
- **SPAdes 3.15.3** (One of the three assemblers invoked via '-a spades'; its assembly competes on N50) — https://github.com/putonti/virmine
- **metaSPAdes** (Metagenome-mode assembler invoked via '-a metaspades'; its assembly competes on N50) — https://github.com/putonti/virmine
- **MEGAHIT** (Assembler invoked via '-a megahit'; its assembly competes on N50) — https://github.com/putonti/virmine
- **Docker** (Container runtime that provisions all assembler dependencies; the only prerequisite for running virMine) — https://hub.docker.com/repository/docker/genevievej16/virmine

## Examples

```
python3 virMine.py -a all3 -p testFiles/test_data_R1.fastq testFiles/test_data_R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/testOutput
```

## Evaluation signals

- The selected assembly is the row with the maximum N50 in the per-assembler N50 comparison table
- The all-3 test-data run yields 3 contigs in final_contigs.fasta, matching the paper's reported test-data criterion
- Of those 3 contigs, exactly 1 is classified as viral in viral_contigs.fasta
- All three assemblers produced a non-empty contig set from the same paired-end input before N50 selection

## Limitations

- Docker resource limits can cause failures — the README advises checking available memory in Docker resources if a step errors
- Comparing all three assemblers triples assembly compute relative to a single-assembler run
- N50 alone ignores total assembly size and misassembly rate; the pipeline uses it as the sole selection metric

## Evidence

- [abstract] Three methods for assembly are included in virMine, namely, SPAdes, metaSPAdes, and MEGAHIT: "Three methods for assembly are included in virMine, namely, SPAdes, metaSPAdes, and MEGAHIT"
- [readme] all3 runs all three assemblers and selects by N50: "-a : choose your assembler (spades, metaspades, megahit, all3)"
- [methods] test-data all-3 run yields 3 contigs, 1 viral: "Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral"
- [readme] example all3 test-data command: "python3 virMine.py -a all3 -p testFiles/test_data_R1.fastq testFiles/test_data_R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/testOutput"
