---
name: otu-table-qc-and-formatting
description: Use when raw 16S amplicon data (e.g., mockrobiota mock4, mock12, mock16)
  must be converted into per-method feature tables that are QC-checked, formatted
  for qiime2, and compared to true community compositions via weighted UniFrac.
when_to_use_negative:
- The input is already a validated qiime2 feature table and the goal is downstream
  co-occurrence network inference (correlation, spieceasi, etc.) rather than DC-method
  benchmarking
- The data are the synthetic benchmark networks from MiCoNE-synthetic-data rather
  than real 16S amplicon reads from mock communities
- Only a single DC method is being applied for exploratory analysis with no need for
  comparison against expected mock-community compositions
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: MiCoNE
  role: Runs the five DC methods (closed-reference, open-reference, de novo, DADA2,
    Deblur) with configured parameters and consensus chimera removal to produce feature
    tables
  repo: https://github.com/segrelab/MiCoNE
- name: qiime2
  role: Consumes the qiime2-format feature tables and computes weighted UniFrac distances
    on the Greengenes phylogeny
- name: DADA2
  role: Denoising method whose default selection was justified by best weighted UniFrac
    performance on all mock datasets
- name: Deblur
  role: Alternative deblur DC method (ncpus 1, min_reads 2, min_size 2) benchmarked
    against DADA2
- name: Python
  role: Hosts the MiCoNE package and scikit-bio UniFrac implementation used to compute
    and assemble distance tables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:e12bf82225d826f3271ba907ecb2d51ac23f8b063df37eb82513873b099aa36b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - DADA2
  - Deblur
  - MiCoNE
  - Python
  - qiime2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# otu-table-qc-and-formatting

## Summary

Extracts and quality-checks predicted representative-sequence (feature) tables produced by the five MiCoNE denoising/clustering (DC) methods, formats them as qiime2-compatible artifacts, and assembles weighted UniFrac distances against true mockrobiota distributions into a tidy CSV for method benchmarking.

## When to use

Use when raw 16S amplicon data (e.g., mockrobiota mock4, mock12, mock16) must be converted into per-method feature tables that are QC-checked, formatted for qiime2, and compared to true community compositions via weighted UniFrac.

## When NOT to use

- The input is already a validated qiime2 feature table and the goal is downstream co-occurrence network inference (correlation, spieceasi, etc.) rather than DC-method benchmarking
- The data are the synthetic benchmark networks from MiCoNE-synthetic-data rather than real 16S amplicon reads from mock communities
- Only a single DC method is being applied for exploratory analysis with no need for comparison against expected mock-community compositions

## Inputs

- Mockrobiota mock4, mock12, and mock16 16S datasets from the MiCoNE data directory (Zenodo record 7051556) and the MiCoNE-synthetic-data repository
- gg_97 reference sequences for closed/open-reference OTU assignment
- Greengenes phylogeny for weighted UniFrac
- MiCoNE pipeline config files (nextflow.config) specifying DC tool selections

## Outputs

- Per method x mock combination representative-sequence (feature) tables in qiime2-compatible format
- unifrac_distances.csv — tidy table of weighted UniFrac distances with columns for mock set and DC method

## How to apply

Run each of the five DC methods in MiCoNE using the config-table defaults: closed-reference and open-reference OTU assignment at percent_identity 0.97 against "gg_97" reference sequences, de novo at 0.97, DADA2 with ncpus 1 and big_data "FALSE", and Deblur with ncpus 1, min_reads 2, min_size 2 — each followed by chimera removal with chimera_method "consensus". Extract the predicted representative-sequence (feature) table for every method x mock combination, ensuring the output follows qiime2 supported formats. Compute the weighted UniFrac distance between each predicted sequence distribution and the expected (true) mockrobiota distribution using QIIME2 or the scikit-bio UniFrac implementation on the Greengenes phylogeny. Assemble the distances into a tidy table with columns for mock set and DC method and write it to unifrac_distances.csv. The rationale is that different tools lead to vastly different inferences of community composition, so a standardized, fully specified DC configuration is required for fair comparison.

## Related tools

- **MiCoNE** (Runs the five DC methods (closed-reference, open-reference, de novo, DADA2, Deblur) with configured parameters and consensus chimera removal to produce feature tables) — https://github.com/segrelab/MiCoNE
- **qiime2** (Consumes the qiime2-format feature tables and computes weighted UniFrac distances on the Greengenes phylogeny)
- **DADA2** (Denoising method whose default selection was justified by best weighted UniFrac performance on all mock datasets)
- **Deblur** (Alternative deblur DC method (ncpus 1, min_reads 2, min_size 2) benchmarked against DADA2)
- **Python** (Hosts the MiCoNE package and scikit-bio UniFrac implementation used to compute and assemble distance tables)

## Examples

```
micone init -w full -o pipeline_dir; bash run.sh
```

## Evaluation signals

- All five DC method x three mock combinations yield a feature table that loads as a qiime2-compatible artifact without format errors
- Chimera removal with chimera_method "consensus" is applied after every DC method before table extraction
- unifrac_distances.csv is a tidy table with exactly the columns mock set and DC method and 15 rows (5 methods x 3 mocks)
- Validation: DADA2 exhibits the lowest weighted UniFrac distance to the expected distribution across mock4, mock12, and mock16, matching Fig. 2C/D
- Config values match the MiCoNE defaults: percent_identity 0.97, reference_sequences "gg_97", DADA2 ncpus 1 / big_data "FALSE", Deblur min_reads 2 / min_size 2

## Limitations

- No single DC method outperforms the rest in all data sets — DADA2 won on weighted UniFrac for the mocks, but method choice should not be assumed universally optimal
- The MiCoNE data directory (nf_micone/data) is not bundled with the package and must be manually downloaded from Zenodo record 7051556
- Different DC tools lead to vastly different inferences of community composition, so comparisons are only fair when chimera removal and all other parameters are standardized

## Evidence

- [methods] These files must follow the ``qiime2`` supported formats: "These files must follow the ``qiime2`` supported formats"
- [methods] Chimera checking default: "| Chimera checking | Remove bimera    | chimera_method      | "consensus" |"
- [results] DADA2-best finding from the source card reproduced against Fig. 2C/D: [evidence span withheld — anchor status: not_found]
- [readme] README on manually downloaded data directory: "The data directory (`nf_micone/data`) needs to be manually downloaded using this [link](https://zenodo.org/record/7051556/files/data.zip?download=1)."
- [intro] Intro motivation for standardized DC comparison: "The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks"
