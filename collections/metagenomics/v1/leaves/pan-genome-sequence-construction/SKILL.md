---
name: pan-genome-sequence-construction
description: Use when a reduced nucleotide reference of species-level gene space is
  needed to assign metagenomic contigs to close relatives or to select species-specific
  gene prediction models, as in the GeneMark-HM pipeline.
when_to_use_negative:
- When the goal is annotating metagenomic contigs themselves rather than building
  the reference database (the downstream GeneMark-HM assignment step consumes this
  database, it does not produce it)
- When you need per-taxon gene finding models directly — those live in the separate
  'refmod' database of model files, not in the pan-genome sequence database
- When studying taxa outside the set of species known to be present in human microbiomes,
  since the database was built from genomes of species present in human gut/oral/skin
  metagenomes
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: GeneMark-HM
  role: Pipeline that consumes the pan-genome database for contig-to-pan-genome assignment
    and species-specific model selection
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: NCBI BLAST (blastn)
  role: Similarity screen admitting a query gene only when no accumulated pan-genome
    target reaches ANI >95% with >90% query coverage
- name: makeblastdb
  role: Formats pan.fasta into the searchable BLAST nucleotide database 'pan' ('-parse_seqids
    -dbtype nucl -out pan -title pan')
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:b38987bdf61ce74bb2b3ee7d79ee1c6a8b65b92b9831096e4ae4aa5b654eb789
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
  - NCBI BLAST (blastn)
  - makeblastdb
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pan-genome-sequence-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a species-level pan-genome sequence database for the human microbiome from reference genomes, seeding each pan-genome with its representative genome's predicted genes and admitting additional genes only when no MegaBLAST target shows ANI >95% with >90% query coverage, yielding 5082 pan-genomes compiled into a ~20 Gb BLAST-searchable nucleotide FASTA.

## When to use

Use when a reduced nucleotide reference of species-level gene space is needed to assign metagenomic contigs to close relatives or to select species-specific gene prediction models, as in the GeneMark-HM pipeline.

## When NOT to use

- When the goal is annotating metagenomic contigs themselves rather than building the reference database (the downstream GeneMark-HM assignment step consumes this database, it does not produce it)
- When you need per-taxon gene finding models directly — those live in the separate 'refmod' database of model files, not in the pan-genome sequence database
- When studying taxa outside the set of species known to be present in human microbiomes, since the database was built from genomes of species present in human gut/oral/skin metagenomes

## Inputs

- Reference genomes (rGenomes) of sequenced isolates and reconstructed prokaryotic genomes, e.g. from GenBank
- Protein-coding gene predictions for each rGenome (query gene space for pan-genome construction)
- Representative genome gene sets used to seed each species pan-genome

## Outputs

- pan.fasta: ~20 Gb nucleotide FASTA containing the 5082 species-level pan-genome sequences
- A BLAST nucleotide database named 'pan' (built with makeblastdb) queryable via blastn
- Species-level pan-genome gene sets satisfying the non-redundancy filter (ANI >95% and >90% query coverage rejected)

## How to apply

First predict protein-coding genes in every reference genome (rGenome) to build the query gene space. Seed each species pan-genome with the gene set of its representative genome, then for each additional query gene run an NCBI blastn similarity screen against the accumulated pan-genome genes, admitting the gene only if no target reaches ANI >95% with >90% query coverage (the non-redundancy filter). Restrict the genomes contributing to each species pan-genome to the 150 longest rGenomes per SGB/tCluster to cap redundancy. Concatenate the accepted sequences for all 5082 species into pan.fasta (~20 Gb nucleotide FASTA) and format it with makeblastdb using '-parse_seqids -dbtype nucl -out pan -title pan' so blastn can search it for contig-to-pan-genome assignment. The rationale is that contigs assigned to a pan-genome can then use pre-computed, species-specific GeneMarkS-2 models instead of generic metagenomic parameters.

## Related tools

- **GeneMark-HM** (Pipeline that consumes the pan-genome database for contig-to-pan-genome assignment and species-specific model selection) — https://github.com/GeneProbe/GeneMark-HM
- **NCBI BLAST (blastn)** (Similarity screen admitting a query gene only when no accumulated pan-genome target reaches ANI >95% with >90% query coverage)
- **makeblastdb** (Formats pan.fasta into the searchable BLAST nucleotide database 'pan' ('-parse_seqids -dbtype nucl -out pan -title pan'))

## Examples

```
makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan
```

## Evaluation signals

- pan.fasta exists and parses as valid nucleotide FASTA at ~20 Gb size
- The 'pan' BLAST database is queryable via blastn (test search returns hits)
- The pan-genome set covers all 5082 species-level reference pan-genomes
- No admitted query gene has a best MegaBLAST target with ANI >95% and >90% query coverage against already-accepted pan-genome genes
- No more than 150 rGenomes (the longest ones) contribute to any single SGB/tCluster pan-genome

## Limitations

- Pan-genome assignment rate is highest for human gut metagenomes (99.4%) but lower for oral (89.0%) and skin (79.4%) samples, partly because oral and skin microbiomes might be mixed with microbes from the environment
- GeneMark-HM accuracy depends on the correct assignment of a metagenomic contig to a reference pan-genome, so database quality and coverage bound annotation quality
- The pan-genome file is ~20 Gb in size and download can take some time

## Evidence

- [readme] Database of species specific parameters was built for more than five thousand taxa.: [evidence span withheld — anchor status: external_doc]
