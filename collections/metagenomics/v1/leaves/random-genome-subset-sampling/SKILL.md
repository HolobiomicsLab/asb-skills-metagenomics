---
name: random-genome-subset-sampling
description: Use when you need to validate whether a core-genome prediction quality
  estimate (e.g. mOTUpan --boots false-positive rate) reliably predicts true accuracy,
  especially for MAGs of varying completeness.
when_to_use_negative:
- When a trusted complete reference genome set already defines the true core genome
  — no subset sampling is needed
- When the goal is core-genome estimation itself rather than validation of mOTUpan's
  quality estimates
- When genomes span multiple species/mOTUs such that a shared core genome is not expected
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: mOTUpan.py
  role: Runs the iterative Bayesian core/accessory likelihood estimation on sampled
    genome subsets and, with --boots, generates artificial rarefied genomes to estimate
    the false-positive rate
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan and supporting utilities
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mmseqs2
  role: Computes gene clusters from the sampled genomes' proteomes within mOTUpan.py
  repo: https://github.com/soedinglab/MMseqs2
- name: mOTUconvert.py
  role: Converts outputs of tools like Roary, PPanGGOLiN, eggNOGmapper or anvio into
    --cog_file input for mOTUpan.py as an alternative to --faas
  repo: https://github.com/moritzbuck/mOTUlizer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:a40a155fef3779c3b6c717f0443accd2ac8a35881a89ec1cce8654a69bc5e8ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/moritzbuck/mOTUlizer
  tools:
  - mOTUconvert.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  - mmseqs2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# random-genome-subset-sampling

## Summary

Establishes a liberal reference ('true') core genome and benchmarks mOTUpan's bootstrapped false-positive rate by repeatedly sampling random subsets of genomes (e.g. 100 Prochlorococcus_A genomes) and comparing empirical accuracy against the bootstrap estimate.

## When to use

Use when you need to validate whether a core-genome prediction quality estimate (e.g. mOTUpan --boots false-positive rate) reliably predicts true accuracy, especially for MAGs of varying completeness.

## When NOT to use

- When a trusted complete reference genome set already defines the true core genome — no subset sampling is needed
- When the goal is core-genome estimation itself rather than validation of mOTUpan's quality estimates
- When genomes span multiple species/mOTUs such that a shared core genome is not expected

## Inputs

- Prochlorococcus_A genome set (sampled subset of 100 genomes per run)
- Protein FASTA proteomes (.faa) per genome, or a pre-computed gene-cluster file (--cog_file JSON/TAB from mOTUconvert.py)
- Genome completeness/contamination estimates (e.g. CheckM output) supplied via --checkm

## Outputs

- Liberal 'true' core genome (union of core genes over repeated runs)
- Table with empirical and bootstrapped false-positive rates, recall, and 'lowest false' statistics per run (cf. Supplementary Table S3 / Figure S1)

## How to apply

Randomly sample a fixed-size subset (e.g. 100 genomes) from a genome set such as Prochlorococcus_A. Compute gene clusters with mmseqs2 via mOTUpan.py --faas (or a pre-computed --cog_file produced by mOTUconvert.py) and run mOTUpan.py multiple times (e.g. 10 runs) with --checkm completenesses and the iterative Bayesian core/accessory likelihood estimation until convergence. Take the union of core genes across the repeated runs as the liberal 'true' core. Then rerun mOTUpan.py with --boots to rarefy artificial genomes from the core/accessory partitioning, obtaining the bootstrapped false-positive rate, recall, and 'lowest false' statistics. Finally, classify genes against the liberal union core to derive an empirical false-positive rate and compare it with the bootstrapped estimate, reproducing the Supplementary Table S3 / Figure S1 relationship.

## Related tools

- **mOTUpan.py** (Runs the iterative Bayesian core/accessory likelihood estimation on sampled genome subsets and, with --boots, generates artificial rarefied genomes to estimate the false-positive rate) — https://github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan and supporting utilities) — https://github.com/moritzbuck/mOTUlizer
- **mmseqs2** (Computes gene clusters from the sampled genomes' proteomes within mOTUpan.py) — https://github.com/soedinglab/MMseqs2
- **mOTUconvert.py** (Converts outputs of tools like Roary, PPanGGOLiN, eggNOGmapper or anvio into --cog_file input for mOTUpan.py as an alternative to --faas) — https://github.com/moritzbuck/mOTUlizer

## Examples

```
mOTUpan.py --faas subset_*.faa --checkm checkm_output.tsv --boots 100 -o output.tsv
```

## Evaluation signals

- The output table contains both empirical and bootstrapped false-positive-rate columns for each run
- Bootstrapped FPR values lie in range [0,1]
- The union of core genes over the 10 runs yields the liberal 'true' core used for classification
- Empirical false-positive rates correlate with the bootstrapped FPR, reproducing the Supplementary Table S3 / Figure S1 relationship
- 'lowest false' statistic is high, indicating that possible false positives are highly prevalent in the genome set

## Limitations

- The liberal union core is a heuristic reference, not a verified truth; it may overestimate the true core
- Bootstrap estimates depend on the posterior completeness estimates used for rarefaction; problematic input genome combinations must be flagged via the bootstrapped FPR
- Higher number of bootstraps are needed to obtain standard deviations for the bootstrap statistics

## Evidence

- [methods] Bootstrapping over randomly sampled genome subsets: "`--boots BOOTS` : runs `BOOTS` bootstraps, where artificial genomes are generated using the gene-partitioning obtained with `mOTUpan`"
- [readme] Iterative Bayesian core/accessory estimation: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [abstract] Bootstrapped FPR predicts true accuracy: "mOTUpan wraps a bootstrapping procedure to estimate the quality of a specific core-genome prediction"
- [methods] Completeness priors from CheckM: "(for example [`checkm`](https://ecogenomics.github.io/CheckM/) output or a fixed value)"
- [methods] Alternative gene-cluster input: "`--cog_file` : can be used as an alternative to `--faas`, you can use it to provide you own gene-clusters (or other genetically encoded traits)."
