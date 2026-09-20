---
name: minhash-genome-sketching
description: Use when you have many genome assemblies (e.g. 3269 MAG FASTA files)
  and need to find each one's closest relative in a reference genome set (e.g. 2505
  HGR genomes) to assign species labels.
when_to_use_negative:
- You need direct read-level taxonomic classification of metagenomic reads rather
  than genome-to-genome comparison — use DeepMicrobes or Kraken instead
- Genomes are highly fragmented so the 60% aligned-query-fraction criterion would
  mislabel novel species as matches; fragmentation degrades alignment-based classifier
  performance as noted in the paper
- You only need coarse similarity clustering of genomes without ANI confirmation (a
  single Mash-distance clustering may suffice)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Mash
  role: MinHash k-mer sketching of query and reference genomes and fast pairwise distance
    estimation (`mash dist`) to find each MAG's closest relative
- name: MUMmer
  role: Whole-genome alignment (nucmer/dnadiff, v3.23) of each MAG against its closest
    Mash relative to compute aligned query fraction (AQ) and ANI for species-label
    decisions
- name: R
  role: Downstream evaluation/plotting of mashdiff results, e.g. mashdiff-hist_scatter
    and mashdiff-counts scripts
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:9488277f2f139f25d5a9ac5bd4ef375133c63f7103ac3d9dec661313bd101706
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MicrobeLab/DeepMicrobes
  tools:
  - DeepMicrobes
  - MUMmer
  - Mash
  - R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# MinHash genome sketching for closest-relative search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rapidly computes pairwise genome distances by sketching genome FASTA files with Mash (MinHash k-mer sketches), identifying each query genome's closest relative among a reference database, then confirming the match with MUMmer whole-genome alignment. Used to assign species labels to benchmark MAGs relative to the 2505 HGR training genomes.

## When to use

Use when you have many genome assemblies (e.g. 3269 MAG FASTA files) and need to find each one's closest relative in a reference genome set (e.g. 2505 HGR genomes) to assign species labels.

## When NOT to use

- You need direct read-level taxonomic classification of metagenomic reads rather than genome-to-genome comparison — use DeepMicrobes or Kraken instead
- Genomes are highly fragmented so the 60% aligned-query-fraction criterion would mislabel novel species as matches; fragmentation degrades alignment-based classifier performance as noted in the paper
- You only need coarse similarity clustering of genomes without ANI confirmation (a single Mash-distance clustering may suffice)

## Inputs

- Query genome assemblies in FASTA format with `.fa` extension (e.g. 3269 gut-derived MAGs from ERP108418)
- Reference genome FASTA files (e.g. 2505 HGR training genomes)
- A Mash reference sketch file `.msh` generated with `mash sketch`

## Outputs

- Per-query `*dbname_parsed.tab` file with columns: Query name / Reference name / Ref length / % Ref covered / Query length / % Query aligned / ANI / Mash distance
- Per-MAG species assignment table (mag_species_assignment.tsv) with closest-relative, AQ, ANI, and same-species label columns

## How to apply

Convert every query MAG assembly and every reference training genome (2505 HGR species-repertoire genomes) into MinHash sketches with `mash sketch`, then run `mash dist` between each MAG sketch and all HGR sketches to find the lowest-Mash-distance closest relative. Next, align each MAG against its closest HGR relative with MUMmer 3.23 (nucmer/dnadiff) and compute the aligned query fraction (AQ) and average nucleotide identity (ANI) from the alignment. Apply the paper's species-assignment decision rule: label a MAG as the same species as its closest relative only when AQ > 60% and ANI > 95%; otherwise the MAG is treated as absent from the reference (this distinguishes true positives from false positives on reads simulated from MAGs excluded from training). The MGS-gut README's `mashdiff.sh` implements exactly this pipeline, parsing dnadiff and Mash results per query genome.

## Related tools

- **Mash** (MinHash k-mer sketching of query and reference genomes and fast pairwise distance estimation (`mash dist`) to find each MAG's closest relative)
- **MUMmer** (Whole-genome alignment (nucmer/dnadiff, v3.23) of each MAG against its closest Mash relative to compute aligned query fraction (AQ) and ANI for species-label decisions)
- **R** (Downstream evaluation/plotting of mashdiff results, e.g. mashdiff-hist_scatter and mashdiff-counts scripts) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Examples

```
mashdiff.sh -i genome_folder/ -r reference.msh -s hgr -p gut
```

## Evaluation signals

- Output table contains exactly one row per query MAG (3269 rows) with AQ, ANI, closest-relative and species-label fields populated
- Same-species labels occur only for rows with AQ > 60% AND ANI > 95%; any labeled row violating either threshold is an error
- Each MAG's closest reference match corresponds to the minimum Mash distance across all 2505 HGR sketches
- Parsed dnadiff/Mash output contains the expected 8 columns: Query name, Reference name, Ref length, % Ref covered, Query length, % Query aligned, ANI, Mash distance
- Reads simulated from MAGs labeled as novel (AQ 10–60%, ANI < 95%) should count as false positives in species classifiers trained without them (per Table S6)

## Limitations

- The species-assignment rule (AQ > 60%, ANI > 95%) was tuned for this gut-microbiota benchmark; MAG fragmentation can push AQ below the threshold and cause true reference species to be missed
- Mash MinHash sketches are approximate; closest-relative identification should be confirmed by whole-genome alignment as done here with MUMmer 3.23
- mashdiff.sh requires Python 2.7 and was coded for LSF cluster environments; Mash tested at v2.0 and MUMmer at v3.23

## Evidence

- [methods] Species labels were assigned by first converting MAGs and training genomes into MinHash sketches to find each MAG's closest relative via lowest Mash distance: [evidence span withheld — anchor status: not_found]
- [methods] labeling a MAG as the same species as its closest relative only when AQ >60% and ANI >95%: [evidence span withheld — anchor status: not_found]
- [readme] Compare a set of genomes against a reference database. Selects best representative for whole-genome alignment.: [evidence span withheld — anchor status: external_doc]
- [readme] Query name / Reference name / Ref length / % Ref covered / Query length / % Query aligned / ANI / Mash distance: [evidence span withheld — anchor status: not_found]
- [supplementary] Reads simulated from 3,269 gut-derived MAGs: [evidence span withheld — anchor status: external_doc]
