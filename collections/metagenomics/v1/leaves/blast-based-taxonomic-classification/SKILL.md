---
name: blast-based-taxonomic-classification
description: Use when OTU representative sequences need taxonomic labels via sequence-similarity
  search against NCBI RefSeq, or when comparing assignment consistency across 16S
  reference databases (Greengenes vs SILVA vs NCBI RefSeq).
when_to_use_negative:
- The input is raw 16S reads rather than OTU representative sequences — demultiplexing,
  trimming, and OTU assignment must precede this step
- A single-database assignment with no interest in cross-database consistency — then
  only one assignment run is needed and the similarity matrix is unnecessary
- Full-length or long-read amplicons where the default MiCoNE 515/806-region classifiers
  and references are not appropriate
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0114
tools:
- name: MiCoNE
  role: Pipeline framework executing the taxonomy-assignment (TA) step and providing
    the Group utilities for the six-level similarity computation
  repo: https://github.com/segrelab/MiCoNE
- name: QIIME2
  role: Hosts the BLAST-based consensus classifier (classify-consensus-blast) and
    the Naive Bayes classifiers used in the comparison
- name: Nextflow
  role: Workflow engine that runs the MiCoNE pipeline template and parallelizes the
    TA step across samples
  repo: https://micone.readthedocs.io/
- name: Python
  role: MiCoNE library used to load assignment tables and compute the taxonomic similarity
    matrix
  repo: https://github.com/segrelab/MiCoNE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:413e020970843bbf53e6e89204b54dd32133606a4fb56fa0e7d22d3ab25d5084
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Nextflow
  - Python
  - QIIME2
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# blast-based-taxonomic-classification

## Summary

Assigns taxonomy to OTU representative sequences using BLAST search against the NCBI RefSeq 16S reference, as implemented in the MiCoNE pipeline's taxonomy-assignment (TA) step, and supports cross-database comparison with Naive Bayes classifiers trained on Greengenes and SILVA.

## When to use

Use when OTU representative sequences need taxonomic labels via sequence-similarity search against NCBI RefSeq, or when comparing assignment consistency across 16S reference databases (Greengenes vs SILVA vs NCBI RefSeq).

## When NOT to use

- The input is raw 16S reads rather than OTU representative sequences — demultiplexing, trimming, and OTU assignment must precede this step
- A single-database assignment with no interest in cross-database consistency — then only one assignment run is needed and the similarity matrix is unnecessary
- Full-length or long-read amplicons where the default MiCoNE 515/806-region classifiers and references are not appropriate

## Inputs

- OTU representative sequences (qiime2-supported formats) from the default DC configuration
- ncbi_refseq reference database (shipped with MiCoNE's data directory from Zenodo record 7051556)
- Greengenes gg_13_8_99_515_806 and SILVA-trained Naive Bayes classifiers (for the comparison arm)

## Outputs

- Taxonomy assignment table per reference database (Greengenes, SILVA, NCBI RefSeq)
- Top-100 most abundant taxa per assignment table
- taxonomic_similarity_matrix.csv with one row per database pair and taxonomic level (Phylum, Class, Order, Family, Genus, Species)

## How to apply

In the MiCoNE pipeline (executed through the Nextflow workflow template with the micone-qiime2 environment), run the BLAST-based assignment on OTU representative sequences from the default DC configuration against the ncbi_refseq reference with max_accepts 10, perc_identity 0.8, evalue 0.001, and min_consensus 0.51. For the cross-database comparison (Fig. 3B), run the QIIME2 Naive Bayes classifier in parallel, trained on Greengenes gg_13_8_99_515_806 and SILVA, both at confidence 0.7 with ncpus 1. Rank taxa in each of the three assignment tables by abundance, retain the top-100 taxa, and compute pairwise similarity between the databases at each taxonomic level (Phylum, Class, Order, Family, Genus, Species) using the MiCoNE Group step's tax_levels setting. Assemble the similarity values into taxonomic_similarity_matrix.csv, one row per database pair and taxonomic level.

## Related tools

- **MiCoNE** (Pipeline framework executing the taxonomy-assignment (TA) step and providing the Group utilities for the six-level similarity computation) — https://github.com/segrelab/MiCoNE
- **QIIME2** (Hosts the BLAST-based consensus classifier (classify-consensus-blast) and the Naive Bayes classifiers used in the comparison)
- **Nextflow** (Workflow engine that runs the MiCoNE pipeline template and parallelizes the TA step across samples) — https://micone.readthedocs.io/
- **Python** (MiCoNE library used to load assignment tables and compute the taxonomic similarity matrix) — https://github.com/segrelab/MiCoNE

## Examples

```
micone init -w ta_op_ni -o pipeline && bash run.sh  # with nextflow.config assign: BLAST, references: ncbi_refseq, max_accepts 10, perc_identity 0.8, evalue 0.001, min_consensus 0.51
```

## Evaluation signals

- Reproduced pairwise similarities match Fig. 3B: every database pair (GG vs SILVA vs NCBI RefSeq) shows below-order-level (Family, Genus, Species) similarity under 45%
- Phylum- and class-level similarity between the three databases remains higher than the below-order levels
- taxonomic_similarity_matrix.csv contains exactly one row per database pair (3 pairs) per taxonomic level (6 levels) = 18 rows
- BLAST run uses the manuscript parameters: max_accepts 10, perc_identity 0.8, evalue 0.001, min_consensus 0.51; Naive Bayes runs use confidence 0.7 and classifier gg_13_8_99_515_806
- Assignment tables are in qiime2-supported formats and rankable by abundance for top-100 extraction

## Limitations

- Assignments from different reference databases diverge sharply: similarity drops below 45% at all taxonomic levels below order, so database choice strongly affects downstream composition inference
- MiCoNE's data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556 before pipeline execution
- The BLAST arm uses a lower perc_identity (0.8) with consensus over 10 accepts, so its labels are not directly comparable string-for-string with confidence-0.7 Naive Bayes labels without harmonization

## Evidence

- [methods] Run the BLAST-based assignment against the ncbi_refseq reference with max_accepts 10, perc_identity 0.8, evalue 0.001, and min_consensus 0.51.: [evidence span withheld — anchor status: not_found]
- [results] Pairwise similarity between the three reference databases' assignments drops below 45% at all taxonomic levels below order: [evidence span withheld — anchor status: not_found]
- [readme] The Microbial Co-occurrence Network Explorer, or MiCoNE (available at https://github.com/segrelab/MiCoNE) follows these default tools and parameters: "The Microbial Co-occurrence Network Explorer, or MiCoNE (available at https://github.com/segrelab/MiCoNE) follows these default tools and parameters"
- [methods] | Assign | BLAST       | references    | "ncbi_refseq"        |: "| Assign | BLAST       | references    | "ncbi_refseq"        |"
