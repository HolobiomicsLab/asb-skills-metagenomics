---
name: r-visualization-of-accuracy-curves
description: Use when per-subsample accuracy results for high-, medium-, and low-abundance
  transcript classes have been computed across sequencing depths and need to be plotted
  to determine the minimum annotated-read depth (5-10M) exceeding 90% accuracy.
when_to_use_negative:
- The input is raw annotation output or per-read abundance counts that have not yet
  been reduced to per-depth accuracy values — compute accuracy first in Python before
  plotting in R
- The goal is differential expression statistics between control vs. experimental
  groups (use the R_organism_script.Rmd / R_functional_script_w-md5s.Rmd DESeq2 workflow
  instead)
- The sample is not deeply sequenced enough to serve as a full-sample reference (the
  paper's pilot was sequenced to ~40-50 million raw sequences yielding 5-10 million
  annotated reads)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Imports the per-depth accuracy results and generates the accuracy-versus-depth
    plot stratified by abundance class; SAMSA lists R, preferably RStudio, as a dependency
  repo: https://github.com/transcript/SAMSA
- name: Python 2.7
  role: 'Upstream step: draws random subsets of annotated reads per subsampling size
    and computes per-class relative-abundance accuracy results that feed the R plotting'
  repo: https://github.com/transcript/SAMSA
- name: MG-RAST
  role: Annotation server whose aggregated organism/functional summary outputs of
    the deeply sequenced pilot sample provide the reference abundances
  repo: https://github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:cd0d4652d689ea07aa66c9f15381e9a696f54e9ccdfa1fd55a9e59f825acb429
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - MG-RAST
  - Python 2.7
  - R
  - SAMSA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# r-visualization-of-accuracy-curves

## Summary

Visualize metatranscriptome sequencing-depth accuracy curves in R (preferably RStudio) by importing per-depth accuracy results computed in Python 2.7 and plotting accuracy versus sequencing depth stratified by transcript abundance class, to identify the depth at which low-abundance accuracy exceeds 90%.

## When to use

Use when per-subsample accuracy results for high-, medium-, and low-abundance transcript classes have been computed across sequencing depths and need to be plotted to determine the minimum annotated-read depth (5-10M) exceeding 90% accuracy.

## When NOT to use

- The input is raw annotation output or per-read abundance counts that have not yet been reduced to per-depth accuracy values — compute accuracy first in Python before plotting in R
- The goal is differential expression statistics between control vs. experimental groups (use the R_organism_script.Rmd / R_functional_script_w-md5s.Rmd DESeq2 workflow instead)
- The sample is not deeply sequenced enough to serve as a full-sample reference (the paper's pilot was sequenced to ~40-50 million raw sequences yielding 5-10 million annotated reads)

## Inputs

- Per-depth accuracy results table (sequencing depth vs. accuracy, stratified by abundance class) exported from the Python 2.7 subsampling step
- Aggregated organism/functional summary files derived from MG-RAST annotation results of a deeply sequenced pilot metatranscriptome sample

## Outputs

- Plot of relative-abundance accuracy versus sequencing depth, stratified by high-, medium-, and low-abundance transcript classes
- Figure annotation marking the depth at which low-abundance accuracy exceeds 90% (5-10 million annotated reads)

## How to apply

First, in Python 2.7, subsample the aggregated MG-RAST organism/functional summary files from 1M to 20M annotated reads in 10% increments, drawing 10 random subsets per size, and compute relative-abundance accuracy for each abundance class (stratified by abundance in the full deeply sequenced sample) against the full-sample reference. Then import the per-depth accuracy results into R (RStudio, consistent with SAMSA's R dependency) and plot accuracy versus sequencing depth with separate strata for high-, medium-, and low-abundance transcripts. Add a visual marker at the depth where low-abundance accuracy first exceeds 90%. Validate the figure against the paper's expectation that >90% low-abundance accuracy is reached at 5-10 million annotated reads, equivalent to 40-50 million raw paired-end 100 bp reads per sample.

## Related tools

- **R** (Imports the per-depth accuracy results and generates the accuracy-versus-depth plot stratified by abundance class; SAMSA lists R, preferably RStudio, as a dependency) — https://github.com/transcript/SAMSA
- **Python 2.7** (Upstream step: draws random subsets of annotated reads per subsampling size and computes per-class relative-abundance accuracy results that feed the R plotting) — https://github.com/transcript/SAMSA
- **MG-RAST** (Annotation server whose aggregated organism/functional summary outputs of the deeply sequenced pilot sample provide the reference abundances) — https://github.com/transcript/SAMSA

## Examples

```
In R (RStudio): results <- read.csv('depth_accuracy_results.csv'); plot(results$depth, results$accuracy_low, type='l', xlab='Annotated reads (millions)', ylab='Accuracy (%)'); abline(h=90, lty=2)
```

## Evaluation signals

- The plotted curve must show low-abundance transcript accuracy exceeding 90% at 5-10 million annotated reads (equivalent to 40-50 million raw sequence reads), matching the paper's recommendation
- Accuracy must increase monotonically (with noise across the 10 random subsets per size) toward the full-sample reference as depth increases from 1M to 20M annotated reads
- The low-abundance class curve must sit below the medium- and high-abundance curves at every depth; if not, the abundance stratification from the full sample was misapplied
- The figure must contain three strata (high, medium, low abundance classes) and an explicit marker at the >90% low-abundance depth
- Per-depth accuracy values should be reproducible within variance across the 10 random subsets drawn per subsampling size

## Limitations

- MG-RAST annotation servers require queue waits of days to weeks, so regenerating the underlying annotated reads can be slow
- Accuracy curves depend on the full deeply sequenced sample as reference; results reflect this single human gut stool metatranscriptome and ribodepletion/kit choices used in the study
- The skill estimates accuracy for relative abundance of annotated mRNA reads only; remaining rRNA sequences must be discarded first or the curve will be distorted

## Evidence

- [abstract] Each sample needs 40-50 million raw sequences, expected to yield 5-10 million annotated reads for accurate abundance measures: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
- [methods] SAMSA dependencies include Python 2.7 and R: "SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R"
- [readme] SAMSA pipeline purpose from README: "After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2."
- [discussion] Four main steps of metatranscriptomic analysis: "Our pipeline fulfills the need for four main steps in metatranscriptomic data analysis: preprocessing, annotation, aggregation, and analysis."
- [abstract] Recommendation on sequencing depth per sample: "We further recommend that stool metatranscriptomes be ribodepleted and sequenced in a 100 bp paired end format with a minimum of 40 million reads per sample."
