---
name: gene-based-similarity-screening
description: Use when constructing a non-redundant species pan-genome reference from
  predicted genes, or when assigning human-microbiome metagenomic contigs to reference
  taxa by -match gene similarity to pick a gene prediction model.
when_to_use_negative:
- The contig cannot be reliably assigned to one of the ~5000 reference taxa (e.g.
  oral or skin metagenome contigs from environmentally mixed microbes) — GeneMark-HM
  falls back to MetaGeneMark or GeneMarkS-2 by contig length instead
- The target data are not human microbiome metagenomic contigs; the pan-genome database
  and models were built for species known to be present in human gut metagenomes
- The input is already a curated, non-redundant gene database — re-applying the ANI
  >95% / >90% coverage admission filter would not change it and would waste blastn
  compute on a ~20 Gb database
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0622
tools:
- name: GeneMark-HM
  role: Pipeline whose initial gene predictions supply the nucleotide queries screened
    against the pan-genome gene database for contig-to-pan-genome assignment
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: NCBI BLAST (blastn)
  role: 'Similarity engine: screens each query gene against accumulated pan-genome
    genes (ANI >95%, >90% query coverage non-redundancy filter) and assigns contigs
    by best match'
- name: makeblastdb
  role: Formats pan.fasta as the searchable 'pan' nucleotide BLAST database (-parse_seqids
    -dbtype nucl -out pan -title pan)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:de757a870a7899ae275bc2eb92a569de5eaec291570617bb4525860f8b384ddf
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
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# gene-based-similarity-screening

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A BLAST-driven screening strategy in the GeneMark-HM pipeline that uses initially predicted protein-coding genes as nucleotide queries against the species-level pan-genome gene database (pan) — both to build a non-redundant pan-genome reference and to assign metagenomic contigs to a taxon whose species-specific model is then used for gene prediction.

## When to use

Use when constructing a non-redundant species pan-genome reference from predicted genes, or when assigning human-microbiome metagenomic contigs to reference taxa by best-match gene similarity to pick a gene prediction model.

## When NOT to use

- The contig cannot be reliably assigned to one of the ~5000 reference taxa (e.g. oral or skin metagenome contigs from environmentally mixed microbes) — GeneMark-HM falls back to MetaGeneMark or GeneMarkS-2 by contig length instead
- The target data are not human microbiome metagenomic contigs; the pan-genome database and models were built for species known to be present in human gut metagenomes
- The input is already a curated, non-redundant gene database — re-applying the ANI >95% / >90% coverage admission filter would not change it and would waste blastn compute on a ~20 Gb database

## Inputs

- Protein-coding gene predictions (nucleotide sequences) from each reference genome (rGenome) for pan-genome construction
- Gene set of the representative genome seeding each species pan-genome
- Genes initially predicted in metagenomic contigs (query sequences at annotation time)
- pan.fasta — ~20 Gb nucleotide FASTA of concatenated pan-genome sequences formatted as the 'pan' BLAST database

## Outputs

- Non-redundant species-level pan-genome gene sequences for 5082 species (pan.fasta / 'pan' BLAST database)
- Contig-to-pan-genome assignment (best match) used to select the species-specific GeneMark.hmm-2 model
- Gene prediction model selected for each contig, or fallback routing to MetaGeneMark (shorter) / GeneMarkS-2 (longer) when no assignment is made

## How to apply

Predict protein-coding genes in every reference genome (rGenome) to build the query gene space, then seed each species pan-genome with the gene set of its representative genome. For each additional query gene, run an NCBI blastn/MegaBLAST screen against the accumulated pan-genome genes and admit the gene only if no target reaches ANI >95% with >90% query coverage (the non-redundancy filter). Restrict the genomes contributing to each species pan-genome to the 150 longest rGenomes per SGB, concatenate the accepted sequences for all 5082 species into the ~20 Gb nucleotide FASTA pan.fasta, and format it with makeblastdb (-parse_seqids -dbtype nucl -out pan -title pan). At annotation time, the genes initially predicted in each metagenomic contig serve as queries for a fast similarity search through this database; the best matches select the taxon-specific model used by GeneMark.hmm-2.

## Related tools

- **GeneMark-HM** (Pipeline whose initial gene predictions supply the nucleotide queries screened against the pan-genome gene database for contig-to-pan-genome assignment) — https://github.com/GeneProbe/GeneMark-HM
- **NCBI BLAST (blastn)** (Similarity engine: screens each query gene against accumulated pan-genome genes (ANI >95%, >90% query coverage non-redundancy filter) and assigns contigs by best match)
- **makeblastdb** (Formats pan.fasta as the searchable 'pan' nucleotide BLAST database (-parse_seqids -dbtype nucl -out pan -title pan))

## Examples

```
makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan
```

## Evaluation signals

- pan.fasta exists and parses as valid nucleotide FASTA
- The 'pan' BLAST database is queryable via blastn after makeblastdb -parse_seqids -dbtype nucl
- The pan-genome set covers all 5082 species-level reference pan-genomes
- Every admitted query gene lacks any MegaBLAST target with ANI >95% AND >90% query coverage (non-redundancy invariant)
- No more than 150 rGenomes (the 150 longest per SGB/tCluster) contribute genes to any single species pan-genome

## Limitations

- GeneMark-HM accuracy depends on correct assignment of each metagenomic contig to a reference pan-genome; a wrong assignment degrades prediction
- Pan-genome assignment coverage varies by niche: 99.4% for human gut metagenomes but only 89.0% (oral) and 79.4% (skin), partly because oral and skin microbiomes might be mixed with microbes from the environment
- The pan.fasta file is ~20 Gb, so download and database construction are slow; no version numbers are stated for GeneMark-HM, GeneMark.hmm-2, GeneMarkS-2, or MetaGeneMark

## Evidence

- [readme] Database of genes is used on taxa assignment step and database of species specific parameters on gene prediction step.: [evidence span withheld — anchor status: external_doc]
