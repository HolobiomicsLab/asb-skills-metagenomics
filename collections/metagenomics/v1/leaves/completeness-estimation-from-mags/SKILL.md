---
name: completeness-estimation-from-mags
description: 'Use when computing core genomes from MAGs/SAGs of diverse completeness:
  completeness priors must be supplied so the Bayesian estimator can weigh presence/absence
  patterns per genome.'
when_to_use_negative:
- Genomes are high-quality complete isolates only — Roary or PPanGGOLiN are validated
  comparators for high-quality genomes
- No completeness information is available and genomes are of unknown, potentially
  wildly wrong quality — the prior drives the core/accessory likelihoods, so a bad
  prior corrupts the partitioning
- The goal is OTU clustering rather than core-genome estimation — use mOTUlize.py
  instead
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan
  role: Consumes completeness priors to iteratively estimate core/accessory likelihoods
    of gene clusters
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan and mOTUlize.py for mOTU clustering
    and completeness-aware core-genome estimation
  repo: https://github.com/moritzbuck/mOTUlizer
- name: CheckM
  role: Source of completeness (and contamination) estimates supplied via --checkm
  repo: https://ecogenomics.github.io/CheckM/
- name: mmseqs2
  role: Computes gene clusters from input proteomes, whose presence/absence patterns
    are evaluated against completeness priors
  repo: https://github.com/soedinglab/MMseqs2
- name: mOTUconvert.py
  role: Converts Roary/PPanGGOLiN/eggNOGmapper/anvi'o outputs into --cog_file input
    for mOTUpan
  repo: https://github.com/moritzbuck/mOTUlizer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:aa1c7e3535d1670e81f9c533fa2f3c08cb538344489b6ee99b2228f55f923b61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/moritzbuck/mOTUlizer
  tools:
  - CheckM
  - mOTUconvert.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  - mmseqs2
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# completeness-estimation-from-mags

## Summary

Establishes genome completeness priors (e.g. from CheckM output or fixed values) for MAGs/SAGs of varying quality, which mOTUpan uses as the basis of its iterative Bayesian core/accessory likelihood estimation.

## When to use

Use when computing core genomes from MAGs/SAGs of diverse completeness: completeness priors must be supplied so the Bayesian estimator can weigh presence/absence patterns per genome.

## When NOT to use

- Genomes are high-quality complete isolates only — Roary or PPanGGOLiN are validated comparators for high-quality genomes
- No completeness information is available and genomes are of unknown, potentially wildly wrong quality — the prior drives the core/accessory likelihoods, so a bad prior corrupts the partitioning
- The goal is OTU clustering rather than core-genome estimation — use mOTUlize.py instead

## Inputs

- Proteome FASTA files (.faa) per genome, or a gene-cluster file (JSON or TAB-separated) via --cog_file
- CheckM completeness/contamination table, or a fixed completeness value, supplied via --checkm
- Optionally a pairwise similarity file (e.g. fastANI output) via --similarities

## Outputs

- Core/accessory partitioning table (output.tsv) with per-gene-cluster core-likelihood estimates
- Posterior completeness estimates per genome (used iteratively as the updated prior)
- With --boots: bootstrapped false-positive rate, recall, and 'lowest false' statistics

## How to apply

Provide a completeness file with `--checkm` (e.g. CheckM output or a fixed value) alongside proteomes (`--faas`) or pre-computed gene clusters (`--cog_file`, optionally produced by mOTUconvert.py from mmseqs2/Roary/PPanGGOLiN/eggNOGmapper/anvi'o). mOTUpan uses the completeness prior to compute, for each gene cluster, the likelihood ratio of the observed presence/absence pattern being core versus accessory; the resulting partitioning updates the completeness prior and is recomputed iteratively until convergence (bounded by `--max_iter`). In bootstrapping mode (`--boots`), artificial genomes are rarefied according to these posterior completeness estimates to derive false-positive rate, recall, and 'lowest false' statistics. Quality thresholds such as `--MAG-completeness`/`--MAG-contamination` control which genomes count as high-quality for mOTU network construction.

## Related tools

- **mOTUpan** (Consumes completeness priors to iteratively estimate core/accessory likelihoods of gene clusters) — https://github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan and mOTUlize.py for mOTU clustering and completeness-aware core-genome estimation) — https://github.com/moritzbuck/mOTUlizer
- **CheckM** (Source of completeness (and contamination) estimates supplied via --checkm) — https://ecogenomics.github.io/CheckM/
- **mmseqs2** (Computes gene clusters from input proteomes, whose presence/absence patterns are evaluated against completeness priors) — https://github.com/soedinglab/MMseqs2
- **mOTUconvert.py** (Converts Roary/PPanGGOLiN/eggNOGmapper/anvi'o outputs into --cog_file input for mOTUpan) — https://github.com/moritzbuck/mOTUlizer

## Examples

```
mOTUpan.py --faas *.faa --checkm checkm_output.tsv --boots 10 -o output.tsv
```

## Evaluation signals

- Output table contains completeness-updated core/accessory likelihoods per gene cluster and the iterative loop converged within --max_iter
- With --boots, bootstrapped false-positive-rate values fall in range [0,1] for each run and correlate with the empirical FPR against a liberal 'true' core (union of cores over 10 runs)
- Completeness file is parsed per genome (CheckM format or fixed value) and every input genome receives a prior; missing genomes should raise an error
- Re-running with the same --checkm input and proteomes reproduces the same partitioning after convergence (deterministic within bootstrap seed)

## Limitations

- The prior is iterative — an initial completeness misestimate propagates, so convergence depends on reasonable starting estimates (e.g. CheckM)
- fastANI similarity computation is the slowest step and memory-greedy; use --keep-simi-file to reuse similarities
- Genomes below the SUB-completeness threshold are treated as satellite bins and only recruited to mOTUs, not used as network anchors
- anvi-run-motupan.py is experimental with fewer options

## Evidence

- [abstract] We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range.: "We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range."
- [methods] iterative prior update until convergence: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] checkm completeness input option: "`--checkm`: provide a file containing completenesses and contaminations, see `mOTUpan`-section for detail"
- [readme] bootstrap rarefaction by posterior completeness: "these genomes are then rarefied according to the posterioir completeness estimates."
