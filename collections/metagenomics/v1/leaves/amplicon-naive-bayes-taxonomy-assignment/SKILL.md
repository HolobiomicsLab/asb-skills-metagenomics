---
name: amplicon-naive-bayes-taxonomy-assignment
description: Use when OTU representative sequences need taxonomic labels under alternative
  16S reference databases (Greengenes, SILVA, NCBI RefSeq) to quantify how database
  choice affects downstream community-composition inferences.
when_to_use_negative:
- Input is raw 16S reads that have not yet been demultiplexed, denoised/clustered
  into OTU representative sequences (the TA step expects qiime2-supported sequence
  inputs)
- The goal is comparing taxonomic diversity or alpha/beta metrics rather than comparing
  assignments across reference databases
- A de novo OTU or ASV workflow outside MiCoNE's configuration system is required
  (e.g., custom classifier training beyond gg_13_8_99_515_806 / SILVA defaults)
edam_operation: http://edamontology.org/operation_0290
tools:
- name: MiCoNE
  role: Python package and modular pipeline whose taxonomy-assignment (TA) step orchestrates
    classifier-based and BLAST-based assignment and provides the grouping utilities
    used to compare the results
  repo: https://github.com/segrelab/MiCoNE
- name: QIIME2
  role: Provides the Naive Bayes classifier training/assignment backend executed inside
    the micone-qiime2 conda environment
- name: Nextflow
  role: Executes the MiCoNE workflow template configured for the TA step; templates
    are initialized with `micone init`
  repo: https://github.com/segrelab/MiCoNE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:0d8109991a62f7bada274b554610e33679242879fe031bf74ab2593a96188f40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Nextflow
  - Python
  - QIIME2
  edam_operation: http://edamontology.org/operation_0290
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# amplicon-naive-bayes-taxonomy-assignment

## Summary

Assigns taxonomy to 16S OTU representative sequences using the MiCoNE taxonomy-assignment (TA) step: QIIME2 Naive Bayes classifiers trained on Greengenes (gg_13_8_99_515_806) or SILVA at confidence 0.7, and BLAST against ncbi_refseq, enabling database-comparison analyses.

## When to use

Use when OTU representative sequences need taxonomic labels under alternative 16S reference databases (Greengenes, SILVA, NCBI RefSeq) to quantify how database choice affects downstream community-composition inferences.

## When NOT to use

- Input is raw 16S reads that have not yet been demultiplexed, denoised/clustered into OTU representative sequences (the TA step expects qiime2-supported sequence inputs)
- The goal is comparing taxonomic diversity or alpha/beta metrics rather than comparing assignments across reference databases
- A de novo OTU or ASV workflow outside MiCoNE's configuration system is required (e.g., custom classifier training beyond gg_13_8_99_515_806 / SILVA defaults)

## Inputs

- OTU representative sequences from the default DC configuration of the MiCoNE pipeline (qiime2-supported format)
- Greengenes 13_8 99% 515/806-trained Naive Bayes classifier (gg_13_8_99_515_806)
- SILVA-trained Naive Bayes classifier
- ncbi_refseq reference sequences for BLAST assignment

## Outputs

- Per-database taxonomy assignment tables over six tax_levels: Phylum, Class, Order, Family, Genus, Species
- Abundance-ranked top-100 taxa tables per reference database
- taxonomic_similarity_matrix.csv with one row per database pair and taxonomic level

## How to apply

Run the MiCoNE TA step on the OTU representative sequences from the default DC configuration via the Nextflow workflow initialized with `micone init`. First assign taxonomy with the QIIME2 Naive Bayes classifier trained on Greengenes gg_13_8_99_515_806 at confidence 0.7 with ncpus 1, then repeat the Naive Bayes assignment with a SILVA-trained classifier under the same confidence. Also run the BLAST-based assignment against the ncbi_refseq reference using max_accepts 10, perc_identity 0.8, evalue 0.001, and min_consensus 0.51. Rank the taxa in each of the three resulting assignment tables by abundance and group them at the tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species'] using the MiCoNE grouping utilities. The rationale, per the source study, is that different reference databases lead to vastly different community-composition inferences, so assignments must be produced identically except for the classifier/reference to isolate that effect.

## Related tools

- **MiCoNE** (Python package and modular pipeline whose taxonomy-assignment (TA) step orchestrates classifier-based and BLAST-based assignment and provides the grouping utilities used to compare the results) — https://github.com/segrelab/MiCoNE
- **QIIME2** (Provides the Naive Bayes classifier training/assignment backend executed inside the micone-qiime2 conda environment)
- **Nextflow** (Executes the MiCoNE workflow template configured for the TA step; templates are initialized with `micone init`) — https://github.com/segrelab/MiCoNE

## Examples

```
micone init -w ta_op_ni -o pipeline_dir && bash run.sh
```

## Evaluation signals

- All three databases produce assignment tables with taxonomic levels exactly ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species'] as in the MiCoNE Group step config
- Pairwise similarity between Greengenes, SILVA, and NCBI RefSeq assignments is below 45% at every level below order (Family, Genus, Species), matching Fig. 3B
- Phylum- and class-level similarity remains higher than sub-order similarity for every database pair
- The BLAST run used max_accepts 10, perc_identity 0.8, evalue 0.001, min_consensus 0.51, and both Naive Bayes runs used confidence 0.7 — confirm via nextflow.config
- `micone validate-results` passes on the pipeline run output

## Limitations

- Database assignments diverge sharply: pairwise similarity drops below 45% at Family, Genus, and Species levels, so assignments from each reference are 'fairly unique' and not interchangeable
- The BLAST-based NCBI RefSeq assignment uses different mechanics (max_accepts 10, perc_identity 0.8, min_consensus 0.51) than the Naive Bayes runs, so part of the difference reflects method as well as reference content
- The pipeline's data directory must be manually downloaded from Zenodo (record 7051556); MiCoNE requires mamba for `micone init` to work

## Evidence

- [methods] Greengenes 99% classifier used for Naive Bayes assignment: "| Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |"
- [methods] BLAST assignment configured against ncbi_refseq reference: "| Assign | BLAST       | references    | "ncbi_refseq"        |"
- [readme] MiCoNE supports alternate tools per pipeline step, configured via the config file: "Each process in the pipeline supports alternate tools for performing the same task, users can use the configuration file to change these values."
- [readme] MiCoNE purpose: exploring tool-choice effects in 16S processing: "`MiCoNE` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow"
- [intro] Different methods lead to vastly different community-composition inferences, motivating controlled database comparisons: "The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks"
