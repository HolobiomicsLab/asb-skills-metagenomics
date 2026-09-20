---
name: fungal-single-copy-marker-gene-identification
description: Use when constructing a fungal taxonomic marker-gene database from a
  pool of assembled, quality-filtered fungal genomes (e.g., ~4,839 NCBI/JGI MycoCosm
  genomes) and you need per-genome single-copy ortholog extraction and redundancy
  removal.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled, quality-filtered genomes —
  BUSCO genome mode with AUGUSTUS requires assemblies
- Goal is building the functional protein database (FunOMIC-P) — that uses annotated
  coding proteins clustered at 95% identity, not single-copy marker genes at 99%
- A genome yields fewer than 30 single-copy marker genes — it should be discarded
  from the marker catalog rather than included
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0622
tools:
- name: BUSCO 5.0.0
  role: Identifies single-copy marker genes in each fungal genome against the 758
    Fungi OrthoDB v10.1 universal ortholog HMMs
- name: AUGUSTUS
  role: Gene predictor used by BUSCO in default mode to call marker-gene sequences
- name: CD-HIT
  role: Clusters extracted marker genes at a 99% identity threshold to remove redundancy
    and produce the final catalog
- name: bowtie2
  role: Maps the final marker-gene catalog to the UHGG prokaryotic database to validate
    fungal specificity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:bf6d35eeb18e6a4dba7e0516e905853a86b0de67487744d3a2b85b8db20255bf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - AUGUSTUS
  - BUSCO
  - BUSCO 5.0.0
  - CD-HIT
  - FunOMIC
  - bowtie2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# fungal-single-copy-marker-gene-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies single-copy marker genes in curated fungal genomes using BUSCO 5.0.0 with the Fungi OrthoDB v10.1 set of 758 universal single-copy ortholog HMMs and the AUGUSTUS gene predictor, then deduplicates them into a taxonomic marker catalog (FunOMIC-T, ~1.69 million genes).

## When to use

Use when constructing a fungal taxonomic marker-gene database from a pool of assembled, quality-filtered fungal genomes (e.g., ~4,839 NCBI/JGI MycoCosm genomes) and you need per-genome single-copy ortholog extraction and redundancy removal.

## When NOT to use

- Input is raw sequencing reads rather than assembled, quality-filtered genomes — BUSCO genome mode with AUGUSTUS requires assemblies
- Goal is building the functional protein database (FunOMIC-P) — that uses annotated coding proteins clustered at 95% identity, not single-copy marker genes at 99%
- A genome yields fewer than 30 single-copy marker genes — it should be discarded from the marker catalog rather than included

## Inputs

- Quality-filtered assembled fungal genomes (FASTA; contigs/scaffolds, e.g., 4,839 genomes passing >500 contigs / N50 <10 kbp filters, from NCBI and JGI MycoCosm)
- Fungi OrthoDB version 10.1 single-copy ortholog HMM set (758 universal fungal orthologs)

## Outputs

- Per-genome BUSCO single-copy marker gene counts
- Set of retained genomes passing the ≥30 single-copy marker gene threshold (e.g., 4,816 genomes)
- Non-redundant marker-gene nucleotide catalog (CD-HIT 99% identity; funomic_t_marker_genes.fasta, ~1.69 million genes)
- Per-species/per-genus marker coverage table for taxonomic profiling

## How to apply

First curate the genome pool: start from strain-level fungal genomes/draft genomes (e.g., 9,401 from NCBI and JGI MycoCosm) and discard assemblies with >500 contigs or N50 <10 kbp, giving a working set such as the paper's 4,839 genomes. Run BUSCO 5.0.0 in default genome mode with the AUGUSTUS gene predictor against the 758 fungal single-copy ortholog HMMs of Fungi OrthoDB version 10.1 on each genome. Apply the completeness filter: discard any genome yielding fewer than 30 single-copy marker genes (the paper's threshold reduced the pool to 4,816 genomes). Extract the marker-gene nucleotide sequences from retained genomes and cluster them with CD-HIT at a 99% identity threshold to remove redundancy, yielding the final catalog (~1.6–1.69 million marker genes, funomic_t_marker_genes.fasta). Tabulate marker-gene presence per taxon to build per-species marker coverage tables (in the paper: 475 genera, 1,916 species, 4,537 strains across eight phyla). Validate specificity by mapping the catalog against a prokaryotic reference (UHGG) with bowtie2: fewer than 0.01% of fungal marker genes should map.

## Related tools

- **BUSCO 5.0.0** (Identifies single-copy marker genes in each fungal genome against the 758 Fungi OrthoDB v10.1 universal ortholog HMMs)
- **AUGUSTUS** (Gene predictor used by BUSCO in default mode to call marker-gene sequences)
- **CD-HIT** (Clusters extracted marker genes at a 99% identity threshold to remove redundancy and produce the final catalog)
- **bowtie2** (Maps the final marker-gene catalog to the UHGG prokaryotic database to validate fungal specificity)

## Examples

```
busco -i genome.fasta -m genome --lineage fungi_odb10 --out genome_buscogenes && cd-hit-est -i markers_all.fasta -o funomic_t_marker_genes.fasta -c 0.99
```

## Evaluation signals

- Retained genome count matches the ≥30 single-copy marker gene filter (paper: 4,839 → 4,816 genomes)
- CD-HIT 99% clustering yields a catalog of the expected scale (~1.6–1.69 million fungal marker genes)
- Taxonomic coverage invariant: catalog spans 475 genera, 1,916 species, and 4,537 strains across eight fungal phyla
- Specificity check passes: fewer than 0.01% of marker genes map to the prokaryotic UHGG catalogue
- Bacterial mock simulation check: none of 999,021 bacterial reads from a 903-genome mock community align to FunOMIC-T (specificity 0.9999)

## Limitations

- Extraction and quality of single-copy marker genes rely on the completeness of available fungal genomes, which may result in lower coverage of fungal taxonomy compared with amplicon databases
- Genomes with fewer than 30 identified single-copy marker genes must be discarded, excluding incomplete assemblies from the catalog
- Catalog building is a one-off database-construction step; FunOMIC-T does not include a bacterial read filtration step before downstream functional annotation

## Evidence

- [methods] 99% identity CD-HIT clustering: [evidence span withheld — anchor status: not_found]
