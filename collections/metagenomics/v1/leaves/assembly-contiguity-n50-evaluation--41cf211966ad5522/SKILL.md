---
name: assembly-contiguity-n50-evaluation
description: Use when short-read metagenomic FASTQ inputs are being assembled within
  virMine and no single assembler is obviously appropriate, so contiguity (N50) should
  decide which assembly proceeds to annotation and viral scoring.
when_to_use_negative:
- Input is already assembled contigs or long reads — use virMine's '-A' input mode
  instead of running short-read assemblers
- The study prescribes a single assembler a priori, making an N50 comparison across
  SPAdes/metaSPAdes/MEGAHIT unnecessary
- Reads have not undergone virMine's read quality control step, which is conducted
  before assembly
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: virMine 2.0
  role: Wraps the three assemblers via the '-a' option (spades, metaspades, megahit,
    all3) and carries the greatest-N50 assembly into annotation and viral contig scoring
  repo: https://github.com/putonti/virmine
- name: SPAdes 3.15.3
  role: One of the three candidate short-read assemblers invoked through virMine ('-a
    spades'; also defines the '-cov' minimum SPAdes coverage filter)
  repo: https://github.com/putonti/virmine
- name: metaSPAdes
  role: Metagenome-mode assembler candidate invoked through virMine ('-a metaspades')
  repo: https://github.com/putonti/virmine
- name: MEGAHIT
  role: Third assembler candidate invoked through virMine ('-a megahit')
  repo: https://github.com/putonti/virmine
- name: Docker
  role: Execution environment; the only prerequisite since the Dockerfile handles
    all other dependencies
  repo: https://hub.docker.com/repository/docker/genevievej16/virmine
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:dd1a14a339156c26e96ac7fea453c6efad71455b44edf209b9b4f2a70bd9dc0c
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
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# assembly-contiguity-n50-evaluation

## Summary

Run all three assemblers bundled with virMine 2.0 (SPAdes v3.15.3, metaSPAdes, MEGAHIT) on the same short-read FASTQ inputs, compute the N50 of each resulting contig set, and select the assembly with the greatest N50 for downstream viral contig scoring.

## When to use

Use when short-read metagenomic FASTQ inputs are being assembled within virMine and no single assembler is obviously appropriate, so contiguity (N50) should decide which assembly proceeds to annotation and viral scoring.

## When NOT to use

- Input is already assembled contigs or long reads — use virMine's '-A' input mode instead of running short-read assemblers
- The study prescribes a single assembler a priori, making an N50 comparison across SPAdes/metaSPAdes/MEGAHIT unnecessary
- Reads have not undergone virMine's read quality control step, which is conducted before assembly

## Inputs

- Paired-end short-read FASTQ files (e.g., testFiles/test_data_R1.fastq, testFiles/test_data_R2.fastq)
- Viral protein database FASTA (-v, e.g., inputFiles/viral_aa.fasta)
- Nonviral protein database FASTA (-nv, e.g., inputFiles/nonviral_aa.fasta)

## Outputs

- Three contig sets (one each from SPAdes, metaSPAdes, MEGAHIT)
- N50 comparison per assembler (e.g., n50_comparison.tsv)
- Selected assembly with the greatest N50, carried forward as final_contigs.fasta (test data: 3 contigs, 1 viral in viral_contigs.fasta)

## How to apply

Assemble the same paired-end FASTQ files (e.g., testFiles/test_data_R1.fastq and test_data_R2.fastq) with each of virMine's three assemblers by invoking virMine.py with '-a spades', '-a metaspades', and '-a megahit' — or in a single run with '-a all3', which executes all three methods and keeps the assembly with the greatest N50 value for further analysis. Compute the N50 of each contig set from its contig length distribution (e.g., writing a per-assembler comparison such as n50_comparison.tsv). Select the assembly whose row has the maximum N50. As a validation anchor, virMine's artificial E. coli K12 / Escherichia phage Lambda test reads should yield 3 contigs in final_contigs.fasta under the all-3 option, with 1 classified as viral in viral_contigs.fasta.

## Related tools

- **virMine 2.0** (Wraps the three assemblers via the '-a' option (spades, metaspades, megahit, all3) and carries the greatest-N50 assembly into annotation and viral contig scoring) — https://github.com/putonti/virmine
- **SPAdes 3.15.3** (One of the three candidate short-read assemblers invoked through virMine ('-a spades'; also defines the '-cov' minimum SPAdes coverage filter)) — https://github.com/putonti/virmine
- **metaSPAdes** (Metagenome-mode assembler candidate invoked through virMine ('-a metaspades')) — https://github.com/putonti/virmine
- **MEGAHIT** (Third assembler candidate invoked through virMine ('-a megahit')) — https://github.com/putonti/virmine
- **Docker** (Execution environment; the only prerequisite since the Dockerfile handles all other dependencies) — https://hub.docker.com/repository/docker/genevievej16/virmine

## Examples

```
python3 virMine.py -a all3 -p testFiles/test_data_R1.fastq testFiles/test_data_R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/testOutput
```

## Evaluation signals

- The selected assembly is the row with the maximum N50 in the per-assembler N50 comparison table
- The all-3 run on the bundled test reads yields 3 contigs in final_contigs.fasta, matching the paper's reported test-data criterion
- Exactly 1 of the 3 test-data contigs is classified as viral in viral_contigs.fasta
- All three assemblers (SPAdes, metaSPAdes, MEGAHIT) produced a contig set from the identical FASTQ inputs
- Output files include the predicted ORFs, viral ORFs only, and BLAST output files as documented for the example run

## Limitations

- Insufficient Docker memory allocation can cause sections of the program to error or fail to run — check Docker resources
- N50 is a contiguity-only criterion; the greatest-N50 assembly is chosen regardless of other assembly quality metrics
- The test-data expectation (3 contigs, 1 viral) applies to the artificial E. coli K12 / Escherichia phage Lambda reads and may not transfer to real metagenomes

## Evidence

- [abstract] Three methods for assembly are included in virMine, namely, SPAdes, metaSPAdes, and MEGAHIT: "Three methods for assembly are included in virMine, namely, SPAdes, metaSPAdes, and MEGAHIT"
- [readme] assembler options: "-a : choose your assembler (spades, metaspades, megahit, all3)"
- [readme] test data expectation: "Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral"
- [readme] Docker prerequisite: "Docker is the only prerequisite for this program to run, all other dependencies are handled by the Dockerfile."
- [abstract] pipeline position of assembly: "read quality control is conducted, followed by assembly"
