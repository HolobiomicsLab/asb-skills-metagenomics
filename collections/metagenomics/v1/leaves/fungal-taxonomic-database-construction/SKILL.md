---
name: fungal-taxonomic-database-construction
description: Use when building a strain/species-level fungal taxonomic reference database
  for read-mapping mycobiome profiling from NCBI/JGI fungal genome assemblies.
when_to_use_negative:
- You need a functional (protein/pathway) reference rather than a taxonomic marker
  catalog — use the 95% CD-HIT clustering of annotated fungal proteins (FunOMIC-P)
  instead
- Your target taxa are bacteria/archaea — marker genes from fungal OrthoDB 10.1 will
  not capture prokaryotic diversity
- Input data are already metagenomic reads rather than reference genome assemblies
  — this skill constructs the reference, not the read-mapping profiling step
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0797
tools:
- name: BUSCO 5.0.0
  role: Identifies single-copy marker genes in each fungal genome using Fungi OrthoDB
    v10.1 HMMs with the AUGUSTUS gene predictor
- name: AUGUSTUS
  role: Gene predictor used in default BUSCO mode to call marker-gene sequences
- name: CD-HIT
  role: Clusters extracted marker genes at a 99% identity threshold to remove redundancy,
    producing the final catalog
- name: bowtie2
  role: Maps the 1.69 million marker genes to UHGG to validate database fungal specificity
- name: FunOMIC
  role: Pipeline into which the resulting FunOMIC-T taxonomic database is integrated
    for automatic mycobiome profiling
  repo: https://github.com/ManichanhLab/FunOMIC
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:d8bafb1ec422b8bf491936a3a0a4e125dbb5b08f44323da596864f5189517fc2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - AUGUSTUS
  - BUSCO
  - BUSCO 5.0.0
  - CD-HIT
  - FunOMIC
  - bowtie2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# fungal-taxonomic-database-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs a fungal single-copy marker-gene taxonomic reference (like FunOMIC-T, ~1.69 million genes) by BUSCO-based marker extraction from thousands of curated fungal genomes followed by 99% identity CD-HIT de-redundancy.

## When to use

Use when building a strain/species-level fungal taxonomic reference database for read-mapping mycobiome profiling from NCBI/JGI fungal genome assemblies.

## When NOT to use

- You need a functional (protein/pathway) reference rather than a taxonomic marker catalog — use the 95% CD-HIT clustering of annotated fungal proteins (FunOMIC-P) instead
- Your target taxa are bacteria/archaea — marker genes from fungal OrthoDB 10.1 will not capture prokaryotic diversity
- Input data are already metagenomic reads rather than reference genome assemblies — this skill constructs the reference, not the read-mapping profiling step

## Inputs

- Fungal genome assemblies or draft genomes from NCBI and JGI MycoCosm (e.g., Supplementary Table S8 accession list of 4,839 reference genomes)
- Fungi OrthoDB version 10.1 set of 758 fungal single-copy ortholog HMMs
- Quality-filter metadata: contig counts and N50 per assembly

## Outputs

- funomic_t_marker_genes.fasta: non-redundant single-copy marker-gene catalog (~1.69 million sequences after 99% CD-HIT clustering)
- Per-species/per-taxon marker coverage table (1,916 species, 4,537 strains, 475 genera, eight fungal phyla)
- Bowtie2 mapping report of marker genes vs. UHGG quantifying cross-kingdom contamination (<0.01% expected)

## How to apply

Collect publicly available strain-level fungal genomes (e.g., 9,401 from NCBI and JGI MycoCosm) and quality-filter them, discarding assemblies with >500 contigs or N50 < 10 kbp. Run BUSCO 5.0.0 with Fungi OrthoDB version 10.1 (758 fungal single-copy ortholog HMMs) in default mode with the AUGUSTUS gene predictor to identify single-copy marker genes per genome, then discard genomes yielding fewer than 30 single-copy marker genes (leaving 4,816 in the reference workflow). Extract the marker-gene nucleotide sequences and cluster them with CD-HIT at a 99% identity threshold to remove redundancy, yielding a catalog of ~1.69 million fungal marker genes (FunOMIC-T). Validate specificity by mapping all marker genes against a prokaryotic reference (UHGG) with bowtie2 and confirming fewer than 0.01% map.

## Related tools

- **BUSCO 5.0.0** (Identifies single-copy marker genes in each fungal genome using Fungi OrthoDB v10.1 HMMs with the AUGUSTUS gene predictor)
- **AUGUSTUS** (Gene predictor used in default BUSCO mode to call marker-gene sequences)
- **CD-HIT** (Clusters extracted marker genes at a 99% identity threshold to remove redundancy, producing the final catalog)
- **bowtie2** (Maps the 1.69 million marker genes to UHGG to validate database fungal specificity)
- **FunOMIC** (Pipeline into which the resulting FunOMIC-T taxonomic database is integrated for automatic mycobiome profiling) — https://github.com/ManichanhLab/FunOMIC

## Examples

```
busco -i genome.fasta -m genome --lineage_dataset fungi_odb10 & cd-hit -i markers.fasta -o funomic_t_marker_genes.fasta -c 0.99
```

## Evaluation signals

- Final genome set count matches expectation: 4,816 genomes retained after discarding assemblies with <30 single-copy marker genes
- Final catalog size is ~1.69 million non-redundant marker genes after 99% CD-HIT clustering (reduced from >1.6M extracted per-genome markers)
- Fewer than 0.01% of the marker genes map to the prokaryotic UHGG catalogue (fungal specificity check)
- Bacterial mock simulation: none of 999,021 simulated bacterial reads (903 genomes, 458 species) align to FunOMIC-T, specificity 0.9999
- Taxonomic coverage table reports 475 genera, 1,916 species, 4,537 strains across eight fungal phyla

## Limitations

- Extraction and quality of single-copy marker genes depend on the completeness of available fungal genomes, which may result in lower fungal taxonomy coverage compared with amplicon databases
- Genome quality filters (>500 contigs, N50 < 10 kbp) exclude highly fragmented assemblies, so poorly represented taxa may be missed
- Database construction is limited to taxa with publicly available reference genomes at build time (4,839 curated genomes in the reference run)

## Evidence
