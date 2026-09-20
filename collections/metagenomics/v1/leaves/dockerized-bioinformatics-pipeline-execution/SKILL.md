---
name: dockerized-bioinformatics-pipeline-execution
description: Use when a tool's README states Docker is the only prerequisite and all
  other dependencies (e.g., Python 3.9, BBMap, SPAdes, MEGAHIT) are handled by the
  Dockerfile, and you need to execute the full pipeline end-to-end.
when_to_use_negative:
- Input is already assembled contigs or long reads — use the -A input mode instead
  of the short-read assembly path
- The user cannot allocate sufficient memory to Docker resources, since the README
  notes pipeline sections will error when Docker memory is insufficient
- The target pipeline is not distributed as a Docker image or Dockerfile, so containerized
  execution is not actually available
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0797
tools:
- name: Docker
  role: 'Container runtime: the only prerequisite; build the image with `sudo docker
    build --tag virmine:latest virmine` and run with a volume mount so inputs and
    outputs pass through virmineDockerOutputFolder'
  repo: https://hub.docker.com/repository/docker/genevievej16/virmine
- name: virMine 2.0
  role: The pipeline executed inside the container; orchestrates read QC, assembly,
    coverage computation, gene prediction, and BLAST-based contig scoring
  repo: https://github.com/putonti/virmine
- name: SPAdes 3.15.3
  role: One of three assemblers selectable via -a inside the containerized run
- name: MEGAHIT
  role: One of three assemblers selectable via -a inside the containerized run
- name: BBMap 38.94
  role: Computes coverage statistics for the coverage filter during the containerized
    pipeline run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:a8ca718bde1fc0d6cd45ecdc2f559bc7f1c5cc723fa730f27731688ea49abc2f
schema_version: 0.2.0
metadata:
  license_tier: open
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# dockerized-bioinformatics-pipeline-execution

## Summary

Run a complete multi-dependency bioinformatics pipeline (here, virMine 2.0 — read QC, assembly with SPAdes/metaSPAdes/MEGAHIT, coverage computation with BBMap 38.94, gene prediction, and BLAST annotation) inside a Docker container so that all dependencies beyond Docker itself are handled by the Dockerfile.

## When to use

Use when a tool's README states Docker is the only prerequisite and all other dependencies (e.g., Python 3.9, BBMap, SPAdes, MEGAHIT) are handled by the Dockerfile, and you need to execute the full pipeline end-to-end.

## When NOT to use

- Input is already assembled contigs or long reads — use the -A input mode instead of the short-read assembly path
- The user cannot allocate sufficient memory to Docker resources, since the README notes pipeline sections will error when Docker memory is insufficient
- The target pipeline is not distributed as a Docker image or Dockerfile, so containerized execution is not actually available

## Inputs

- paired-end FASTQ reads (e.g., testFiles/test_data_R1.fastq, test_data_R2.fastq)
- viral protein database FASTA (viral_aa.fasta)
- nonviral protein database FASTA (nonviral_aa.fasta)
- Docker image built from the repo Dockerfile or pulled from Docker Hub

## Outputs

- final_contigs.fasta (assembled contigs)
- viral_contigs.fasta (contigs classified as viral)
- predicted ORF files (all ORFs and viral ORFs only)
- BLAST output files for annotated ORFs

## How to apply

Clone the repository (git clone https://github.com/putonti/virmine.git) or pull the prebuilt image (docker pull genevievej16/virmine:latest). Stage paired-end FASTQ inputs plus the viral (viral_aa.fasta) and nonviral (nonviral_aa.fasta) protein databases into the Docker-accessible folder — for the Docker Hub image these live in virmineDockerOutputFolder rather than inputFiles — then build or run the image with a volume bind (-v ~/pathToLocalFolder/virmine:/virmineDockerOutputFolder -i -t). Invoke the pipeline script (python3 virMine.py) with required flags: assembler choice (-a spades/metaspades/megahit/all3), paired reads (-p), databases (-v, -nv), and output (-o). Check Docker resource allocation if any pipeline section errors or fails to run, since memory exhaustion is the documented failure mode.

## Related tools

- **Docker** (Container runtime: the only prerequisite; build the image with `sudo docker build --tag virmine:latest virmine` and run with a volume mount so inputs and outputs pass through virmineDockerOutputFolder) — https://hub.docker.com/repository/docker/genevievej16/virmine
- **virMine 2.0** (The pipeline executed inside the container; orchestrates read QC, assembly, coverage computation, gene prediction, and BLAST-based contig scoring) — https://github.com/putonti/virmine
- **SPAdes 3.15.3** (One of three assemblers selectable via -a inside the containerized run)
- **MEGAHIT** (One of three assemblers selectable via -a inside the containerized run)
- **BBMap 38.94** (Computes coverage statistics for the coverage filter during the containerized pipeline run)

## Examples

```
python3 virMine.py -a all3 -p testFiles/test_data_R1.fastq testFiles/test_data_R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/testOutput
```

## Evaluation signals

- With test data (artificial paired-end reads of E. coli K12 NC_000913.3 and phage Lambda J02459.1) run with -a all3, final_contigs.fasta contains exactly 3 contigs
- viral_contigs.fasta contains exactly 1 contig classified as viral, matching the documented example output
- All expected side-output files are present: predicted ORFs, viral ORFs only, and BLAST output files
- Pipeline completes without errors; if a section errors, check that Docker has enough memory in its resources
- Output files appear in the volume-bound virmineDockerOutputFolder path passed to docker run

## Limitations

- Docker memory limits cause pipeline sections to error or fail to run; resource allocation must be increased
- Inputs must be staged into the correct folder depending on the installation route: inputFiles when building from the cloned repo, virmineDockerOutputFolder when using the Docker Hub image
- The output-count expectation (3 contigs, 1 viral) is specific to the bundled test data and will not generalize to other inputs

## Evidence

- [methods] Docker is the only prerequisite for this program to run, all other dependencies are handled by the Dockerfile.: "Docker is the only prerequisite for this program to run, all other dependencies are handled by the Dockerfile."
- [readme] sudo docker run -v ~/pathToLocalFolder/virmine:/virmineDockerOutputFolder -i -t genevievej16/virmine: "sudo docker run -v ~/pathToLocalFolder/virmine:/virmineDockerOutputFolder -i -t genevievej16/virmine"
- [readme] If any section of the program causes an error or is unable to run, check that you have enough memory in your Docker resources.: "If any section of the program causes an error or is unable to run, check that you have enough memory in your Docker resources."
- [methods] Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral: "Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral"
