---
name: viral-protein-database-construction
description: 'Use when preparing inputs for virMine 2.0: you must supply viral and
  nonviral protein FASTA databases (-v and -nv arguments) and need to construct them
  from current COG and NCBI RefSeq releases.'
when_to_use_negative:
- You already have curated viral and nonviral protein FASTA databases accepted by
  virMine.py's -v and -nv parameters
- You are running a viral-discovery tool other than virMine whose reference database
  format and origin-annotation requirements differ
- Your analysis needs nucleotide-level or genome-level reference databases rather
  than protein FASTA
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: virMine 2.0
  role: Consumer of the constructed databases; accepts them as the -v (viral database)
    and -nv (nonviral database) arguments of virMine.py
  repo: https://github.com/putonti/virmine
- name: Docker
  role: Container runtime in which the inputFiles folder containing both FASTA databases
    is staged before building/running the virMine image
- name: Python 3.9
  role: Language of the virMine 2.0 database-generation script that downloads, filters
    (COG category X removal), and formats the databases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:97a27f923befd0f0afa0c97e559f5a5ed9d36ccd2a8ec5012533dee6417097b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/putonti/virmine
  tools:
  - Docker
  - Python
  - Python 3.9
  - virMine 2.0
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# viral-protein-database-construction

## Summary

Builds the paired protein reference databases that virMine 2.0 requires: a nonviral bacterial database derived from the 2020 COG release with viral-origin (category X) sequences removed, and a viral database from RefSeq eukaryotic viral and phage protein sequences. These FASTA files feed virMine's contig scoring by gene-content origin (cellular, viral, or unknown).

## When to use

Use when preparing inputs for virMine 2.0: you must supply viral and nonviral protein FASTA databases (-v and -nv arguments) and need to construct them from current COG and NCBI RefSeq releases.

## When NOT to use

- You already have curated viral and nonviral protein FASTA databases accepted by virMine.py's -v and -nv parameters
- You are running a viral-discovery tool other than virMine whose reference database format and origin-annotation requirements differ
- Your analysis needs nucleotide-level or genome-level reference databases rather than protein FASTA

## Inputs

- NCBI RefSeq eukaryotic viral and phage protein sequences (all.faa.tar.gz from ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/)
- COG bacterial protein database, 2020 release, with origin annotations including category X (viral/phage origin)

## Outputs

- viral_aa.fasta — protein FASTA of RefSeq viral/phage proteins (virMine -v argument)
- nonviral_aa.fasta — protein FASTA of COG bacterial proteins with category X removed (virMine -nv argument), both placed in the inputFiles folder

## How to apply

Download the latest collection of RefSeq eukaryotic viral and phage protein sequences from the NCBI Genomes FTP (all.faa.tar.gz) and extract the protein FASTA; this becomes viral_aa.fasta. Retrieve the bacterial Clusters of Orthologous Genes (COG) database (2020 release), which contains bacterial protein sequences with origin annotations, and filter it to remove all sequences annotated as viral/phage origin (category X), retaining only cellular bacterial proteins; write this as nonviral_aa.fasta. Place both files in the inputFiles folder of the virMine repository (or the mounted Docker output folder if using the Docker Hub image) before building or running the container. The rationale is that virMine 2.0 scores contigs based on their gene content's origin, so the databases must cleanly separate cellular from viral proteins for the exclusion-based scoring to work.

## Related tools

- **virMine 2.0** (Consumer of the constructed databases; accepts them as the -v (viral database) and -nv (nonviral database) arguments of virMine.py) — https://github.com/putonti/virmine
- **Docker** (Container runtime in which the inputFiles folder containing both FASTA databases is staged before building/running the virMine image)
- **Python 3.9** (Language of the virMine 2.0 database-generation script that downloads, filters (COG category X removal), and formats the databases)

## Examples

```
python3 virMine.py -a spades -p inputFiles/R1.fastq inputFiles/R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/output
```

## Evaluation signals

- Both viral_aa.fasta and nonviral_aa.fasta exist in the inputFiles folder and are non-empty
- Both files parse as valid protein FASTA
- nonviral_aa.fasta contains no sequences annotated as viral/phage origin (COG category X)
- virMine.py accepts the two files as its -v and -nv required arguments and a test run completes
- Running the README's test data with all 3 assembly methods yields 3 contigs, with 1 classified as viral in viral_contigs.fasta

## Limitations

- Databases reflect the COG 2020 release and the latest RefSeq viral collection at build time; novel viruses absent from RefSeq will be scored as unknown unless homologous to known elements
- If any section of the program errors inside Docker, check that enough memory is allocated to Docker resources
- virMine does not rely solely on homology to known viruses, but database currency still affects classification of contigs

## Evidence

- [abstract] The virMine 2.0 database generation script retrieves the latest bacterial Clusters of Orthologous Genes (COG) database, released in 2020: "The script retrieves the latest bacterial Clusters of Orthologous Genes (COG) database, released in 2020 (6)"
- [abstract] Viral database from RefSeq viral/phage proteins: "generates a viral database from the latest collection of RefSeq eukaryotic viral and phage genomes (ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/all.faa.tar.gz)"
- [abstract] Contig scoring depends on gene-content origin separation: "Contigs are scored based on their gene content’s origin, i.e., cellular, viral, or unknown."
- [readme] virMine command-line interface requires -v and -nv: "Required parameters:
*	-v : list your viral database
*	-nv : list your nonviral database"
- [methods] Example invocation showing database usage: "python3 virMine.py -a spades -p inputFiles/R1.fastq inputFiles/R2.fastq -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta"
