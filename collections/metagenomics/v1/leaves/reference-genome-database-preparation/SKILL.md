---
name: reference-genome-database-preparation
description: Use when a pipeline step requires a pre-built reference index (e.g.,
  Bowtie 2 index of hs37d5 for host read removal) and the reference FASTA/index must
  be downloaded and prepared before annotation can start.
when_to_use_negative:
- You already have a prepared host reference index or use a non-human host database
  you have built yourself
- You only need to run steps downstream of host filtering (e.g., starting from an
  already quality-controlled FASTQ via a different starting step)
- Disk space, RAM (~40GB), or runtime (several hours) budgets cannot accommodate the
  full database preparation
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaLAFFA
  role: Pipeline whose prepare_databases.py script downloads and prepares the hs37d5
    decoy reference and other default databases
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Conda
  role: Package manager used to install MetaLAFFA and all software dependencies, and
    whose environment prefix hosts the prepared databases
- name: Bowtie 2
  role: Aligner for which the hs37d5-plus-decoy reference index is prepared, enabling
    host read filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:a2321c8db1f766ab8502bbbe1b608c5394c2b3a097bcdd019d0db1f597cf031a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - Bowtie 2
  - Conda
  - MetaLAFFA
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# reference-genome-database-preparation

## Summary

Prepares reference sequence databases (e.g., the hs37d5 human genome plus 1000 Genomes Project decoy sequences as a Bowtie 2 index) so downstream pipeline steps like host read filtering can run against correctly formatted indexes, using MetaLAFFA's prepare_databases.py script.

## When to use

Use when a pipeline step requires a pre-built reference index (e.g., Bowtie 2 index of hs37d5 for host read removal) and the reference FASTA/index must be downloaded and prepared before annotation can start.

## When NOT to use

- You already have a prepared host reference index or use a non-human host database you have built yourself
- You only need to run steps downstream of host filtering (e.g., starting from an already quality-controlled FASTQ via a different starting step)
- Disk space, RAM (~40GB), or runtime (several hours) budgets cannot accommodate the full database preparation

## Inputs

- Reference database selections as CLI flags to prepare_databases.py (-hr, -km, -u)
- Conda environment with MetaLAFFA dependencies (Conda >= 4.8)

## Outputs

- Bowtie 2 index of hs37d5 human reference plus 1000 Genomes Project decoy sequences
- Prepared UniRef90 database for read mapping and functional annotation
- KEGG ko-to-module and ko-to-pathway mapping files
- Databases installed under $CONDA_PREFIX/MetaLAFFA

## How to apply

Install MetaLAFFA via Conda (conda create -n metalaffa metalaffa -c bioconda -c borenstein-lab), activate the environment, and run prepare_databases.py with flags selecting the databases you need: -hr for the human reference and decoy sequences used in host filtering, -km for KEGG ko-to-module/pathway mappings, -u for UniRef90, and optional -c to clean intermediate files. The -hr flag installs the hs37d5 human reference plus decoy sequences from the 1000 Genomes Project as a Bowtie 2 index so host reads can be aligned and removed during quality control. Databases are installed into $CONDA_PREFIX/MetaLAFFA. Plan for the run's resource cost (several hours, ~40GB RAM, ~300GB disk without cleanup, ~150GB with cleanup), and verify the host-filtering step afterward by confirming the host-filtered FASTQ exists and summary statistics report the number of host reads removed.

## Related tools

- **MetaLAFFA** (Pipeline whose prepare_databases.py script downloads and prepares the hs37d5 decoy reference and other default databases) — https://github.com/borenstein-lab/MetaLAFFA
- **Conda** (Package manager used to install MetaLAFFA and all software dependencies, and whose environment prefix hosts the prepared databases)
- **Bowtie 2** (Aligner for which the hs37d5-plus-decoy reference index is prepared, enabling host read filtering)

## Examples

```
conda create -n metalaffa metalaffa -c bioconda -c borenstein-lab; conda activate metalaffa; prepare_databases.py -hr -km -u
```

## Evaluation signals

- The hs37d5 Bowtie 2 index files exist under $CONDA_PREFIX/MetaLAFFA after prepare_databases.py -hr completes
- The subsequent host filtering step produces a host-filtered FASTQ for the input sample
- The per-step summary statistics table reports the number of host reads removed (e.g., for HMP stool sample SRS017307)
- Database preparation used the documented flags (-hr, -km, -u) rather than ad hoc downloads, matching MetaLAFFA's file_organization.py paths

## Limitations

- Database preparation is time and resource intensive: several hours, ~40GB of RAM, ~300GB free disk space without cleaning intermediate files, ~150GB with -c cleanup
- If the preparation script is interrupted without the -c cleanup option, you may have to restart the entire database download and preparation process from the beginning
- The default -hr database is human-specific (hs37d5 plus 1000 Genomes decoys); other host organisms require substituting an alternative host database

## Evidence

- [readme] Default reference databases can be downloaded and prepared for MetaLAFFA using the `prepare_databases.py` script.: "Default reference databases can be downloaded and prepared for MetaLAFFA using the `prepare_databases.py` script."
- [readme] -hr: Download and prepare the database of human reference and decoy sequences (used in the 1000 genomes project) for host filtering.: "-hr: Download and prepare the database of human reference and decoy sequences (used in the 1000 genomes project) for host filtering."
- [intro] MetaLAFFA removes host reads by mapping reads to the hs37d5 human genome reference... using Bowtie 2 [12] and then discarding any reads identified as human: [evidence span withheld — anchor status: not_found]
- [readme] This process can be time and resource intensive, taking several hours, ~40GB of RAM, and ~300GB of free disk space if not cleaning up intermediate files: "This process can be time and resource intensive, taking several hours, ~40GB of RAM, and ~300GB of free disk space if not cleaning up intermediate files"
- [intro] mapping reads to the hs37d5 human genome reference with decoy sequences available from the 1000 genomes project: "mapping reads to the hs37d5 human genome reference with decoy sequences available from the 1000 genomes project"
