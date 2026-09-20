---
name: amplicon-otu-clustering-method-comparison
description: Use when demultiplexed 16S amplicon reads must be processed by multiple
  OTU-assignment/denoising methods and the similarity of the resulting representative
  sequence sets needs to be quantified with UniFrac distances.
when_to_use_negative:
- The goal is comparing co-occurrence networks or downstream network-inference steps
  rather than the denoising/clustering step itself
- Input data is shotgun metagenomic or non-16S amplicon data outside the qiime2-supported
  naming the MiCoNE pipeline expects
- Only a single OTU-assignment method is needed; the skill is specifically for cross-method
  comparison
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0699
- http://edamontology.org/topic_3301
tools:
- name: MiCoNE
  role: Python package and Nextflow pipeline that executes the five DC methods under
    controlled defaults and configs
  repo: https://github.com/segrelab/MiCoNE
- name: qiime2
  role: Runs OTU assignment, chimera checking, and taxonomy assignment within the
    DC step; requires qiime2-supported input file formats
- name: Nextflow
  role: Workflow engine for the DC pipeline template initialized via 'micone init
    -w <workflow>'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:371a31d98f501d494e282118d50b1c7709dab0fd03a6d8d270f7c22f79a42a1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Nextflow
  - Python
  - qiime2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0699
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# amplicon-otu-clustering-method-comparison

## Summary

Reproduces the MiCoNE denoising/clustering (DC) step comparison on FMT 16S reads by running five representative-sequence methods (vsearch open-reference, closed-reference, and de novo clustering, DADA2, Deblur) and quantifying mean weighted/unweighted UniFrac distances between method pairs.

## When to use

Use when demultiplexed 16S amplicon reads must be processed by multiple OTU-assignment/denoising methods and the similarity of the resulting representative sequence sets needs to be quantified with UniFrac distances.

## When NOT to use

- The goal is comparing co-occurrence networks or downstream network-inference steps rather than the denoising/clustering step itself
- Input data is shotgun metagenomic or non-16S amplicon data outside the qiime2-supported naming the MiCoNE pipeline expects
- Only a single OTU-assignment method is needed; the skill is specifically for cross-method comparison

## Inputs

- Demultiplexed, trimmed paired-end 16S FASTQ reads (forward.fastq.gz, reverse.fastq.gz, barcodes.fastq.gz) from runs 2-4 of Qiita study 10532
- Sample metadata mapping file (mapping.tsv) in qiime2-supported format
- gg_97 reference sequences for closed/open-reference clustering
- Greengenes 13_8 99% 515/806-region naive Bayes classifier (gg_13_8_99_515_806)

## Outputs

- Per-method OTU/ASV feature (abundance) tables with representative sequences and assigned taxonomy
- Method-by-method matrix of mean weighted and unweighted UniFrac distances written to unifrac_distances.csv

## How to apply

Install the MiCoNE conda environments with 'micone install' and initialize the denoising/clustering workflow template with 'micone init -w <workflow>' so the DC step runs under Nextflow. Load demultiplexed, trimmed paired-end reads (runs 2-4 of Qiita study 10532) in qiime2-supported format (forward.fastq.gz, reverse.fastq.gz, barcodes.fastq.gz, mapping.tsv). Run OTU assignment with the five methods using MiCoNE defaults: closed- and open-reference vsearch clustering at percent_identity 0.97 against gg_97 reference sequences, de novo clustering at percent_identity 0.97, DADA2 (big_data = FALSE), and Deblur (min_reads = 2, min_size = 2). Apply chimera checking with 'Remove bimera' (chimera_method = 'consensus') and assign taxonomy with the Naive Bayes classifier gg_13_8_99_515_806 at confidence 0.7. From each OTU table build sample-by-observation abundance profiles with phylogenetic information, compute weighted and unweighted UniFrac distances between all samples for each pair of methods, average over samples, and write the method-by-method table to unifrac_distances.csv.

## Related tools

- **MiCoNE** (Python package and Nextflow pipeline that executes the five DC methods under controlled defaults and configs) — https://github.com/segrelab/MiCoNE
- **qiime2** (Runs OTU assignment, chimera checking, and taxonomy assignment within the DC step; requires qiime2-supported input file formats)
- **Nextflow** (Workflow engine for the DC pipeline template initialized via 'micone init -w <workflow>')

## Examples

```
micone install && micone init -w dc -o fmt_dc && bash run.sh
```

## Evaluation signals

- The weighted-UniFrac column of unifrac_distances.csv identifies Deblur as the outlier relative to the other four methods, which are similar
- Unweighted-UniFrac dissimilarity is attributable to low-abundance sequences, reproducing the Fig. 2A/B pattern
- All five DC methods ran with MiCoNE defaults: percent_identity 0.97 for vsearch clustering, DADA2 big_data = FALSE, Deblur min_reads = 2 and min_size = 2
- Chimera checking used chimera_method = 'consensus' and taxonomy was assigned with gg_13_8_99_515_806 at confidence 0.7
- unifrac_distances.csv has a valid symmetric method-by-method layout over all five methods averaged across all samples

## Limitations

- Deblur behaves as an outlier on the FMT data, so cross-method conclusions should not assume all denoisers converge
- Results are tied to MiCoNE default parameters and the gg_97 reference; other parameters or references may change the comparison
- MiCoNE requires mamba for 'micone init'; a preinstalled julia may conflict with the micone-flashweave environment, and the pipeline data directory must be manually downloaded from Zenodo record 7051556

## Evidence

- [methods] OTU assignment at 0.97 identity: "| OTU assignment   | Closed reference | percent_identity    | 0.97        |"
- [methods] Chimera method consensus: "| Chimera checking | Remove bimera    | chimera_method      | "consensus" |"
- [readme] Pipeline setup via micone commands: "In order to run the pipeline various ``conda`` environments must first be installed on the system. Use the following comand to initialize all the environments: ``micone install``"
- [abstract] MiCoNE explores tool effects on networks: "``MiCoNE`` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow"
