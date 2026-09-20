---
name: genome-database-construction
description: Use when you have FASTA genome files (public references plus MAGs) with
  a table of source associations (e.g., cow, pig, dog, sewage) and need a formatted
  database to run sourceapp.py for metagenomic fecal source tracking.
when_to_use_negative:
- Input is raw metagenomic reads rather than assembled genomes/MAGs — reads must first
  be assembled and binned into MAGs before database construction
- The prebuilt default SourceApp database (available on Zenodo, doi:10.5281/zenodo.12795076,
  browsable on MiGA at disc-genomics.uibk.ac.at) already covers your sources and supplementary
  genomes are not needed
- The goal is running source attribution on water metagenomes — that is the sourceapp.py
  analysis step, which consumes the database this skill produces
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SourceApp
  role: Provides sourceapp_build.py, the script that orchestrates quality filtering,
    dereplication, and database formatting
  repo: https://github.com/bglindner/SourceApp
- name: CheckM2
  role: Computes genome completeness and redundancy from the .dmnd database to apply
    the aggregate quality threshold of 50
- name: dRep
  role: Dereplicates genomes within and across source categories at the 95% ANI clustering
    threshold (S_algorithm, fastANI-based)
- name: fastANI
  role: Computes ANI values used for dereplication and cross-reactive genome flagging
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:fc1cc8b9aecebced097e0a52ce845cb9ef58f0e431bc9deb4d593977dfab5d3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CheckM2
  - Python
  - SourceApp
  - dRep
  - fastANI
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# genome-database-construction

## Summary

Builds a de novo SourceApp source-associated genome database by quality-filtering (CheckM2, aggregate quality ≥ 50), dereplicating (95% ANI), and cross-reactivity-flagging input reference genomes and MAGs assigned to fecal source categories, enabling metagenomic fecal source tracking without sequencing the sources.

## When to use

Use when you have FASTA genome files (public references plus MAGs) with a table of source associations (e.g., cow, pig, dog, sewage) and need a formatted database to run sourceapp.py for metagenomic fecal source tracking.

## When NOT to use

- Input is raw metagenomic reads rather than assembled genomes/MAGs — reads must first be assembled and binned into MAGs before database construction
- The prebuilt default SourceApp database (available on Zenodo, doi:10.5281/zenodo.12795076, browsable on MiGA at disc-genomics.uibk.ac.at) already covers your sources and supplementary genomes are not needed
- The goal is running source attribution on water metagenomes — that is the sourceapp.py analysis step, which consumes the database this skill produces

## Inputs

- Directory of genome FASTA files (*.fna) — publicly available reference genomes plus MAGs from short- (Illumina NovaSeq 6000) and long-read (Oxford Nanopore minION/Flongle) sequencing of fecal spike-in slurries
- Headerless tab-separated source association table mapping genome file name (col 1) to source category (col 2, e.g., cow, pig, dog, cat, chicken, goat, sewage, septage)
- CheckM2 database file (.dmnd) supplied via -d

## Outputs

- SourceApp-formatted database directory named '<output>_SourceAppdb/' (e.g., 'SourceAppdb') usable via -d in sourceapp.py
- Species-cluster representative genomes retained after 95% ANI dereplication
- Recorded cross-reactive genome pairs across source categories (ANI ≥ 95%)
- Set of excluded genomes below aggregate quality score 50

## How to apply

Collect input genomes as FASTA files (e.g., *.fna) in one directory, combining publicly available reference genomes with MAGs recovered from short- and long-read sequencing of fecal spike-ins (NCBI BioProject PRJNA1092107). Compile a headerless tab-separated source-association table mapping each genome file name (col 1) to its fecal source category (col 2). Run sourceapp_build.py with -i (input genome directory), -o (output path), -s (association TSV), and -d (CheckM2 .dmnd database); CheckM2 computes completeness and redundancy and genomes with aggregate quality (completeness − 5 × redundancy) below 50 are discarded. Dereplicate surviving genomes within/across source categories at 95% ANI using fastANI-based S_algorithm (dRep) to retain species-cluster representatives, and record pairs in different source categories with ANI ≥ 95% as cross-reactive. Validation succeeds only if the output directory '<output>_SourceAppdb/' is created and is well-formed enough to pass via -d to sourceapp.py.

## Related tools

- **SourceApp** (Provides sourceapp_build.py, the script that orchestrates quality filtering, dereplication, and database formatting) — https://github.com/bglindner/SourceApp
- **CheckM2** (Computes genome completeness and redundancy from the .dmnd database to apply the aggregate quality threshold of 50)
- **dRep** (Dereplicates genomes within and across source categories at the 95% ANI clustering threshold (S_algorithm, fastANI-based))
- **fastANI** (Computes ANI values used for dereplication and cross-reactive genome flagging)

## Examples

```
sourceapp_build.py -i genomes_dir -o SourceAppdb -s source_associations.tsv -d checkm2.dmnd
```

## Evaluation signals

- Run creates an output directory ending in '_SourceAppdb/' (e.g., 'SourceAppdb') rather than failing
- All genomes with aggregate quality (completeness − 5 × redundancy) below 50 are absent from the final database
- Within/across source categories, no retained genomes share ≥ 95% ANI except representatives; cross-category pairs at ANI ≥ 95% are flagged as cross-reactive
- The produced directory is accepted as a valid -d argument by sourceapp.py
- Each database genome maps to exactly one source category matching the association TSV

## Limitations

- Cow sources were underestimated in benchmarks due to under-sampling of cow fecal microbiomes, so a database built with sparse cow references will yield false negatives
- Low-quality MAGs (aggregate quality < 50) are silently discarded, which can remove source-relevant organisms
- Genome pairs in different source categories with ANI ≥ 95% are cross-reactive and can confound source attribution

## Evidence

- [methods] genomes with aggregate quality scores below 50% as calculated by quality = completeness −5 × redundancy were discarded: "genomes with aggregate quality scores below 50% as calculated by quality = completeness −5 × redundancy were discarded"
- [methods] All medium and high-quality MAGs were dereplicated (at 95% ANI) within their source category: "All medium and high-quality MAGs were dereplicated (at 95% ANI) within their source category"
- [methods] Genome pairs in separate source categories were flagged as cross-reactive when ANI values greater than or equal to 95% were observed: "Genome pairs in separate source categories were flagged as cross-reactive when ANI values greater than or equal to 95% were observed"
- [readme] De novo databases can be constructed by the user with `sourceapp_build.py` and then supplied in the call to `sourceapp.py` itself in place of the default database.: "De novo databases can be constructed by the user with `sourceapp_build.py` and then supplied in the call to `sourceapp.py` itself in place of the default database."
- [readme] SourceApp works best when the user is able to provide supplementary reference genomes to bolster SourceApp's default genomic reference database.: "SourceApp works best when the user is able to provide supplementary reference genomes to bolster SourceApp's default genomic reference database."
