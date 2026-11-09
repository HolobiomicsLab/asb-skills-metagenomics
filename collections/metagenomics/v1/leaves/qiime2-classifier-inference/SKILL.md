---
name: qiime2-classifier-inference
description: Use when raw or denoised 16S rRNA amplicon representative sequences need
  taxonomic assignment under multiple reference databases (Greengenes, SILVA, NCBI
  RefSeq) to compare predicted vs. expected taxonomic compositions, e.g. on mock community
  benchmark data from Zenodo record 7051556.
when_to_use_negative:
- Input is already a feature table with taxonomy assignments rather than representative
  sequences
- The goal is co-occurrence network inference (correlation/SpiecEsi steps), not taxonomic
  assignment accuracy assessment
- Data are shotgun metagenomic reads rather than 16S rRNA amplicon sequences
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MiCoNE
  role: Modular pipeline that wraps the taxonomy assignment step and provides the
    Group step for collapsing abundances across tax_levels
  repo: https://github.com/segrelab/MiCoNE
- name: QIIME2
  role: Runs the Naive Bayes taxonomy classifier (gg_13_8_99_515_806, confidence 0.7)
    via the micone-qiime2 environment
- name: BLAST
  role: Alternative assignment tool against ncbi_refseq with max_accepts=10, perc_identity=0.8,
    evalue=0.001, min_consensus=0.51
- name: Python
  role: Used to compute Bray-Curtis dissimilarity between predicted and expected taxonomic
    distributions and write bray_curtis_dissimilarity.csv
  repo: https://github.com/segrelab/MiCoNE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:affa872083918967f621c4933170c51c80e9d80749958c5984a644504fb2f344
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - BLAST
  - MiCoNE
  - Python
  - QIIME2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# qiime2-classifier-inference

## Summary

Assign taxonomy to 16S amplicon representative sequences within the MiCoNE pipeline using the QIIME2 Naive Bayes classifier (Greengenes gg_13_8_99_515_806, confidence 0.7) and BLAST against ncbi_refseq, producing predicted taxonomic distributions that can be assessed against mock-community expectations via Bray-Curtis dissimilarity.

## When to use

Use when raw or denoised 16S rRNA amplicon representative sequences need taxonomic assignment under multiple reference databases (Greengenes, SILVA, NCBI RefSeq) to compare predicted vs. expected taxonomic compositions, e.g. on mock community benchmark data from Zenodo record 7051556.

## When NOT to use

- Input is already a feature table with taxonomy assignments rather than representative sequences
- The goal is co-occurrence network inference (correlation/SpiecEsi steps), not taxonomic assignment accuracy assessment
- Data are shotgun metagenomic reads rather than 16S rRNA amplicon sequences

## Inputs

- Representative sequences from the mock-dataset (MiCoNE benchmark data, Zenodo record 7051556 data.zip)
- QIIME2-format feature/representative sequence files
- Greengenes gg_13_8_99_515_806 Naive Bayes classifier
- ncbi_refseq BLAST reference database

## Outputs

- Predicted taxonomic abundance distributions per sample for each database (Greengenes, SILVA, NCBI RefSeq)
- Grouped abundance tables at Phylum, Class, Order, Family, Genus, Species levels (MiCoNE Group step)
- bray_curtis_dissimilarity.csv with dissimilarity values in [0, 1] per database and taxonomic level

## How to apply

Download the MiCoNE benchmark data from Zenodo record 7051556 (data.zip) and place it in the nf_micone/data directory; representative-sequence files must follow the qiime2 supported formats. Run taxonomy assignment with the Naive Bayes classifier 'gg_13_8_99_515_806' (Greengenes 13_8 99% 515/806-region classifier, confidence = 0.7) inside the micone-qiime2 conda environment, and with BLAST against 'ncbi_refseq' (max_accepts = 10, perc_identity = 0.8, evalue = 0.001, min_consensus = 0.51). Group the predicted abundances at the tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species'] using the MiCoNE Group step. The rationale, per the paper's benchmark, is that no database is absolutely better: all three show high Bray-Curtis dissimilarity against expected mock-community composition (> 0.5 for Family and Genus, > 0.9 for Species), so assignments from each database must be evaluated comparatively rather than trusted individually.

## Related tools

- **MiCoNE** (Modular pipeline that wraps the taxonomy assignment step and provides the Group step for collapsing abundances across tax_levels) — https://github.com/segrelab/MiCoNE
- **QIIME2** (Runs the Naive Bayes taxonomy classifier (gg_13_8_99_515_806, confidence 0.7) via the micone-qiime2 environment)
- **BLAST** (Alternative assignment tool against ncbi_refseq with max_accepts=10, perc_identity=0.8, evalue=0.001, min_consensus=0.51)
- **Python** (Used to compute Bray-Curtis dissimilarity between predicted and expected taxonomic distributions and write bray_curtis_dissimilarity.csv) — https://github.com/segrelab/MiCoNE

## Examples

```
micone install -e "micone-qiime2"
```

## Evaluation signals

- bray_curtis_dissimilarity.csv exists and contains a dissimilarity column with all values in [0, 1]
- Values reproduce the Fig. 3C pattern: dissimilarity > 0.5 for Family and Genus and > 0.9 for Species for all databases
- No database is strictly best — dissimilarities should be comparable in magnitude across Greengenes, SILVA, and NCBI RefSeq
- Grouped abundance tables contain rows for all requested tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']
- Any value outside [0, 1] or a database strictly outperforming others should trigger a flag for expert review

## Limitations

- No taxonomy database (Greengenes, SILVA, NCBI RefSeq) performs better than the others in absolute terms; residual dissimilarity remains high (> 0.5 for Family/Genus, > 0.9 for Species)
- The data directory (nf_micone/data) must be manually downloaded from Zenodo before the pipeline can run
- MiCoNE requires the mamba package manager; otherwise micone init will not work, and preinstalled julia versions can conflict with the micone-flashweave environment

## Evidence

- [methods] | Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |: "| Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |"
- [methods] | Assign | BLAST       | references    | "ncbi_refseq"        |: "| Assign | BLAST       | references    | "ncbi_refseq"        |"
- [methods] | Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |: "| Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |"
- [methods] These files must follow the ``qiime2`` supported formats: "These files must follow the ``qiime2`` supported formats"
- [readme] The data directory (`nf_micone/data`) needs to be manually downloaded using this [link](https://zenodo.org/record/7051556/files/data.zip?download=1).: "The data directory (`nf_micone/data`) needs to be manually downloaded using this [link](https://zenodo.org/record/7051556/files/data.zip?download=1)."
