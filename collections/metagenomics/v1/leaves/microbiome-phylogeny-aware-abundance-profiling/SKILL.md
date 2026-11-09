---
name: microbiome-phylogeny-aware-abundance-profiling
description: Use when you have feature/OTU tables from multiple denoising or clustering
  methods (e.g. vsearch open/closed-reference, de novo, DADA2, Deblur) and need to
  assess how similarly they resolve community composition using phylogeny-aware beta-diversity
  metrics.
when_to_use_negative:
- You only have a single OTU/feature table from one method and no comparison across
  DC methods — UniFrac cross-method comparison requires the same samples processed
  by multiple methods
- Your sequences lack a phylogeny (e.g. non-16S amplicons without reference-tree placement)
  — weighted/unweighted UniFrac requires phylogenetic relationships between observations
- The goal is network inference (correlation/direct association steps like sparcc
  or spieceasi) rather than comparing abundance profiles themselves
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0194
tools:
- name: MiCoNE
  role: Runs the five DC methods under controlled defaults and provides the workflow
    in which the per-method OTU tables and abundance profiles are generated
  repo: https://github.com/segrelab/MiCoNE
- name: qiime2
  role: Enforces supported input file formats (forward.fastq.gz, reverse.fastq.gz,
    barcodes.fastq.gz, mapping.tsv) for the demultiplexed reads and executes the UniFrac
    computation on the abundance profiles
- name: Nextflow
  role: Executes the MiCoNE denoising/clustering workflow template reproducibly via
    the run.sh script and nextflow.config
- name: Python
  role: Loads interaction/abundance matrices (tsv or csv format) for downstream computation
    of averaged UniFrac distance tables
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:8ab5820338db00c0a62714b9e971096a0604558ac994e7438f20594cf80d6a7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Nextflow
  - Python
  - qiime2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# microbiome-phylogeny-aware-abundance-profiling

## Summary

Converts OTU tables produced by alternative denoising/clustering (DC) methods into sample-by-observation abundance profiles that retain phylogenetic information, then quantifies similarity between methods using weighted and unweighted UniFrac distances, as done in the MiCoNE pipeline evaluation on the FMT (Qiita study 10532) 16S data.

## When to use

Use when you have feature/OTU tables from multiple denoising or clustering methods (e.g. vsearch open/closed-reference, de novo, DADA2, Deblur) and need to assess how similarly they resolve community composition using phylogeny-aware beta-diversity metrics.

## When NOT to use

- You only have a single OTU/feature table from one method and no comparison across DC methods — UniFrac cross-method comparison requires the same samples processed by multiple methods
- Your sequences lack a phylogeny (e.g. non-16S amplicons without reference-tree placement) — weighted/unweighted UniFrac requires phylogenetic relationships between observations
- The goal is network inference (correlation/direct association steps like sparcc or spieceasi) rather than comparing abundance profiles themselves

## Inputs

- Demultiplexed, trimmed paired-end 16S FASTQ files (forward.fastq.gz, reverse.fastq.gz, barcodes.fastq.gz) with mapping.tsv, in qiime2-supported formats
- OTU tables (sample-by-observation abundance matrices) from each DC method: vsearch open-reference, vsearch closed-reference, vsearch de novo, DADA2, Deblur
- Representative-sequence phylogenies derived from each method's OTU representative sequences
- gg_97 reference sequences for closed/open-reference clustering

## Outputs

- Per-method sample-by-observation abundance profiles with phylogenetic information
- unifrac_distances.csv: a method-by-method table of mean weighted and mean unweighted UniFrac distances averaged over all sample pairs

## How to apply

Run each DC method under MiCoNE defaults on the same demultiplexed, trimmed paired-end reads (vsearch clustering at percent_identity 0.97 against gg_97 for closed- and open-reference, de novo at 0.97, DADA2 with big_data = FALSE, Deblur with min_reads = 2 and min_size = 2), then apply chimera checking with chimera_method = 'consensus' and taxonomy assignment with the Naive Bayes classifier gg_13_8_99_515_806 at confidence 0.7. From each method's resulting OTU table build the sample-by-observation abundance profiles together with their phylogenetic tree. Compute weighted and unweighted UniFrac distances between all samples for each pair of methods, then average the pairwise distances over all samples. Rationale: weighting by abundance captures whether the representative sequence sets agree for dominant taxa, while the unweighted metric exposes differences from low-abundance sequences; averaging over samples yields a single method-by-method distance matrix suitable for outlier detection. Write the averaged table to unifrac_distances.csv for comparison against the published Fig. 2A/B pattern.

## Related tools

- **MiCoNE** (Runs the five DC methods under controlled defaults and provides the workflow in which the per-method OTU tables and abundance profiles are generated) — https://github.com/segrelab/MiCoNE
- **qiime2** (Enforces supported input file formats (forward.fastq.gz, reverse.fastq.gz, barcodes.fastq.gz, mapping.tsv) for the demultiplexed reads and executes the UniFrac computation on the abundance profiles)
- **Nextflow** (Executes the MiCoNE denoising/clustering workflow template reproducibly via the run.sh script and nextflow.config)
- **Python** (Loads interaction/abundance matrices (tsv or csv format) for downstream computation of averaged UniFrac distance tables)

## Examples

```
micone install && micone init -w full -o pipeline_dir && bash run.sh && micone validate-results
```

## Evaluation signals

- The weighted-UniFrac column of unifrac_distances.csv identifies Deblur as the outlier relative to the other four methods, reproducing the Fig. 2A pattern
- The unweighted-UniFrac dissimilarity between methods is attributable to low-abundance sequences, reproducing the Fig. 2B pattern
- The output table has one row/column per DC method (open-reference, closed-reference, de novo vsearch, DADA2, Deblur) with 10 method pairs and both weighted and unweighted mean distances
- All five DC methods were run with identical defaults: percent_identity 0.97, chimera_method 'consensus', classifier gg_13_8_99_515_806 at confidence 0.7, so differences reflect DC choice only
- micone validate-results reports no failures for the completed pipeline run

## Limitations

- Deblur (min_reads = 2, min_size = 2) stands out as the outlier under mean weighted UniFrac, so cross-method agreement cannot be assumed for all DC choices
- Unweighted UniFrac disagreement between methods does not imply disagreement for dominant taxa — it can arise entirely from low-abundance sequences
- MiCoNE requires the mamba package manager or micone init will not work, and the pipeline data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556

## Evidence

- [results] The representative sequences generated by the five DC methods on the FMT study reads are similar to each other when weighted by their abundance (mean weighted UniFrac), with the exception of Deblur, which stands out as the outlier.: [evidence span withheld — anchor status: not_found]
- [methods] From each method's resulting OTU table build the sample-by-observation abundance profiles with phylogenetic information and compute weighted and unweighted UniFrac distances between all samples for each pair of methods.: [evidence span withheld — anchor status: not_found]
- [results] verify that the weighted-UniFrac column identifies Deblur as the outlier relative to the other four methods and that the unweighted-UniFrac dissimilarity is attributable to low-abundance sequences: [evidence span withheld — anchor status: not_found]
- [readme] These files must follow the ``qiime2`` supported formats: "The MiCoNE framework is introduced in: Kishore, D., Birzu, G., Hu, Z., DeLisi, C., Korolev, K., & Segrè, D. (2023). Inferring microbial co-occurrence networks"
