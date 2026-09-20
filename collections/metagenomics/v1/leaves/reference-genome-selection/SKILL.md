---
name: reference-genome-selection
description: Use when building a reduced, non-redundant species-level reference gene
  database (pan-genomes) from thousands of complete genomes to support gene prediction
  model selection on metagenomic contigs.
when_to_use_negative:
- Input is already a non-redundant gene/pan-genome database rather than a genome collection
  needing redundancy filtering
- The target taxa lack complete genome representations in public repositories, so
  species-specific parameters cannot be estimated
- The goal is de novo gene prediction without a similarity-based model selection step
  (contigs not assignable to a pan-genome should instead fall back to MetaGeneMark
  or GeneMarkS-2 based on contig length)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NCBI BLAST (blastn/MegaBLAST)
  role: 'Similarity screen enforcing the non-redundancy filter: a query gene is admitted
    only if no pan-genome target reaches ANI >95% with >90% query coverage'
- name: makeblastdb
  role: Formats the concatenated pan-genome FASTA into the searchable 'pan' BLAST
    database used later for contig-to-pan-genome assignment
- name: GeneMark-HM
  role: Pipeline whose gene prediction and model-selection steps consume the pan-genome
    database and species-specific models built here
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMarkS-2
  role: Self-training algorithm used to derive species-specific parameters from the
    selected complete genomes
  repo: https://github.com/GeneProbe/GeneMark-HM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:0f612d443f915cca2ff40685d363baef0058e356d16eff373c7c1ffc2c422ea4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GeneProbe/GeneMark-HM
  repo_url: https://github.com/GeneProbe/GeneMark-HM
  tools:
  - GeneMark-HM
  - GeneMarkS-2
  - NCBI BLAST (blastn/MegaBLAST)
  - makeblastdb
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# reference-genome-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects a non-redundant, species-level pan-genome reference set from >310,000 prokaryotic rGenomes by seeding each pan-genome with its representative genome's genes and admitting additional genes only when no MegaBLAST/blastn target reaches ANI >95% with >90% query coverage, capped at the 150 longest rGenomes per SGB. This yields a reduced reference database (~20 Gb pan.fasta) for model selection in GeneMark-HM.

## When to use

Use when building a reduced, non-redundant species-level reference gene database (pan-genomes) from thousands of complete genomes to support gene prediction model selection on metagenomic contigs.

## When NOT to use

- Input is already a non-redundant gene/pan-genome database rather than a genome collection needing redundancy filtering
- The target taxa lack complete genome representations in public repositories, so species-specific parameters cannot be estimated
- The goal is de novo gene prediction without a similarity-based model selection step (contigs not assignable to a pan-genome should instead fall back to MetaGeneMark or GeneMarkS-2 based on contig length)

## Inputs

- Complete or near-complete prokaryotic reference genomes (rGenomes) from GenBank (~310,000 genomes, ~1 Tb nt)
- Predicted protein-coding gene nucleotide sequences from each rGenome
- Species-level genome clusters (SGB/tCluster) with designated representative genomes

## Outputs

- Species-level pan-genome gene sequences for 5082 species (pan.fasta, ~20 Gb nucleotide FASTA)
- BLAST-searchable database 'pan' built with makeblastdb (-parse_seqids -dbtype nucl -out pan -title pan)
- Database of species-specific gene finding model files (refmod)

## How to apply

First predict protein-coding genes in every reference genome (rGenome) to build the query gene space. Seed each species pan-genome with the gene set of its representative genome, then for each additional query gene run an NCBI blastn/MegaBLAST screen against the accumulated pan-genome genes and admit the gene only if its best target fails the non-redundancy threshold of ANI >95% with >90% query coverage. Restrict the pool of contributing genomes to the 150 longest rGenomes per SGB/tCluster to cap computational cost. Concatenate accepted sequences for all 5082 species into the ~20 Gb pan.fasta and format it with makeblastdb ('-parse_seqids -dbtype nucl') so that initially predicted contig genes can be searched to select species-specific GeneMark.hmm-2 models. The rationale is that species-specific parameters outperform general metagenomic parameters, but they can only be estimated from complete genomes of close relatives available in public repositories.

## Related tools

- **NCBI BLAST (blastn/MegaBLAST)** (Similarity screen enforcing the non-redundancy filter: a query gene is admitted only if no pan-genome target reaches ANI >95% with >90% query coverage)
- **makeblastdb** (Formats the concatenated pan-genome FASTA into the searchable 'pan' BLAST database used later for contig-to-pan-genome assignment)
- **GeneMark-HM** (Pipeline whose gene prediction and model-selection steps consume the pan-genome database and species-specific models built here) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMarkS-2** (Self-training algorithm used to derive species-specific parameters from the selected complete genomes) — https://github.com/GeneProbe/GeneMark-HM

## Examples

```
makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan
```

## Evaluation signals

- pan.fasta exists, parses as valid nucleotide FASTA, and totals ~20 Gb in size
- The 'pan' BLAST database is queryable via blastn after makeblastdb formatting
- The pan-genome set covers 5082 species-level references (more than five thousand taxa known in human gut metagenomes)
- No admitted query gene has a best target with ANI >95% and >90% query coverage (non-redundancy invariant)
- Per-species contributor cap respected: only the 150 longest rGenomes per SGB/tCluster were used

## Limitations

- GeneMark-HM accuracy depends on correct assignment of a metagenomic contig to a reference pan-genome; wrong assignments degrade predictions
- Coverage is uneven across body sites: pan-genome assignment reached 99.4% for human gut metagenomes but only 89.0% (oral) and 79.4% (skin), so taxa missing from the reference are processed by fallback algorithms
- The ~20 Gb pan.fasta download from Amazon S3 can take considerable time

## Evidence

- [readme] Species-specific parameters are impossible to estimate from short contigs. For many species present in human gut microbiome, complete or almost genomes are available in public repositories.: [evidence span withheld — anchor status: external_doc]
- [readme] More than five thousand taxa selected for the reference database: [evidence span withheld — anchor status: external_doc]
