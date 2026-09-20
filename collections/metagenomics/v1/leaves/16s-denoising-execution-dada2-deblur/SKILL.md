---
name: 16s-denoising-execution-dada2-deblur
description: Use when demultiplexed, trimmed paired-end 16S reads must be processed
  through multiple denoising/clustering alternatives (DADA2, Deblur, vsearch open/closed/de
  novo) and the resulting representative sequence sets compared via pairwise UniFrac
  distances.
when_to_use_negative:
- Input is already a feature/OTU table from a single denoising or clustering run —
  the DC comparison step presupposes raw demultiplexed reads processed under all five
  alternatives
- Only one DC method is of interest and no cross-method agreement assessment (UniFrac
  comparison) is needed
- Data are not 16S rRNA amplicon reads (e.g., shotgun metagenomics) — the pipeline
  is built around 16S rRNA amplicon profiling
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: MiCoNE
  role: Python package/pipeline that orchestrates the five DC alternatives (dada2,
    deblur, vsearch open/closed/de novo) under Nextflow and provides the 'micone install',
    'micone init', and 'micone validate-results' commands
  repo: https://github.com/segrelab/MiCoNE
- name: Nextflow
  role: Execution engine for the workflow template initialized by 'micone init'; run
    via the generated run.sh script
- name: qiime2
  role: Underlying processing environment (micone-qiime2) whose supported file formats
    and tools handle demultiplexed reads, chimera removal, and taxonomy assignment
- name: vsearch
  role: Performs open-reference, closed-reference, and de novo OTU clustering at percent_identity
    0.97 against the gg_97 reference sequences
- name: DADA2
  role: Denoising alternative run with big_data = FALSE on the FMT study reads
- name: Deblur
  role: Denoising alternative run with min_reads = 2 and min_size = 2; its representative
    sequences emerge as the weighted-UniFrac outlier
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:93b3874e1d2bd97b1f1e07fe8a5259634302fcd8c9616199792ec3ca144f80e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - DADA2
  - Deblur
  - MiCoNE
  - Nextflow
  - Python
  - qiime2
  - vsearch
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# 16s-denoising-execution-dada2-deblur

## Summary

Executes the MiCoNE denoising/clustering (DC) step on demultiplexed 16S paired-end reads using five alternatives — vsearch open-reference, closed-reference, and de novo clustering, DADA2, and Deblur — then compares the representative sequence sets by mean weighted and unweighted UniFrac distances between method pairs.

## When to use

Use when demultiplexed, trimmed paired-end 16S reads must be processed through multiple denoising/clustering alternatives (DADA2, Deblur, vsearch open/closed/de novo) and the resulting representative sequence sets compared via pairwise UniFrac distances.

## When NOT to use

- Input is already a feature/OTU table from a single denoising or clustering run — the DC comparison step presupposes raw demultiplexed reads processed under all five alternatives
- Only one DC method is of interest and no cross-method agreement assessment (UniFrac comparison) is needed
- Data are not 16S rRNA amplicon reads (e.g., shotgun metagenomics) — the pipeline is built around 16S rRNA amplicon profiling

## Inputs

- Demultiplexed, trimmed paired-end 16S FASTQ reads (forward.fastq.gz, reverse.fastq.gz, barcodes.fastq.gz) in qiime2-supported formats
- Sample metadata mapping file (mapping.tsv)
- gg_97 reference sequences (for closed/open-reference vsearch clustering)
- Greengenes gg_13_8_99_515_806 99% Naive Bayes classifier (taxonomy assignment)

## Outputs

- Per-method OTU/feature tables (abundance profiles) with phylogenetic trees from the five DC methods
- unifrac_distances.csv — a method-by-method matrix of mean weighted and unweighted UniFrac distances averaged over all samples

## How to apply

