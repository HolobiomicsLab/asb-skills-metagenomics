---
name: genome-completeness-rarefaction
description: Use when you need to validate a core-genome prediction made on genomes
  of heterogeneous completeness (e.g. MAGs with CheckM completeness priors) by simulating
  synthetic genomes degraded to match the incompleteness of the real input set.
when_to_use_negative:
- When the genome set consists of complete, high-quality genomes only — the rarefaction
  models incompleteness that is not present, so bootstrap metrics would be uninformative
- When no completeness estimates (CheckM output or fixed values) are available for
  the input genomes, since the posterior completeness estimates used for rarefaction
  cannot be computed
- When you only need the core/accessory partitioning itself and do not need quality
  estimates (false discovery rate and sensitivity) for that prediction
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: mOTUpan
  role: Generates the artificial genomes (--boots), rarefies them by posterior completeness
    estimates, and re-classifies their gene clusters with the core-likelihood classifier
    to estimate false-positive rate and recall
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan and its bootstrapping procedure
  repo: github.com/moritzbuck/mOTUlizer
- name: CheckM
  role: Source of the per-genome completeness estimates supplied via --checkm that
    drive the rarefaction of artificial genomes
  repo: ecogenomics.github.io/CheckM
- name: MMseqs2
  role: Computes the gene clusters (easy_cluster) whose core/accessory partitioning
    feeds the bootstrap generation and re-classification
  repo: soedinglab/MMseqs2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:53f204bc899462aa4e5c55dbfb8f515bf4740e7bc45419cfe7e275808663f740
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - MMseqs2
  - mOTUlizer
  - mOTUpan
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# genome-completeness-rarefaction

## Summary

A bootstrap-quality step in mOTUpan where artificially generated genomes — built from all core gene clusters plus accessory clusters sampled from a frequency-conserving gene pool — are rarefied according to the posterior completeness estimates so that synthetic genomes are missing the same fraction of genes as the real (incomplete) MAGs.

## When to use

Use when you need to validate a core-genome prediction made on genomes of heterogeneous completeness (e.g. MAGs with CheckM completeness priors) by simulating synthetic genomes degraded to match the incompleteness of the real input set.

## When NOT to use

- When the genome set consists of complete, high-quality genomes only — the rarefaction models incompleteness that is not present, so bootstrap metrics would be uninformative
- When no completeness estimates (CheckM output or fixed values) are available for the input genomes, since the posterior completeness estimates used for rarefaction cannot be computed
- When you only need the core/accessory partitioning itself and do not need quality estimates (false discovery rate and sensitivity) for that prediction

## Inputs

- Gene-cluster partitioning (core vs non-core) produced by mOTUpan from proteome FASTA files (--faas) or a user-supplied --cog_file (JSON or TAB-separated)
- Per-genome completeness estimates (CheckM output or fixed values) supplied via the --checkm file
- Posterior completeness estimates computed by the mOTUpan iterative classifier
- BOOTS: number of bootstrap replicates specified via --boots

## Outputs

- Per-replicate bootstrap metrics: false-positive rate, recall, and 'lowest false' frequency
- Aggregated bootstrap quality statistics (means and standard deviations over BOOTS replicates) written to bootstrap_stats.tsv

## How to apply

First run mOTUpan.py on the input proteomes (gene clusters computed with mmseqs2) with completeness priors supplied via --checkm (CheckM output or fixed values) to obtain the core/accessory partitioning and posterior completeness estimates. Invoke mOTUpan.py with --boots BOOTS to generate BOOTS artificial genomes: every artificial genome contains all core genes, and the remaining genes are sampled from a gene pool that conserves their observed frequencies in the real genome set. Rarefy each artificial genome according to the posterior completeness estimates so it is missing the same fraction of genes as the corresponding incomplete real MAG. Re-classify the gene clusters of these rarefied artificial genomes with the mOTUpan core-likelihood classifier and, per replicate, compute the false-positive rate (fraction of core genes that might not be core), the recall (fraction of core genes classified as core in the bootstrap), and 'lowest false' (lowest frequency of any false-positive gene cluster, expected to be high). Aggregate over all BOOTS replicates to obtain standard deviations for the three metrics; rationale is that only by matching the synthetic incompleteness to the real incompleteness do false positives in the bootstrap reflect errors the classifier would actually make on the real data.

## Related tools

- **mOTUpan** (Generates the artificial genomes (--boots), rarefies them by posterior completeness estimates, and re-classifies their gene clusters with the core-likelihood classifier to estimate false-positive rate and recall) — github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan and its bootstrapping procedure) — github.com/moritzbuck/mOTUlizer
- **CheckM** (Source of the per-genome completeness estimates supplied via --checkm that drive the rarefaction of artificial genomes) — ecogenomics.github.io/CheckM
- **MMseqs2** (Computes the gene clusters (easy_cluster) whose core/accessory partitioning feeds the bootstrap generation and re-classification) — soedinglab/MMseqs2

## Examples

```
mOTUpan.py --faas *.faa --checkm checkm_output.tsv --boots 100 -o output.tsv
```

## Evaluation signals

- All BOOTS bootstrap replicates complete and the run finishes without errors
- Per-replicate false-positive rate, recall, and 'lowest false' values are finite and interpretable
- 'lowest false' is high, indicating that candidate false-positive gene clusters are highly prevalent in the genome set
- Aggregated statistics include standard deviations for the three quality metrics (higher --boots gives tighter standard deviations)
- bootstrap_stats.tsv is written with the bootstrap quality statistics

## Limitations

- Bootstrap quality metrics are estimates tied to the supplied completeness priors; inaccurate CheckM completeness estimates propagate into the rarefaction and hence into the false-positive-rate and recall estimates
- Artificial genomes are limited by the observed gene-pool frequencies — accessory clusters are sampled conserving observed frequencies, so unobserved gene-pool compositions are not modeled
- Interpretation of 'lowest false' depends on genome-set prevalence: it should be high, and a low value signals that possible false positives are rare, weakening the confidence interpretation
- Higher numbers of bootstrap replicates are needed to obtain meaningful standard deviations on the three metrics

## Evidence

- [other] bootstrapping builds complete synthetic genomes by drawing all core gene clusters plus randomly drawn accessory clusters, rarefies them by posterior completeness estimates: [evidence span withheld — anchor status: not_found]
- [readme] rarefied according to the posterior completeness estimates: "these genomes are then rarefied according to the posterioir completeness estimates"
- [readme] The bootstrap will provide an estimate of the false positive rate ... the recall ... and 'lowest false': "The bootstrap will provide an estimate of the false positive rate (e.g. fraction of core genes that might not be), the recall (fraction of core genes that have been classified as such in the"
- [readme] artificial genomes are generated using the gene-partitioning obtained with mOTUpan: "`--boots BOOTS` : runs `BOOTS` bootstraps, where artificial genomes are generated using the gene-partitioning obtained with `mOTUpan`"
- [readme] provide to mOTUpan ... a completeness prior of these genomes (for example checkm output or a fixed value): [evidence span withheld — anchor status: not_found]
