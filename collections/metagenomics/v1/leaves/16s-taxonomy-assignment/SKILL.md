---
name: 16s-taxonomy-assignment
description: Reference for assigning taxonomy to 16S rRNA representative sequences
  when comparing multiple reference databases (e.g., Greengenes vs NCBI RefSeq) or
  when predicted taxonomic abundances are needed for mock-community accuracy assessment.
when_to_use_negative:
- Input is already a taxonomy-annotated feature table or abundance profile — assignment
  has already been performed and re-running classifiers would not change results.
- The goal is de novo taxonomic discovery of novel lineages — closed-reference Naive
  Bayes/BLAST assignment against Greengenes/RefSeq only classifies sequences the references
  can explain.
- You need a single canonical taxonomy for downstream analysis and cannot afford to
  run multiple databases — this skill is explicitly for comparing database-dependent
  outputs, since no database performed better than the others in absolute terms.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
tools:
- name: MiCoNE
  role: Pipeline framework that orchestrates the taxonomy assignment (Assign) and
    Group steps with configurable tools and parameters
  repo: https://github.com/segrelab/MiCoNE
- name: QIIME2
  role: Hosts the Naive Bayes taxonomy classifier (gg_13_8_99_515_806, confidence
    0.7) used for one arm of assignment
- name: BLAST
  role: Alternative assigner against the ncbi_refseq reference with max_accepts=10,
    perc_identity=0.8, evalue=0.001, min_consensus=0.51
- name: Python
  role: Implementation language of MiCoNE; used to compute Bray-Curtis dissimilarity
    between predicted and expected grouped abundances
  repo: https://github.com/segrelab/MiCoNE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:03685037238e47b8c05bd78ae5cff0c01adfdb25e3457b61a46da16651ebbfcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - BLAST
  - MiCoNE
  - Python
  - QIIME2
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# 16s-taxonomy-assignment

## Summary

Assigns taxonomy to 16S amplicon representative sequences using parallel taxonomic classifiers (QIIME2 Naive Bayes with the Greengenes 'gg_13_8_99_515_806' classifier and BLAST against 'ncbi_refseq') as implemented in the MiCoNE pipeline, enabling comparison of how taxonomy database choice affects downstream taxonomic abundance profiles.

## When to use

Reference for assigning taxonomy to 16S rRNA representative sequences when comparing multiple reference databases (e.g., Greengenes vs NCBI RefSeq) or when predicted taxonomic abundances are needed for mock-community accuracy assessment.

## When NOT to use

- Input is already a taxonomy-annotated feature table or abundance profile — assignment has already been performed and re-running classifiers would not change results.
- The goal is de novo taxonomic discovery of novel lineages — closed-reference Naive Bayes/BLAST assignment against Greengenes/RefSeq only classifies sequences the references can explain.
- You need a single canonical taxonomy for downstream analysis and cannot afford to run multiple databases — this skill is explicitly for comparing database-dependent outputs, since no database performed better than the others in absolute terms.

## Inputs

- Representative sequences of mock-dataset 16S amplicons (qiime2-supported format files)
- MiCoNE benchmark data from Zenodo record 7051556 (data.zip) including expected mock-community taxonomic compositions
- Reference taxonomy data: Greengenes 'gg_13_8_99_515_806' classifier and 'ncbi_refseq' reference sequences (bundled in nf_micone/data)
- MiCoNE nextflow.config pipeline configuration

## Outputs

- Predicted taxonomic abundance tables per database and sample
- Abundances grouped at tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']
- bray_curtis_dissimilarity.csv with predicted-vs-expected Bray-Curtis dissimilarity values per database and taxonomic level

## How to apply

Within the MiCoNE pipeline (QIIME2 environment installed via 'micone install -e micone-qiime2'), assign taxonomy to representative sequences using the Naive Bayes classifier with the Greengenes 'gg_13_8_99_515_806' classifier at confidence = 0.7, and in parallel run BLAST against 'ncbi_refseq' with max_accepts = 10, perc_identity = 0.8, evalue = 0.001, and min_consensus = 0.51. Input files must follow the qiime2 supported formats; the data directory must be manually downloaded from Zenodo record 7051556 (data.zip). After assignment, use the MiCoNE Group step to collapse predicted abundances at the taxonomic levels 'Phylum', 'Class', 'Order', 'Family', 'Genus', and 'Species'. The rationale, from the source study, is that different databases yield different predicted compositions, so the assignment step is run for each database to quantify which best recaptures expected mock-community composition via Bray-Curtis dissimilarity (none was strictly best; dissimilarity exceeded 0.5 at Family/Genus and 0.9 at Species).

## Related tools

- **MiCoNE** (Pipeline framework that orchestrates the taxonomy assignment (Assign) and Group steps with configurable tools and parameters) — https://github.com/segrelab/MiCoNE
- **QIIME2** (Hosts the Naive Bayes taxonomy classifier (gg_13_8_99_515_806, confidence 0.7) used for one arm of assignment)
- **BLAST** (Alternative assigner against the ncbi_refseq reference with max_accepts=10, perc_identity=0.8, evalue=0.001, min_consensus=0.51)
- **Python** (Implementation language of MiCoNE; used to compute Bray-Curtis dissimilarity between predicted and expected grouped abundances) — https://github.com/segrelab/MiCoNE

## Examples

```
micone install -e "micone-qiime2"
```

## Evaluation signals

- bray_curtis_dissimilarity.csv exists and contains a dissimilarity column with all values in [0, 1]
- Reproduced dissimilarity matches the reported Fig. 3C pattern: values > 0.5 at Family and Genus levels and > 0.9 at Species level for all three databases
- No database is strictly best across all taxonomic levels — a run showing one database uniformly lowest at every level should be flagged for expert review
- Assignment output files conform to the qiime2 supported formats expected by the MiCoNE Assign step
- Grouped abundances cover exactly the tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species'] configured in the MiCoNE Group step

## Limitations

- All three taxonomy databases show high Bray-Curtis dissimilarity to expected mock-community composition (> 0.5 Family/Genus, > 0.9 Species), so assignment accuracy at Species level is poor regardless of database choice.
- The MiCoNE data directory (nf_micone/data) is not installed automatically and must be manually downloaded from Zenodo record 7051556.
- MiCoNE requires the mamba package manager for environment creation; 'micone init' will not work otherwise.

## Evidence

- [methods] Naive Bayes classifier uses Greengenes 99% OTU reference trained on 515-806 region: "| Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |"
- [methods] BLAST assignment against NCBI RefSeq reference: "| Assign | BLAST       | references    | "ncbi_refseq"        |"
- [methods] Group step collapses abundances at all six taxonomic levels: "| Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |"
- [methods] Assignment inputs must be in QIIME2 format: "These files must follow the ``qiime2`` supported formats"
- [readme] Benchmark data source for the mock-community assessment: "Data related to the publication can be found on Zenodo: https://doi.org/10.5281/zenodo.7051556."
- [readme] Data directory must be manually downloaded: "The data directory (`nf_micone/data`) needs to be manually downloaded using this [link](https://zenodo.org/record/7051556/files/data.zip?download=1)."
