---
name: representative-sequence-distribution-comparison
description: Use when comparing 16S DC methods (DADA2, Deblur, de novo, closed/open-reference)
  by measuring how accurately each predicted feature table recapitulates a known mock-community
  composition via weighted UniFrac.
when_to_use_negative:
- Input is already a co-occurrence network or interaction matrix — this skill operates
  on feature/representative-sequence tables, not network objects
- The true (expected) community composition is unknown — weighted UniFrac comparison
  to a ground truth cannot be computed
- Comparing methods on natural environmental samples where no expected distribution
  exists
edam_operation: http://edamontology.org/operation_0246
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MiCoNE
  role: Runs the five DC methods (closed/open-reference, de novo OTU assignment at
    percent_identity 0.97 with reference_sequences gg_97, DADA2, Deblur) with consensus
    chimera removal to produce feature tables
  repo: https://github.com/segrelab/MiCoNE
- name: DADA2
  role: Denoising method benchmarked; best performer on the weighted UniFrac metric
    across mock datasets and set as default DC method
- name: Deblur
  role: Denoising method benchmarked against DADA2 on the weighted UniFrac metric
- name: qiime2
  role: Provides feature-table format and weighted UniFrac computation on the Greengenes
    phylogeny
- name: Python
  role: Implementation language for MiCoNE and for assembling the unifrac_distances.csv
    tidy table
  repo: https://github.com/segrelab/MiCoNE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:c350bd39a9aa0de4e615760318c7b850ebafe62e8b7e3ff34a62b3ca1ab1fb2b
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
  edam_operation: http://edamontology.org/operation_0246
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# representative-sequence-distribution-comparison

## Summary

Benchmark denoising/clustering (DC) methods by computing the weighted UniFrac distance between predicted representative-sequence (feature) distributions and the expected mockrobiota distributions on the Greengenes phylogeny, to identify which DC method best recapitulates true mock-community sequences.

## When to use

Use when comparing 16S DC methods (DADA2, Deblur, de novo, closed/open-reference) by measuring how accurately each predicted feature table recapitulates a known mock-community composition via weighted UniFrac.

## When NOT to use

- Input is already a co-occurrence network or interaction matrix — this skill operates on feature/representative-sequence tables, not network objects
- The true (expected) community composition is unknown — weighted UniFrac comparison to a ground truth cannot be computed
- Comparing methods on natural environmental samples where no expected distribution exists

## Inputs

- mockrobiota mock4, mock12, and mock16 16S datasets (Zenodo record 7051556 and MiCoNE-synthetic-data repository)
- Expected (true) mock-community sequence distributions
- Greengenes gg_97 phylogeny/reference sequences
- Predicted representative-sequence (feature) tables from the five DC methods in qiime2-compatible format

## Outputs

- unifrac_distances.csv — tidy table of weighted UniFrac distances with columns for mock set and DC method
- Method ranking (which DC method has the lowest weighted UniFrac distance to the expected distribution)

## How to apply

Obtain the mockrobiota mock4, mock12, and mock16 16S datasets from the MiCoNE data directory on Zenodo (record 7051556) and run each of the five DC methods with MiCoNE defaults: closed-reference and open-reference OTU assignment at percent_identity 0.97 against gg_97, de novo at 0.97, DADA2 (ncpus 1, big_data FALSE), and Deblur (ncpus 1, min_reads 2, min_size 2), each followed by consensus chimera removal. Extract each predicted representative-sequence (feature) table in qiime2-compatible format and compute the weighted UniFrac distance between the predicted sequence distribution and the expected (true) mock distribution using QIIME2 or scikit-bio on the Greengenes phylogeny. Assemble the distances into a tidy table (mock set x DC method) and write it to unifrac_distances.csv.

## Related tools

- **MiCoNE** (Runs the five DC methods (closed/open-reference, de novo OTU assignment at percent_identity 0.97 with reference_sequences gg_97, DADA2, Deblur) with consensus chimera removal to produce feature tables) — https://github.com/segrelab/MiCoNE
- **DADA2** (Denoising method benchmarked; best performer on the weighted UniFrac metric across mock datasets and set as default DC method)
- **Deblur** (Denoising method benchmarked against DADA2 on the weighted UniFrac metric)
- **qiime2** (Provides feature-table format and weighted UniFrac computation on the Greengenes phylogeny)
- **Python** (Implementation language for MiCoNE and for assembling the unifrac_distances.csv tidy table) — https://github.com/segrelab/MiCoNE

## Examples

```
micone init -w full -o pipeline_dir
```

## Evaluation signals

- DADA2 exhibits the lowest weighted UniFrac distance to the expected distribution across mock4, mock12, and mock16, matching Fig. 2C/D of the paper
- unifrac_distances.csv has tidy columns for mock set and DC method with a weighted UniFrac value for each of the 3 mocks x 5 methods
- Each DC run used the documented MiCoNE config values (percent_identity 0.97 with gg_97 for reference-based methods; chimera_method "consensus"; Deblur min_reads 2, min_size 2)
- Feature tables are in qiime2-compatible format and are placed on the Greengenes phylogeny before UniFrac computation
- All five DC methods (closed-reference, open-reference, de novo, DADA2, Deblur) are represented in the comparison, not just a subset

## Limitations

- No single DC method outperforms the rest in all data sets — DADA2's superiority on weighted UniFrac does not generalize to every dataset
- The benchmark is specific to mockrobiota mock communities (mock4, mock12, mock16) and the Greengenes phylogeny; results may not transfer to environmental samples or other reference phylogenies
- MiCoNE requires the mamba package manager or micone init will not work, and the data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556

## Evidence

- [results] DADA2, compared to Deblur, displays better performance on all the mock data sets on the weighted UniFrac metric: [evidence span withheld — anchor status: not_found]
- [results] no single method outperforms the rest in all data sets: [evidence span withheld — anchor status: not_found]
- [methods] OTU assignment Closed reference percent_identity 0.97: "| OTU assignment   | Closed reference | percent_identity    | 0.97        |"
- [methods] Chimera checking Remove bimera chimera_method consensus: "| Chimera checking | Remove bimera    | chimera_method      | "consensus" |"
- [readme] The Microbial Co-occurrence Network Explorer, or MiCoNE (available at https://github.com/segrelab/MiCoNE) follows these default tools and parameters: "The Microbial Co-occurrence Network Explorer, or MiCoNE (available at https://github.com/segrelab/MiCoNE) follows these default tools and parameters"
- [readme] Zenodo record 7051556 data directory: "The data directory (`nf_micone/data`) needs to be manually downloaded using this [link](https://zenodo.org/record/7051556/files/data.zip?download=1)."
