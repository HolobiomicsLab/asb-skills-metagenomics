---
name: reference-database-preparation-for-viral-identification
description: Reference for preparing the required -v (viral) and -nv (nonviral) protein
  FASTA databases before running virMine 2.0, when the inputFiles folder of the cloned
  repository lacks viral_aa.fasta and nonviral_aa.fasta.
when_to_use_negative:
- viral_aa.fasta and nonviral_aa.fasta already exist in the inputFiles folder and
  validate — re-downloading is unnecessary
- input consists of already-assembled contigs or reads and the goal is running the
  virMine analysis itself rather than building its reference databases
- the study needs custom databases (e.g. organism-specific reference genomes) not
  covered by the 2020 COG release or RefSeq viral collection
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: virMine 2.0
  role: provides the database-generation script (virmine_make_dbs.py) that produces
    viral_aa.fasta and nonviral_aa.fasta for the -v and -nv arguments
  repo: https://github.com/putonti/virmine
- name: Docker
  role: container environment in which the database generation runs; all other dependencies
    are handled by the Dockerfile
- name: Python 3.9
  role: language of the database-generation script and of virMine.py which consumes
    the outputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:10133b0122124cdb12f69bea67c4d1123842997877b82703e81919b2a516cc47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/putonti/virmine
  tools:
  - Docker
  - Python
  - Python 3.9
  - virMine 2.0
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reference-database-preparation-for-viral-identification

## Summary

Reconstruct the virMine 2.0 database-generation module (virmine_make_dbs.py) that builds the two protein FASTA reference databases required by virMine.py: a nonviral (COG-derived, viral sequences removed) and a viral (RefSeq viral/phage) database. These databases drive virMine's contig scoring, which classifies genes as cellular, viral, or unknown origin.

## When to use

Reference for preparing the required -v (viral) and -nv (nonviral) protein FASTA databases before running virMine 2.0, when the inputFiles folder of the cloned repository lacks viral_aa.fasta and nonviral_aa.fasta.

## When NOT to use

- viral_aa.fasta and nonviral_aa.fasta already exist in the inputFiles folder and validate — re-downloading is unnecessary
- input consists of already-assembled contigs or reads and the goal is running the virMine analysis itself rather than building its reference databases
- the study needs custom databases (e.g. organism-specific reference genomes) not covered by the 2020 COG release or RefSeq viral collection

## Inputs

- RefSeq eukaryotic viral and phage protein sequences (all.faa.tar.gz from ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/)
- bacterial COG database, 2020 release (protein FASTA with origin annotations including viral category X)

## Outputs

- nonviral_aa.fasta (COG-derived cellular bacterial proteins, category X removed)
- viral_aa.fasta (RefSeq eukaryotic viral and phage proteins)

## How to apply

Run the database-generation script in the virMine Docker environment (Docker is the only prerequisite; the Dockerfile handles all dependencies). The script first downloads the latest collection of RefSeq eukaryotic viral and phage protein sequences from the NCBI Genomes FTP (all.faa.tar.gz at ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/all.faa.tar.gz) and extracts the protein FASTA to form the viral database. It then retrieves the latest bacterial Clusters of Orthologous Genes (COG) database, 2020 release, which contains bacterial protein sequences with origin annotations, and filters out all sequences annotated as viral/phage origin (category X), retaining only cellular bacterial proteins. Write the two sets as protein FASTA files named nonviral_aa.fasta (COG-derived) and viral_aa.fasta (RefSeq-derived) and place them in the inputFiles folder of the cloned virMine repository (or the Docker output folder when using the Docker Hub image). These files become the required -v and -nv arguments of virMine.py, so they must exist, be non-empty, and parse as valid protein FASTA.

## Related tools

- **virMine 2.0** (provides the database-generation script (virmine_make_dbs.py) that produces viral_aa.fasta and nonviral_aa.fasta for the -v and -nv arguments) — https://github.com/putonti/virmine
- **Docker** (container environment in which the database generation runs; all other dependencies are handled by the Dockerfile)
- **Python 3.9** (language of the database-generation script and of virMine.py which consumes the outputs)

## Evaluation signals

- both nonviral_aa.fasta and viral_aa.fasta exist in the inputFiles folder and are non-empty
- both files parse as valid protein FASTA (headers plus amino-acid sequences)
- nonviral_aa.fasta contains no sequences annotated as COG category X (viral/phage origin)
- the files are accepted by virMine.py as the -v and -nv arguments without argument errors
- a test run (E. coli K12 + phage Lambda reads) classifies 1 of the 3 assembled contigs as viral, confirming the databases function in scoring

## Limitations

- databases reflect a snapshot: the 2020 COG release and the latest NCBI RefSeq viral collection at download time; newer releases require re-running the script
- the COG filter removes category X (viral/phage) sequences, so cellular-annotation errors or misannotated viral genes in COG could propagate into the nonviral reference
- the databases must be in place before building the Docker image (Option 1) or must be placed in the designated Docker output folder (Option 2)
- RefSeq viral database covers known eukaryotic viruses and phages; virMine still relies on this relatedness signal, so divergent/novel viruses have limited reference coverage

## Evidence

- [abstract] generates a viral database from the latest collection of RefSeq eukaryotic viral and phage genomes (ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/all.faa.tar.gz): "generates a viral database from the latest collection of RefSeq eukaryotic viral and phage genomes (ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/all.faa.tar.gz)"
- [abstract] The script retrieves the latest bacterial Clusters of Orthologous Genes (COG) database, released in 2020: "The script retrieves the latest bacterial Clusters of Orthologous Genes (COG) database, released in 2020 (6)"
- [readme] Move paired-end fastq files, as well as the viral and nonviral databases, to the inputFiles folder prior to building the docker image.: "Move paired-end fastq files, as well as the viral and nonviral databases, to the inputFiles folder prior to building the docker image."
- [methods] python3 virMine.py ... -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta: "python3 virMine.py -a spades -p inputFiles/R1.fastq inputFiles/R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta"
