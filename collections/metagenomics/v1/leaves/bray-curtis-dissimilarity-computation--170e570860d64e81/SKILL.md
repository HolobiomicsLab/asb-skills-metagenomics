---
name: bray-curtis-dissimilarity-computation
description: Use when mock-community 16S data have been taxonomically assigned with
  alternative reference databases/classifiers and you need a single [0,1] dissimilarity
  metric per taxonomic level to compare assignment accuracy against a known expected
  composition.
when_to_use_negative:
- Input is already a co-occurrence network or interaction matrix — MiCoNE handles
  those separately via Network objects, not via taxonomy-accuracy dissimilarity
- There is no known expected composition (real environmental samples), so no reference
  distribution exists to compare against
- The goal is network-inference step comparison (correlation/direct methods) rather
  than taxonomy assignment accuracy
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MiCoNE
  role: Pipeline framework providing the Group step that aggregates predicted abundances
    at tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']
  repo: https://github.com/segrelab/MiCoNE
- name: QIIME2
  role: Environment for running Naive Bayes taxonomy assignment with the gg_13_8_99_515_806
    classifier (confidence = 0.7)
- name: BLAST
  role: Alternative taxonomy assignment against ncbi_refseq with max_accepts = 10,
    perc_identity = 0.8, evalue = 0.001, min_consensus = 0.51
- name: Python
  role: Language used to compute Bray-Curtis dissimilarity between predicted and expected
    taxonomic distributions and write bray_curtis_dissimilarity.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:2a4cc49ac1c6ec8c838a70f89c03d498b08c915e982414651c641750e113f33d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - BLAST
  - MiCoNE
  - Python
  - QIIME2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bray-curtis-dissimilarity-computation

## Summary

A validation-oriented skill for quantifying how accurately 16S taxonomy assignment recaptures expected mock-community composition, by computing Bray-Curtis dissimilarity between predicted and expected taxonomic distributions at Family, Genus, and Species levels (as used in the MiCoNE benchmarking of Greengenes, SILVA, and NCBI RefSeq).

## When to use

Use when mock-community 16S data have been taxonomically assigned with alternative reference databases/classifiers and you need a single [0,1] dissimilarity metric per taxonomic level to compare assignment accuracy against a known expected composition.

## When NOT to use

- Input is already a co-occurrence network or interaction matrix — MiCoNE handles those separately via Network objects, not via taxonomy-accuracy dissimilarity
- There is no known expected composition (real environmental samples), so no reference distribution exists to compare against
- The goal is network-inference step comparison (correlation/direct methods) rather than taxonomy assignment accuracy

## Inputs

- Mock-dataset representative 16S sequences from MiCoNE benchmark data (Zenodo record 7051556, data.zip)
- Expected taxonomic composition tables for the mock communities
- Predicted taxonomic abundance distributions from Naive Bayes (gg_13_8_99_515_806, confidence 0.7) and BLAST (ncbi_refseq) assignment

## Outputs

- bray_curtis_dissimilarity.csv containing a dissimilarity column with values in [0, 1] per database and taxonomic level (Family, Genus, Species)

## How to apply

Download the MiCoNE benchmark data from Zenodo record 7051556 (data.zip) and load the mock-dataset representative sequences together with their expected taxonomic compositions. Assign taxonomy in the QIIME2 environment using the Naive Bayes classifier with the Greengenes 'gg_13_8_99_515_806' classifier at confidence = 0.7, and with BLAST against 'ncbi_refseq' (max_accepts = 10, perc_identity = 0.8, evalue = 0.001, min_consensus = 0.51). Group predicted abundances at the taxonomic levels 'Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species' via the MiCoNE Group step, then compute Bray-Curtis dissimilarity between predicted and expected distributions at Family, Genus, and Species levels for each database and write the values to bray_curtis_dissimilarity.csv. The rationale is that dissimilarity against known expected composition gives a direct, interpretable accuracy measure: in the source study it exceeded 0.5 for Family and Genus and 0.9 for Species for all databases, with no single database strictly best.

## Related tools

- **MiCoNE** (Pipeline framework providing the Group step that aggregates predicted abundances at tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']) — https://github.com/segrelab/MiCoNE
- **QIIME2** (Environment for running Naive Bayes taxonomy assignment with the gg_13_8_99_515_806 classifier (confidence = 0.7))
- **BLAST** (Alternative taxonomy assignment against ncbi_refseq with max_accepts = 10, perc_identity = 0.8, evalue = 0.001, min_consensus = 0.51)
- **Python** (Language used to compute Bray-Curtis dissimilarity between predicted and expected taxonomic distributions and write bray_curtis_dissimilarity.csv)

## Examples

```
from scipy.spatial.distance import braycurtis; d = braycurtis(predicted_grouped, expected_grouped)  # per database at Family/Genus/Species, saved to bray_curtis_dissimilarity.csv
```

## Evaluation signals

- bray_curtis_dissimilarity.csv exists and contains a dissimilarity column with all values within [0, 1]
- Family- and Genus-level dissimilarities exceed 0.5 for all three databases, reproducing the Fig. 3C pattern
- Species-level dissimilarities exceed 0.9 for all three databases
- No taxonomy database is strictly best in absolute terms across levels; if one dominates everywhere, flag for expert review
- Predicted abundances are grouped to exactly the six tax_levels supported by the MiCoNE Group step before dissimilarity computation

## Limitations

- Residual dissimilarity is high even for the best-performing database (> 0.5 at Family/Genus, > 0.9 at Species), so the metric cannot crown a single 'best' database in absolute terms
- Species-level comparisons are least reliable, consistent with dissimilarity > 0.9 for all databases
- The pipeline's data directory (nf_micone/data) must be manually downloaded from Zenodo; missing data files will stall the workflow
- MiCoNE requires mamba; 'micone init' will not work without it

## Evidence

- [readme] Download the data directory from `here <https://zenodo.org/record/7051556/files/data.zip?download=1>`__: "Download the data directory from `here <https://zenodo.org/record/7051556/files/data.zip?download=1>`__"
- [methods] | Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |: "| Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |"
- [methods] | Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |: "| Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |"
- [results] No taxonomy database performs better than the others in absolute terms; the Bray-Curtis dissimilarity with the expected mock-community composition is high for all three (> 0.5 for Family and Genus, > 0.9 for Species).: [evidence span withheld — anchor status: not_found]
- [abstract] develop consensus network algorithms based on benchmarks with mock and synthetic data sets: "develop consensus network algorithms based on benchmarks with mock and synthetic data sets"
