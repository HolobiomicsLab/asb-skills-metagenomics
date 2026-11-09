---
name: cog-database-category-filtering
description: Use when preparing the -nv (nonviral) and -v (viral) protein FASTA inputs
  for virMine 2.0, or when a bacterial-only protein reference with viral-origin sequences
  excluded is needed.
when_to_use_negative:
- When running virMine on long-read or pre-assembled contigs without needing to rebuild
  the reference databases — reuse the existing viral_aa.fasta and nonviral_aa.fasta
  instead
- When a nucleotide-level or RNA viral reference database is required — this skill
  produces protein FASTA only, from RefSeq eukaryotic viral/phage proteins and bacterial
  COG proteins
edam_operation: http://edamontology.org/operation_0256
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: virMine 2.0
  role: Pipeline whose database-generation script (virmine_make_dbs.py) downloads
    and filters the COG and RefSeq viral databases; outputs are consumed as -v and
    -nv arguments of virMine.py
  repo: https://github.com/putonti/virmine
- name: Docker
  role: Container runtime in which the virMine 2.0 database-generation script is executed;
    the only prerequisite, with dependencies handled by the Dockerfile
- name: Python 3.9
  role: Language of the virMine 2.0 database-generation module that retrieves the
    COG database and removes viral-origin (category X) sequences
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:dcfd536a4536ffa2af39b913d45f81c5423530766100ec731926fe1503921b5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/putonti/virmine
  tools:
  - Docker
  - Python
  - Python 3.9
  - virMine 2.0
  edam_operation: http://edamontology.org/operation_0256
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cog-database-category-filtering

## Summary

Constructs the nonviral (bacterial) reference database used by virMine 2.0 by retrieving the 2020-release bacterial COG database and removing all sequences annotated as viral/phage origin (category X), yielding nonviral_aa.fasta; pairs with a RefSeq viral database (viral_aa.fasta) for contig origin scoring.

## When to use

Use when preparing the -nv (nonviral) and -v (viral) protein FASTA inputs for virMine 2.0, or when a bacterial-only protein reference with viral-origin sequences excluded is needed.

## When NOT to use

- When running virMine on long-read or pre-assembled contigs without needing to rebuild the reference databases — reuse the existing viral_aa.fasta and nonviral_aa.fasta instead
- When a nucleotide-level or RNA viral reference database is required — this skill produces protein FASTA only, from RefSeq eukaryotic viral/phage proteins and bacterial COG proteins

## Inputs

- Bacterial COG database, 2020 release, with origin annotations (protein sequences)
- RefSeq eukaryotic viral and phage protein sequences from NCBI (all.faa.tar.gz, NCBI Genomes FTP)

## Outputs

- inputFiles/nonviral_aa.fasta — COG-derived bacterial protein FASTA with category X (viral/phage origin) sequences removed
- inputFiles/viral_aa.fasta — RefSeq-derived viral/phage protein FASTA

## How to apply

Download the latest collection of RefSeq eukaryotic viral and phage protein sequences from NCBI (all.faa.tar.gz on the NCBI Genomes FTP) and extract the protein FASTA. Retrieve the bacterial Clusters of Orthologous Genes (COG) database, 2020 release, which contains bacterial protein sequences with origin annotations. Filter the COG database to remove all sequences annotated as viral/phage origin (category X), retaining only cellular bacterial proteins. Write the two sets as protein FASTA files named nonviral_aa.fasta (COG-derived) and viral_aa.fasta (RefSeq-derived) and place them in the inputFiles folder of the virMine repository layout so they can be passed as the -v and -nv arguments of virMine.py.

## Related tools

- **virMine 2.0** (Pipeline whose database-generation script (virmine_make_dbs.py) downloads and filters the COG and RefSeq viral databases; outputs are consumed as -v and -nv arguments of virMine.py) — https://github.com/putonti/virmine
- **Docker** (Container runtime in which the virMine 2.0 database-generation script is executed; the only prerequisite, with dependencies handled by the Dockerfile)
- **Python 3.9** (Language of the virMine 2.0 database-generation module that retrieves the COG database and removes viral-origin (category X) sequences)

## Examples

```
python3 virMine.py -a spades -p inputFiles/R1.fastq inputFiles/R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/output
```

## Evaluation signals

- Both FASTA files (nonviral_aa.fasta and viral_aa.fasta) exist and are non-empty
- Both files parse as valid protein FASTA
- nonviral_aa.fasta contains no sequences annotated as viral/phage origin (category X) from the 2020 COG release
- The files are accepted as the -v (viral database) and -nv (nonviral database) arguments of virMine.py and placed in the inputFiles folder of the virMine repository layout

## Limitations

- COG bacterial database is the 2020 release; newly sequenced bacteria or recently added viral proteins in RefSeq will be absent until the databases are regenerated
- Docker memory limits can cause errors — the README notes to check Docker resources if any section of the program fails to run
- Exclusion is homology-based against these reference databases; viral sequences with no similarity to RefSeq viral proteins may escape the viral database and novel bacterial-like viral genes may be mis-scored

## Evidence

- [abstract] The script retrieves the latest bacterial Clusters of Orthologous Genes (COG) database, released in 2020: "The script retrieves the latest bacterial Clusters of Orthologous Genes (COG) database, released in 2020"
- [abstract] generates a viral database from the latest collection of RefSeq eukaryotic viral and phage genomes (ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/all.faa.tar.gz): "generates a viral database from the latest collection of RefSeq eukaryotic viral and phage genomes (ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/all.faa.tar.gz)"
- [readme] Move paired-end fastq files, as well as the viral and nonviral databases, to the inputFiles folder prior to building the docker image.: "Move paired-end fastq files, as well as the viral and nonviral databases, to the inputFiles folder prior to building the docker image."
- [methods] python3 virMine.py -a spades -p inputFiles/R1.fastq inputFiles/R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta: "python3 virMine.py -a spades -p inputFiles/R1.fastq inputFiles/R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta"
