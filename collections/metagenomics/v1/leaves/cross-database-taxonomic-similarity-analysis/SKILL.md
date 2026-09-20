---
name: cross-database-taxonomic-similarity-analysis
description: Use when the same OTU representative sequences have been taxonomically
  assigned with multiple reference databases and you need to quantify assignment concordance
  per taxonomic level (Phylum through Species).
when_to_use_negative:
- You only have a single taxonomic assignment (one reference database) — there is
  no second table to compare, so pairwise similarity is undefined
- Your goal is OTU picking or denoising rather than evaluating assignment concordance
  — use the MiCoNE OTU assignment (dada2/deblur/closed-reference gg_97) step instead
- Your sequences are shotgun metagenomic reads rather than 16S amplicon representatives
  — the MiCoNE TA step and this comparison are built for 16S data
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MiCoNE
  role: Python package executing the taxonomy assignment step and providing the grouping/similarity
    utilities used to compute per-level pairwise similarity
  repo: https://github.com/segrelab/MiCoNE
- name: QIIME2
  role: Hosts the Naive Bayes classifier used for Greengenes (gg_13_8_99_515_806)
    and SILVA assignments, and the BLAST-based assigner for ncbi_refseq
- name: Nextflow
  role: Runs the micone workflow template (via micone init and run.sh) that parallelizes
    the three TA runs reproducibly
- name: Python
  role: Used to rank the top-100 taxa by abundance and assemble taxonomic_similarity_matrix.csv
    from MiCoNE grouping utilities
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:862e2e49809236383792418f109ddb177553a1c0b57bb6f2ae2ae6082d810f74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Nextflow
  - Python
  - QIIME2
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-database-taxonomic-similarity-analysis

## Summary

Quantifies how much taxonomic assignments of the same OTU representative sequences disagree when three 16S reference databases (Greengenes, SILVA, NCBI RefSeq) are used, by computing pairwise similarity of the top-100 most abundant taxa at each taxonomic level. It exposes reference-database choice as a major variance source in 16S pipelines such as MiCoNE.

## When to use

Use when the same OTU representative sequences have been taxonomically assigned with multiple reference databases and you need to quantify assignment concordance per taxonomic level (Phylum through Species).

## When NOT to use

- You only have a single taxonomic assignment (one reference database) — there is no second table to compare, so pairwise similarity is undefined
- Your goal is OTU picking or denoising rather than evaluating assignment concordance — use the MiCoNE OTU assignment (dada2/deblur/closed-reference gg_97) step instead
- Your sequences are shotgun metagenomic reads rather than 16S amplicon representatives — the MiCoNE TA step and this comparison are built for 16S data

## Inputs

- OTU representative sequences from the MiCoNE default DC configuration
- Greengenes gg_13_8_99_515_806 99% Naive Bayes classifier (qiime2 format)
- SILVA-trained QIIME2 Naive Bayes classifier
- ncbi_refseq reference sequences for BLAST-based assignment
- Abundance (taxonomy) tables produced by each of the three assignment runs

## Outputs

- taxonomic_similarity_matrix.csv with one row per database pair (GG vs SILVA, GG vs RefSeq, SILVA vs RefSeq) and taxonomic level (Phylum, Class, Order, Family, Genus, Species)
- Top-100 abundance-ranked taxonomy tables per reference database

## How to apply

Run the MiCoNE taxonomy assignment (TA) step on the OTU representative sequences from the default DC configuration three times: (1) QIIME2 Naive Bayes classifier trained on Greengenes gg_13_8_99_515_806 with confidence 0.7 and ncpus 1, (2) a SILVA-trained Naive Bayes classifier at the same confidence, and (3) BLAST against ncbi_refseq with max_accepts 10, perc_identity 0.8, evalue 0.001, and min_consensus 0.51. Rank taxa in each assignment table by abundance and retain the top-100 taxa, then group them at the six tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species'] using MiCoNE's Group transform and grouping utilities. Compute pairwise similarity between the three database assignments at every level and assemble the values into taxonomic_similarity_matrix.csv with one row per database pair and taxonomic level. The rationale is that different reference databases produce vastly different community-composition inferences, so measuring per-level concordance identifies the levels where database choice dominates pipeline variance.

## Related tools

- **MiCoNE** (Python package executing the taxonomy assignment step and providing the grouping/similarity utilities used to compute per-level pairwise similarity) — https://github.com/segrelab/MiCoNE
- **QIIME2** (Hosts the Naive Bayes classifier used for Greengenes (gg_13_8_99_515_806) and SILVA assignments, and the BLAST-based assigner for ncbi_refseq)
- **Nextflow** (Runs the micone workflow template (via micone init and run.sh) that parallelizes the three TA runs reproducibly)
- **Python** (Used to rank the top-100 taxa by abundance and assemble taxonomic_similarity_matrix.csv from MiCoNE grouping utilities)

## Examples

```
micone init -w ta_op_ni -o ./pipeline_dir && bash run.sh
```

## Evaluation signals

- Validation against Fig. 3B: every database pair shows below-order-level (Family, Genus, Species) similarity under 45%
- Phylum- and class-level similarity should remain substantially higher than the below-order levels, reproducing the paper's reported pattern
- taxonomic_similarity_matrix.csv must contain exactly 3 database pairs x 6 taxonomic levels = 18 rows, all populated
- Each input assignment table must contain exactly 100 abundance-ranked taxa after the top-100 filter
- All three runs must use identical confidence (0.7) or BLAST parameters (max_accepts 10, perc_identity 0.8, evalue 0.001, min_consensus 0.51) so differences are attributable to the reference database alone

## Limitations

- Findings are tied to the top-100 most abundant taxa; conclusions may not generalize to rare taxa
- The three databases use different assignment algorithms (Naive Bayes vs BLAST), so some disagreement reflects method, not just reference content
- The published configuration assumes ncpus 1 and default MiCoNE DC settings; changing these may alter results
- MiCoNE requires the mamba package manager, otherwise micone init will not work, and the data directory must be manually downloaded from Zenodo record 7051556

## Evidence

- [methods] | Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |: "| Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |"
- [methods] | Assign | BLAST       | references    | "ncbi_refseq"        |: "| Assign | BLAST       | references    | "ncbi_refseq"        |"
- [methods] | Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |: "| Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |"
- [intro] The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks: "The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks"
- [results] Pairwise similarity between the three reference databases' assignments drops below 45% at all taxonomic levels below order: [evidence span withheld — anchor status: not_found]
- [readme] MiCoNE is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow: "MiCoNE is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow"