Install the MiCoNE conda environments with 'micone install' and initialize the workflow template with 'micone init -w <workflow>' so the DC step runs under Nextflow; load demultiplexed, trimmed paired-end reads (e.g., runs 2-4 of Qiita study 10532) using the qiime2-supported naming (forward.fastq.gz, reverse.fastq.gz, barcodes.fastq.gz, mapping.tsv). Run OTU assignment with all five methods under MiCoNE defaults: closed-reference and open-reference vsearch clustering at percent_identity 0.97 against the gg_97 reference sequences, de novo clustering at percent_identity 0.97, DADA2 with big_data = FALSE, and Deblur with min_reads = 2 and min_size = 2. Apply chimera checking with the 'Remove bimera' tool using chimera_method = 'consensus' and assign taxonomy with the Naive Bayes classifier gg_13_8_99_515_806 at confidence 0.7. From each method's OTU table, build sample-by-observation abundance profiles with phylogenetic information, compute weighted and unweighted UniFrac distances between all samples for each pair of methods, average over samples, and write the method-by-method table to unifrac_distances.csv. The rationale is that different DC choices lead to vastly different inferences of community composition, so the comparison quantifies how similar the five representative sequence sets are — in the FMT study they are similar under weighted UniFrac except for Deblur, which stands out as the outlier.

## Related tools

- **MiCoNE** (Python package/pipeline that orchestrates the five DC alternatives (dada2, deblur, vsearch open/closed/de novo) under Nextflow and provides the 'micone install', 'micone init', and 'micone validate-results' commands) — https://github.com/segrelab/MiCoNE
- **Nextflow** (Execution engine for the workflow template initialized by 'micone init'; run via the generated run.sh script)
- **qiime2** (Underlying processing environment (micone-qiime2) whose supported file formats and tools handle demultiplexed reads, chimera removal, and taxonomy assignment)
- **vsearch** (Performs open-reference, closed-reference, and de novo OTU clustering at percent_identity 0.97 against the gg_97 reference sequences)
- **DADA2** (Denoising alternative run with big_data = FALSE on the FMT study reads)
- **Deblur** (Denoising alternative run with min_reads = 2 and min_size = 2; its representative sequences emerge as the weighted-UniFrac outlier)

## Examples

```
micone install && micone init -w full -o dc_pipeline
```

## Evaluation signals

- All five DC methods complete with MiCoNE defaults: vsearch clustering at percent_identity 0.97 (open/closed reference against gg_97 and de novo), DADA2 with big_data = FALSE, Deblur with min_reads = 2 and min_size = 2
- unifrac_distances.csv contains a symmetric method-by-method table of mean weighted and unweighted UniFrac distances averaged over all samples for all method pairs
- The weighted-UniFrac column identifies Deblur as the outlier relative to the other four methods, reproducing the Fig. 2A pattern
- The unweighted-UniFrac dissimilarity between methods is attributable to low-abundance sequences, reproducing the Fig. 2B pattern
- 'micone validate-results' passes on the executed pipeline run

## Limitations

- Deblur's representative sequences are the weighted-UniFrac outlier on the FMT data, so the five DC methods cannot be assumed fully interchangeable; unweighted-UniFrac differences are driven by low-abundance sequences and are sensitive to rare OTUs
- DADA2 is run with big_data = FALSE, which is not appropriate for very large read sets
- MiCoNE requires the mamba package manager (otherwise 'micone init' will not work), and the data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556

## Evidence

- [methods] | OTU assignment   | Closed reference | percent_identity    | 0.97        |: "| OTU assignment   | Closed reference | percent_identity    | 0.97        |"
- [methods] | Chimera checking | Remove bimera    | chimera_method      | "consensus" |: "| Chimera checking | Remove bimera    | chimera_method      | "consensus" |"
- [methods] | Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |: "| Assign | Naive Bayes | classifer     | "gg_13_8_99_515_806" |"
- [methods] To set up the ``nextflow`` workflow template for the desired workflow, you can use the ``micone init`` command: "To set up the ``nextflow`` workflow template for the desired workflow, you can use the ``micone init`` command"
- [intro] we present a standardized 16S data analysis pipeline called Microbial Co-occurrence Network Explorer (MiCoNE) that produces robust and reproducible co-occurrence networks: "we present a standardized 16S data analysis pipeline called Microbial Co-occurrence Network Explorer (MiCoNE) that produces robust and reproducible co-occurrence networks"
- [readme] MiCoNE requires the `mamba` package manager, otherwise `micone init` will not work.: "MiCoNE requires the `mamba` package manager, otherwise `micone init` will not work."
