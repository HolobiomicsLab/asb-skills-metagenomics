---
name: abundance-ranked-taxon-matching
description: Use when you need to quantify how much taxonomic assignments of the same
  OTU representative sequences disagree between 16S reference databases (Greengenes
  vs SILVA vs NCBI RefSeq), pairwise at each taxonomic level from Phylum to Species.
when_to_use_negative:
- Input is raw 16S reads with no OTU representative sequences or OTU table yet — the
  DC (denoising/clustering) step must run first
- The goal is a single consensus taxonomy assignment for one database rather than
  a pairwise cross-database similarity comparison
- You need to compare the full taxon universe including rare taxa — this skill is
  scoped to the top-100 most abundant taxa per database
edam_operation: http://edamontology.org/operation_2927
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MiCoNE
  role: Provides the taxonomy-assignment step, grouping utilities for tax_levels Phylum–Species,
    and abundance ranking used to build the similarity matrix
  repo: https://github.com/segrelab/MiCoNE
- name: QIIME2
  role: Runs the Naive Bayes classifier (confidence 0.7) for Greengenes- and SILVA-trained
    taxonomic assignment
- name: Nextflow
  role: Executes the ta_op_ni pipeline template configured for the three reference-database
    assignments
- name: Python
  role: Library layer of MiCoNE used for the grouping/similarity computation and assembly
    of taxonomic_similarity_matrix.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:99750c551dd01703cf3e89ca8221f90801163b41bf1cb0a01a342a6d0ca0ad27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Nextflow
  - Python
  - QIIME2
  edam_operation: http://edamontology.org/operation_2927
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# abundance-ranked-taxon-matching

## Summary

A comparison procedure from the MiCoNE paper that quantifies how much taxonomic assignments of the same OTU representative sequences diverge between 16S reference databases (Greengenes, SILVA, NCBI RefSeq), by ranking taxa by abundance, retaining the top 100, and computing pairwise similarity at each taxonomic level.

## When to use

Use when you need to quantify how much taxonomic assignments of the same OTU representative sequences disagree between 16S reference databases (Greengenes vs SILVA vs NCBI RefSeq), pairwise at each taxonomic level from Phylum to Species.

## When NOT to use

- Input is raw 16S reads with no OTU representative sequences or OTU table yet — the DC (denoising/clustering) step must run first
- The goal is a single consensus taxonomy assignment for one database rather than a pairwise cross-database similarity comparison
- You need to compare the full taxon universe including rare taxa — this skill is scoped to the top-100 most abundant taxa per database

## Inputs

- OTU representative sequences (FASTA) from the default DC-configuration run of the MiCoNE pipeline
- Greengenes 13_8 99% 515-806-region Naive Bayes classifier (gg_13_8_99_515_806)
- SILVA-trained QIIME2 Naive Bayes classifier (same confidence setting)
- ncbi_refseq reference for BLAST-based assignment
- MiCoNE nextflow.config / pipeline template (ta_op_ni workflow)

## Outputs

- Three taxonomy assignment tables (Greengenes, SILVA, NCBI RefSeq) over the same representative sequences
- Top-100 abundance-ranked taxon lists per database at each taxonomic level
- taxonomic_similarity_matrix.csv with one row per database pair and taxonomic level (Phylum through Species)

## How to apply

Run the MiCoNE taxonomy-assignment (TA) step via the nextflow template (`micone init -w ta_op_ni`) on the OTU representative sequences from the default DC (denoising-clustering) configuration, using the QIIME2 Naive Bayes classifier trained on Greengenes gg_13_8_99_515_806 with confidence 0.7 and ncpus 1, then repeat with a SILVA-trained classifier under the same settings. For the third database, run the BLAST-based assignment against ncbi_refseq with max_accepts 10, perc_identity 0.8, evalue 0.001, and min_consensus 0.51. Using the MiCoNE Group transform, collapse abundances at tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species'], rank taxa by abundance in each of the three assignment tables, and retain only the top-100 taxa per database. Compute pairwise similarity between the three databases at each of the six taxonomic levels with the MiCoNE grouping utilities and assemble the values into taxonomic_similarity_matrix.csv with one row per database pair and level. The rationale is that abundance weighting focuses the comparison on taxa that dominate the community, where assignment differences actually affect downstream co-occurrence networks.

## Related tools

- **MiCoNE** (Provides the taxonomy-assignment step, grouping utilities for tax_levels Phylum–Species, and abundance ranking used to build the similarity matrix) — https://github.com/segrelab/MiCoNE
- **QIIME2** (Runs the Naive Bayes classifier (confidence 0.7) for Greengenes- and SILVA-trained taxonomic assignment)
- **Nextflow** (Executes the ta_op_ni pipeline template configured for the three reference-database assignments)
- **Python** (Library layer of MiCoNE used for the grouping/similarity computation and assembly of taxonomic_similarity_matrix.csv)

## Examples

```
micone init -w ta_op_ni -o gg_silva_refseq_similarity && bash run.sh
```

## Evaluation signals

- taxonomic_similarity_matrix.csv contains 3 database pairs × 6 taxonomic levels (Phylum, Class, Order, Family, Genus, Species) of pairwise similarity values
- Every below-order pairwise similarity (Family, Genus, Species) is below 45%, matching Fig. 3B of the paper
- Phylum- and class-level similarities remain higher than the below-order levels in the reproduced matrix
- Naive Bayes runs were executed at confidence 0.7 on gg_13_8_99_515_806 (ncpus 1); the BLAST run used perc_identity 0.8, evalue 0.001, max_accepts 10, min_consensus 0.51
- Each comparison uses exactly the top-100 abundance-ranked taxa per database, not the full taxon table

## Limitations

- The BLAST-based NCBI RefSeq assignment uses a different algorithm and parameters (perc_identity 0.8, evalue 0.001, max_accepts 10, min_consensus 0.51) than the Naive Bayes runs, so some observed dissimilarity is method-driven as well as database-driven
- Comparison is limited to the top-100 most abundant taxa; low-abundance taxa are excluded from the similarity estimates
- MiCoNE requires the mamba package manager for micone init, and the pipeline data directory must be downloaded manually from Zenodo record 7051556

## Evidence

- [methods] | Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |: "| Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |"
- [methods] | Assign | BLAST       | references    | "ncbi_refseq"        |: "| Assign | BLAST       | references    | "ncbi_refseq"        |"
- [methods] | Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |: "| Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |"
- [intro] we present a standardized 16S data analysis pipeline called Microbial Co-occurrence Network Explorer (MiCoNE) that produces robust and reproducible co-occurrence networks: "we present a standardized 16S data analysis pipeline called Microbial Co-occurrence Network Explorer (MiCoNE) that produces robust and reproducible co-occurrence networks"
- [readme] Example configuration files used for the analyses in the manuscript can be found [here](https://github.com/segrelab/MiCoNE-pipeline-paper/tree/master/scripts/runs).: "Example configuration files used for the analyses in the manuscript can be found [here](https://github.com/segrelab/MiCoNE-pipeline-paper/tree/master/scripts/runs)."
