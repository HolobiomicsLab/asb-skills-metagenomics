---
name: denoising-clustering-method-benchmarking
description: Use when amplicon ASV/OTU inference must be compared across DC tools
  against ground-truth mock community data (e.g., mockrobiota mock4/mock12/mock16)
  using a phylogenetic metric such as weighted UniFrac.
when_to_use_negative:
- Input is already a curated feature/abundance table rather than raw 16S reads requiring
  denoising or clustering
- The goal is co-occurrence network inference rather than evaluating sequence-inference
  accuracy of DC methods
- No ground-truth (expected) community composition is available for the dataset, so
  weighted UniFrac against truth cannot be computed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: MiCoNE
  role: Pipeline framework used to execute each DC method with standardized config
    defaults and consensus chimera removal
  repo: https://github.com/segrelab/MiCoNE
- name: DADA2
  role: Denoising method under benchmark; set as default DC method for best weighted
    UniFrac performance
- name: Deblur
  role: Denoising method under benchmark, run with min_reads 2 and min_size 2
- name: QIIME2
  role: Executes the DC steps and computes weighted UniFrac distances in qiime2-compatible
    formats
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:6c0639193c3b256450ce48af0362740813654b4fbc97eb8660bfaf184e289dc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - DADA2
  - Deblur
  - MiCoNE
  - Python
  - QIIME2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# denoising-clustering-method-benchmarking

## Summary

Benchmarks 16S denoising/clustering (DC) methods — closed-reference, open-reference, and de novo OTU assignment at 0.97 identity, plus DADA2 and Deblur — by measuring weighted UniFrac distance between predicted feature tables and the true (expected) mockrobiota mock-community distributions. Used to identify which DC method best recapitulates known community composition.

## When to use

Use when amplicon ASV/OTU inference must be compared across DC tools against ground-truth mock community data (e.g., mockrobiota mock4/mock12/mock16) using a phylogenetic metric such as weighted UniFrac.

## When NOT to use

- Input is already a curated feature/abundance table rather than raw 16S reads requiring denoising or clustering
- The goal is co-occurrence network inference rather than evaluating sequence-inference accuracy of DC methods
- No ground-truth (expected) community composition is available for the dataset, so weighted UniFrac against truth cannot be computed

## Inputs

- mockrobiota mock4, mock12, and mock16 16S amplicon FASTQ datasets (MiCoNE data directory, Zenodo record 7051556)
- Greengenes gg_97 reference sequences for reference-based OTU assignment
- Expected (true) mock-community representative-sequence distributions
- MiCoNE pipeline config tables (nextflow.config) specifying DC method parameters

## Outputs

- Per method x mock feature (representative-sequence) tables in qiime2-compatible format
- Weighted UniFrac distances between predicted and expected sequence distributions, assembled as unifrac_distances.csv with columns for mock set and DC method

## How to apply

Obtain the mockrobiota mock4, mock12, and mock16 16S datasets from the MiCoNE data directory (Zenodo record 7051556) and the MiCoNE-synthetic-data repository. Run each of the five DC methods with MiCoNE defaults — closed-reference (percent_identity 0.97, reference_sequences "gg_97"), open-reference (percent_identity 0.97, "gg_97"), de novo (percent_identity 0.97), DADA2 (ncpus 1, big_data "FALSE"), and Deblur (ncpus 1, min_reads 2, min_size 2) — each followed by chimera removal with chimera_method "consensus". Extract each predicted representative-sequence (feature) table in qiime2-compatible format and compute the weighted UniFrac distance between the predicted distribution and the expected (true) mock-community distribution using QIIME2 or scikit-bio on the Greengenes phylogeny. Assemble distances into a tidy table (mock set x DC method) written to unifrac_distances.csv and select the method with lowest distance across all mock sets.

## Related tools

- **MiCoNE** (Pipeline framework used to execute each DC method with standardized config defaults and consensus chimera removal) — https://github.com/segrelab/MiCoNE
- **DADA2** (Denoising method under benchmark; set as default DC method for best weighted UniFrac performance)
- **Deblur** (Denoising method under benchmark, run with min_reads 2 and min_size 2)
- **QIIME2** (Executes the DC steps and computes weighted UniFrac distances in qiime2-compatible formats)

## Examples

```
micone init -w full -o mock_bench && bash run.sh  # with denoise_cluster selection = ['dada2', 'deblur', ...] and chimera_method "consensus" in nextflow.config
```

## Evaluation signals

- DADA2 exhibits the lowest weighted UniFrac distance to the expected distribution across mock4, mock12, and mock16, matching Fig. 2C/D of the paper
- All five DC methods x three mock sets yield a complete, non-empty entry in unifrac_distances.csv (15 rows)
- Chimera removal (chimera_method "consensus") was applied after every DC method run
- Reference-based OTU assignment used percent_identity 0.97 against gg_97 reference sequences
- Feature tables are readable in qiime2-compatible format and can be loaded with QIIME2 or scikit-bio for UniFrac computation on the Greengenes phylogeny

## Limitations

- No single DC method outperforms the rest in all data sets, so the DADA2-best result should be interpreted per dataset rather than as a universal ranking
- The benchmark depends on mockrobiota mock communities and the Greengenes phylogeny; conclusions may not transfer to biomes lacking ground truth
- The data directory (nf_micone/data) must be manually downloaded from Zenodo; missing reference data (e.g., gg_97) will break reference-based OTU assignment

## Evidence

- [other] DADA2, compared to Deblur, displays better performance on all the mock data sets on the weighted UniFrac metric and was therefore set as the default DC method; no single method outperforms the rest in all data sets.: [evidence span withheld — anchor status: not_found]
- [methods] OTU assignment closed reference percent_identity 0.97 gg_97: "| OTU assignment   | Closed reference | percent_identity    | 0.97        |"
- [methods] Chimera checking remove bimera consensus method: "| Chimera checking | Remove bimera    | chimera_method      | "consensus" |"
- [readme] dada2 and deblur selectable for denoise_cluster OTU assignment: "to change the tool used for OTU assignment to `dada2` and `deblur`, you can add the following to `nextflow.config`"
- [readme] The data directory needs to be manually downloaded from Zenodo record 7051556: "The data directory (`nf_micone/data`) needs to be manually downloaded using this [link](https://zenodo.org/record/7051556/files/data.zip?download=1)."
