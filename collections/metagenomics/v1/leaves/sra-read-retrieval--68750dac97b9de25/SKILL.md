---
name: sra-read-retrieval
description: Use when a task provides an SRA accession (e.g. SRR9875293, SRR3458562)
  or BioProject (PRJNA431646, PRJNA319556, PRJNA395784) and local FASTQ input is needed
  for downstream VirMAP or metagenomic classification.
when_to_use_negative:
- Reads are already available locally as FASTQ files, so no SRA download step is needed
- The task is downstream analysis (assembly, classification, comparison) on existing
  reads rather than data acquisition
- The dataset is not deposited in the Sequence Read Archive and no accession can be
  resolved
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: fasterq-dump
  role: SRA Toolkit utility invoked to pull raw paired-end reads from the Sequence
    Read Archive into FASTQ files for VirMAP input
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:0ecfbabfe43827315290f75496d274b18267d525f2e37771393d32b574e818c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBMap
  - DIAMOND
  - MEGAHIT
  - Perl
  - Statistics::Basic
  - VirMAP
  - fasterq-dump
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# sra-read-retrieval

## Summary

Retrieves raw paired-end sequencing reads from the Sequence Read Archive using fasterq-dump, as done throughout the VirMAP paper for samples like SRR9875293 (Viral Mock Community) and the NetoVir and Brazil datasets. Downloading FASTQ reads into a fast temp/scratch location is the entry step of every VirMAP validation and comparison workflow.

## When to use

Use when a task provides an SRA accession (e.g. SRR9875293, SRR3458562) or BioProject (PRJNA431646, PRJNA319556, PRJNA395784) and local FASTQ input is needed for downstream VirMAP or metagenomic classification.

## When NOT to use

- Reads are already available locally as FASTQ files, so no SRA download step is needed
- The task is downstream analysis (assembly, classification, comparison) on existing reads rather than data acquisition
- The dataset is not deposited in the Sequence Read Archive and no accession can be resolved

## Inputs

- SRA accession string (e.g. SRR9875293, SRR3458562)
- BioProject identifiers used to identify samples (PRJNA431646, PRJNA319556, PRJEB7888, PRJNA395784)

## Outputs

- Paired-end FASTQ files (accession_1.fastq and accession_2.fastq) in the output directory (e.g. /home/$USER/VirmapTest)
- Populated download directory ready as --readF/--readF2 input to Virmap.pl

## How to apply

Run fasterq-dump with the accession as the sole positional argument, pointing the temporary directory at fast storage (-t /dev/shm for in-memory temp) and parallelizing with thread count (-e 4), and setting -O to the output directory that downstream Virmap.pl runs will read from (e.g. /home/$USER/VirmapTest). For large batch jobs, route intermediates to local instance-store scratch (e.g. /scratch/tmp via TMPDIR) rather than the root volume, mirroring the paper's use of local SSD for VirMAP databases and outputs. Each accession yields paired-end FASTQ files (accession_1.fastq and accession_2.fastq) that are passed to Virmap.pl via --readF/--readR for assembly and taxonomic classification.

## Related tools

- **fasterq-dump** (SRA Toolkit utility invoked to pull raw paired-end reads from the Sequence Read Archive into FASTQ files for VirMAP input)

## Examples

```
fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293
```

## Evaluation signals

- The output directory (e.g. /home/$USER/VirmapTest) exists and contains populated accession_1.fastq and accession_2.fastq files (file_exists check)
- Line counts of the FASTQ pair are equal, confirming intact paired-end retrieval
- The run log shows fasterq-dump completed without error and the number of spots matches the SRA accession metadata
- The retrieved FASTQ can be passed to Virmap.pl with --useMegahit --useBbnorm and produces populated taxonomic classification outputs

## Limitations

- Using /dev/shm as the temp directory ties retrieval to available RAM; very large samples may need a scratch location instead (e.g. /scratch/tmp)
- No software version is stated for the SRA Toolkit or fasterq-dump in the paper, so reproducibility depends on pinning your own toolkit version
- Downloads can be slow for large virome samples (e.g. four-sample Brazil PRJNA395784 batch), so parallelization flags and local instance storage matter

## Evidence

- [methods] fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293: "`fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293`"
- [discussion] The raw sequence datasets generated and analyzed during the current study are available through the Sequence Read Archive: "The raw sequence datasets generated and analyzed during the current study are available through the Sequence Read Archive"
- [discussion] Viral Mock Community, BioProject ID PRJNA431646: "Viral Mock Community, BioProject ID PRJNA431646"
