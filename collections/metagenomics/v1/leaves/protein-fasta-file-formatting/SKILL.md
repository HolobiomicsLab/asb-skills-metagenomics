---
name: protein-fasta-file-formatting
description: Use when preparing the -v (viral) and -nv (nonviral) protein FASTA databases
  required as input to virMine.py, or when rebuilding virMine 2.0 reference databases
  after a COG or RefSeq release update.
when_to_use_negative:
- The input is nucleotide (e.g. genomic FASTQ or contig) data rather than protein
  sequences — these databases must be amino-acid FASTA for BLAST-based contig scoring.
- The user only intends to run virMine on supplied long-read/contig assemblies with
  pre-existing viral and nonviral databases; rebuilding the databases is unnecessary.
- A de novo viral discovery context that deliberately avoids any reference database;
  this skill builds virMine-specific reference inputs, not a general reference preparation
  tool.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3174
tools:
- name: virMine 2.0
  role: Database-generation module (virmine_make_dbs.py) downloads, filters, and formats
    the two protein FASTA databases consumed by virMine.py
  repo: https://github.com/putonti/virmine
- name: Docker
  role: Runs the virMine environment in which the database-generation script executes;
    Docker is the only prerequisite, other dependencies handled by the Dockerfile
- name: Python 3.9
  role: Language of the virMine 2.0 database-generation script that filters COG category
    X sequences and writes the FASTA files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:4c1a86356d2200f25c19d6225d88160e5853126212b1557e258e36eac7f4bd82
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
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# protein-fasta-file-formatting

## Summary

Format virMine 2.0's reference protein databases: derive a nonviral bacterial database from the 2020 COG release (dropping category X viral/phage sequences) and a viral database from RefSeq viral/phage proteins, writing both as valid protein FASTA files consumed by virMine.py.

## When to use

Use when preparing the -v (viral) and -nv (nonviral) protein FASTA databases required as input to virMine.py, or when rebuilding virMine 2.0 reference databases after a COG or RefSeq release update.

## When NOT to use

- The input is nucleotide (e.g. genomic FASTQ or contig) data rather than protein sequences — these databases must be amino-acid FASTA for BLAST-based contig scoring.
- The user only intends to run virMine on supplied long-read/contig assemblies with pre-existing viral and nonviral databases; rebuilding the databases is unnecessary.
- A de novo viral discovery context that deliberately avoids any reference database; this skill builds virMine-specific reference inputs, not a general reference preparation tool.

## Inputs

- RefSeq eukaryotic viral and phage protein collection (all.faa.tar.gz from ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/)
- Bacterial COG database, 2020 release, protein sequences with origin annotations (including category X viral/phage entries to be removed)

## Outputs

- nonviral_aa.fasta — COG-derived bacterial protein FASTA with category X (viral/phage) sequences removed, placed in inputFiles/
- viral_aa.fasta — RefSeq-derived viral/phage protein FASTA, placed in inputFiles/

## How to apply

Run the virMine database-generation module (virmine_make_dbs.py) via the virMine 2.0 Docker/Python setup. Step 1: download the latest RefSeq eukaryotic viral and phage protein collection (all.faa.tar.gz from the NCBI Genomes FTP, ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/) and extract the protein FASTA. Step 2: retrieve the 2020 release of the bacterial Clusters of Orthologous Genes (COG) database, whose protein sequences carry origin annotations. Step 3: filter out every COG sequence annotated as viral/phage origin (category X) so only cellular bacterial proteins remain — this exclusion step is what lets virMine score contigs by gene origin rather than homology alone. Step 4: write the two sets as protein FASTA named nonviral_aa.fasta (COG-derived) and viral_aa.fasta (RefSeq-derived) into the inputFiles folder of the virMine repository. Verify both files are non-empty, parse as protein FASTA, and are accepted as the -v and -nv arguments of virMine.py.

## Related tools

- **virMine 2.0** (Database-generation module (virmine_make_dbs.py) downloads, filters, and formats the two protein FASTA databases consumed by virMine.py) — https://github.com/putonti/virmine
- **Docker** (Runs the virMine environment in which the database-generation script executes; Docker is the only prerequisite, other dependencies handled by the Dockerfile)
- **Python 3.9** (Language of the virMine 2.0 database-generation script that filters COG category X sequences and writes the FASTA files)

## Examples

```
python3 virMine.py -a spades -p inputFiles/R1.fastq inputFiles/R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/output
```

## Evaluation signals

- Both inputFiles/nonviral_aa.fasta and inputFiles/viral_aa.fasta exist and are non-empty.
- Each output parses as valid protein FASTA (amino-acid alphabet, consistent headers).
- The nonviral FASTA contains no COG category X (viral/phage origin) sequences; the viral FASTA contains only RefSeq viral/phage proteins.
- virMine.py accepts the files as -v (viral database) and -nv (nonviral database) arguments without error.
- Running the README test data with all 3 assembly methods yields 3 contigs (final_contigs.fasta), 1 classified as viral (viral_contigs.fasta), confirming the databases function in scoring.

## Limitations

- The COG database is versioned (2020 release at time of publication); databases must be regenerated when NCBI/COG releases update to stay current.
- The exclusion approach depends on COG origin annotations — sequences lacking reliable category assignments may be misclassified between the viral and nonviral databases.
- If any section of the program errors, check that Docker has enough memory resources; Docker is the only prerequisite but resource limits can fail the run.

## Evidence

- [abstract] generates a viral database from the latest collection of RefSeq eukaryotic viral and phage genomes (ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/all.faa.tar.gz): "generates a viral database from the latest collection of RefSeq eukaryotic viral and phage genomes (ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/all.faa.tar.gz)"
- [abstract] COG 2020 retrieval and category X removal: "The script retrieves the latest bacterial Clusters of Orthologous Genes (COG) database, released in 2020"
- [methods] virMine.py CLI accepting the formatted FASTA files: "python3 virMine.py -a spades -p inputFiles/R1.fastq inputFiles/R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta"
- [readme] README: databases must be moved to inputFiles before docker build: "Move paired-end fastq files, as well as the viral and nonviral databases, to the inputFiles folder prior to building the docker image."
- [abstract] Exclusion scoring rationale: "virMine 2.0 uses an exclusion technique to remove nonviral data from sequencing reads and scores the remaining data based on relatedness to viral elements"
