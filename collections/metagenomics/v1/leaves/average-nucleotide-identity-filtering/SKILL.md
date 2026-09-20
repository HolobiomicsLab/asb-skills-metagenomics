---
name: average-nucleotide-identity-filtering
description: Use when constructing a non-redundant pan-genome gene database from many
  reference genomes, or when assigning metagenomic contigs to species-level pan-genomes
  by BLASTn of predicted genes using ANI thresholds.
when_to_use_negative:
- Complete or nearly complete genomes are available for the sequence itself — GeneMarkS-2
  self-training is used instead of pan-genome model selection
- The metagenome is not from the human microbiome — the pan-genome database covers
  only ~5082 taxa known from human microbiome metagenomes (pan.fasta is human-microbiome
  specific)
- Contigs are too short to produce reliable initial MetaGeneMark gene predictions,
  since those predictions are the blastn queries driving the ANI-based assignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: NCBI BLAST (blastn)
  role: Similarity search providing ANI, alignment length and E-value used for both
    pan-genome non-redundancy filtering and contig-to-pan-genome assignment
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: makeblastdb
  role: Formats pan.fasta as the searchable nucleotide BLAST database ('-parse_seqids
    -dbtype nucl -out pan -title pan') targeted by the blastn queries
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaGeneMark
  role: Supplies the initial gene predictions used as blastn queries for the ANI-filtered
    contig assignment step
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMark-HM
  role: Pipeline wrapper (gm_hm.pl) that integrates the filtered similarity search,
    majority-vote assignment, and model selection
  repo: https://github.com/GeneProbe/GeneMark-HM
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:7ea9c8fca74de151f93d40ff327b6f9528fb923032db83d980728c1727c2d9f0
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
  - MetaGeneMark
  - NCBI BLAST (blastn)
  - makeblastdb
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# average-nucleotide-identity-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies ANI-based hit filtering at two points in the GeneMark-HM human microbiome pipeline: admitting non-redundant genes into species-level pan-genomes (MegaBLAST ANI >95% with >90% query coverage), and retaining blastn hits between initially predicted contig genes and the pan-genome database (E-value <1e-5, >90 nt aligned, ANI >95%) to assign contigs to a taxon-specific prediction model.

## When to use

Use when constructing a non-redundant pan-genome gene database from many reference genomes, or when assigning metagenomic contigs to species-level pan-genomes by BLASTn of predicted genes using ANI thresholds.

## When NOT to use

- Complete or nearly complete genomes are available for the sequence itself — GeneMarkS-2 self-training is used instead of pan-genome model selection
- The metagenome is not from the human microbiome — the pan-genome database covers only ~5082 taxa known from human microbiome metagenomes (pan.fasta is human-microbiome specific)
- Contigs are too short to produce reliable initial MetaGeneMark gene predictions, since those predictions are the blastn queries driving the ANI-based assignment

## Inputs

- Protein-coding gene predictions from every reference genome (rGenome) in nucleotide/query gene space
- Species-level pan-genome gene sequences (pan.fasta, ~20 Gb nucleotide FASTA) as a BLAST nucleotide database
- Genes initially predicted by MetaGeneMark in each metagenomic contig (e.g. ../genemark.gtf)
- Metagenomic assembly contigs in FASTA (e.g. ../sequence.fasta)

## Outputs

- 5082 species-level pan-genomes of non-redundant genes (pan.fasta) plus a queryable 'pan' blastn database
- Per-hit filtered BLAST results retaining only E-value <1e-5, >90 nt alignment, ANI >95% matches
- Contig-to-pan-genome assignments selecting taxon-specific model files from the refmod library

## How to apply

For pan-genome construction, seed each species pan-genome with its representative genome's genes, then for each additional query gene run NCBI blastn against the accumulated pan-genome genes and admit the gene only if no target reaches ANI >95% with >90% query coverage; restrict contributing genomes to the 150 longest rGenomes per SGB, yielding 5082 species pan-genomes concatenated into the ~20 Gb pan.fasta. Format the database with makeblastdb '-parse_seqids -dbtype nucl -out pan -title pan'. For contig-to-pan-genome assignment, use MetaGeneMark initial gene predictions (e.g. ../genemark.gtf) as blastn queries against the pan database, keep hits with E-value <1e-5, alignment length >90 nt, and ANI >95%, then assign each contig to the pan-genome holding the majority of its matched genes, which triggers selection of the corresponding pre-computed model from the refmod library. The rationale is that ANI >95% defines close-relative (species-level) matches, letting species-specific parameters replace crude GC-based models.

## Related tools

- **NCBI BLAST (blastn)** (Similarity search providing ANI, alignment length and E-value used for both pan-genome non-redundancy filtering and contig-to-pan-genome assignment) — https://github.com/GeneProbe/GeneMark-HM
- **makeblastdb** (Formats pan.fasta as the searchable nucleotide BLAST database ('-parse_seqids -dbtype nucl -out pan -title pan') targeted by the blastn queries) — https://github.com/GeneProbe/GeneMark-HM
- **MetaGeneMark** (Supplies the initial gene predictions used as blastn queries for the ANI-filtered contig assignment step) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMark-HM** (Pipeline wrapper (gm_hm.pl) that integrates the filtered similarity search, majority-vote assignment, and model selection) — https://github.com/GeneProbe/GeneMark-HM

## Examples

```
makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan
```

## Evaluation signals

- pan.fasta exists and parses as nucleotide FASTA; the 'pan' BLAST database is queryable via blastn and the pan-genome set covers the 5082 species
- No admitted hit fails the thresholds: E-value <1e-5, aligned query length >90 nt, ANI >95% for assignment; admitted pan-genome genes lack any target with ANI >95% and >90% query coverage
- Each contig is assigned to the pan-genome holding the majority of its ANI-filtered matched genes
- Run the bundled test and compare with '../../bin/compp -a ../genemark.gtf -b output.gtf' to confirm the reported low false-positive, high-sensitivity behaviour
- Expected contig-to-pan-genome assignment rates: up to 99.4% for human gut metagenomes, 89.0% oral, 79.4% skin

## Limitations

- GeneMark-HM accuracy depends on the correct assignment of a metagenomic contig to a reference pan-genome; a wrong assignment propagates a wrong prediction model
- Assignment coverage drops for oral (89.0%) and skin (79.4%) metagenomes, possibly because these microbiomes are mixed with environmental microbes; unmatched contigs fall back to crude GC-specific models or GeneMarkS-2
- Only the 150 longest rGenomes per SGB/tCluster contribute to each species pan-genome, and the pan-genome file is ~20 Gb, so downloads and BLAST database construction are resource-heavy

## Evidence

- [abstract] Genes initially predicted in each contig served as queries for a fast similarity search through the pan-genome database. The best matches led to selection of the model for gene prediction.: "Genes initially predicted in each contig served as queries for a fast similarity search through the pan-genome database."
- [abstract] Contigs not assigned to pan-genomes were analyzed by crude, but still accurate models designed for sequences with particular GC compositions.: "Contigs not assigned to pan-genomes were analyzed by crude, but still accurate models designed for sequences with particular GC compositions."
- [methods] Pan-genome file size and database build: "Pan-genome file is ~20 Gb in size and download can take some time."
- [readme] Database of genes is used on taxa assignment step: [evidence span withheld — anchor status: external_doc]
- [readme] Pipeline checks reliable taxa assignment: [evidence span withheld — anchor status: external_doc]
