---
name: trait-presence-absence-pattern-modeling
description: 'Use when computing core genomes from MAGs/SAGs of varying completeness:
  presence/absence patterns of gene clusters or traits across genomes must be partitioned
  into core vs accessory while accounting for incompleteness.'
when_to_use_negative:
- Genomes are all high-quality/complete and a standard pangenome tool like Roary or
  PPanGGOLiN suffices; mOTUpan's value is robustness to incompleteness
- You need OTU-tables computed directly from reads — that is the unrelated 'mOTUs'
  tool, not mOTUlizer/mOTUpan
- Traits are quantitative or continuously distributed rather than discrete presence/absence
  genome-encoded traits
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan.py
  role: Runs the iterative Bayesian core/accessory likelihood loop, completeness re-estimation,
    and optional bootstrapping; writes output.tsv
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan; provides mOTUlize.py for OTU clustering,
    mOTUconvert.py for converting external pangenome outputs to mOTUpan inputs
  repo: github.com/moritzbuck/mOTUlizer
- name: MMseqs2
  role: Computes gene clusters (mmseqs easy_cluster) from input proteomes before trait
    classification
  repo: soedinglab/MMseqs2
- name: CheckM
  role: Source of completeness/contamination priors used for the Bayesian estimator
    and iterative re-estimation
  repo: ecogenomics.github.io/CheckM
- name: mOTUconvert.py
  role: Converts Roary, PPanGGOLiN, eggNOGmapper, mmseqs2, and anvi'o pangenome outputs
    into --cog_file-compatible inputs for mOTUpan.py
  repo: github.com/moritzbuck/mOTUlizer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:bb04c4e045e06c92f0c4812c549ceb39b5cc0cf786c383539c2ca9b513329769
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - MMseqs2
  - mOTUconvert.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# trait-presence-absence-pattern-modeling

## Summary

Iterative Bayesian classification of genome-encoded traits (e.g. gene clusters) as core or accessory by comparing the likelihood of each trait's presence/absence pattern under core vs accessory hypotheses, with completeness re-estimation, as implemented in mOTUpan.

## When to use

Use when computing core genomes from MAGs/SAGs of varying completeness: presence/absence patterns of gene clusters or traits across genomes must be partitioned into core vs accessory while accounting for incompleteness.

## When NOT to use

- Genomes are all high-quality/complete and a standard pangenome tool like Roary or PPanGGOLiN suffices; mOTUpan's value is robustness to incompleteness
- You need OTU-tables computed directly from reads — that is the unrelated 'mOTUs' tool, not mOTUlizer/mOTUpan
- Traits are quantitative or continuously distributed rather than discrete presence/absence genome-encoded traits

## Inputs

- Set of proteomes (.faa files) or precomputed gene clusters via --cog_file (JSON or TAB-separated genome-to-trait mapping)
- Completeness prior: CheckM output file (Bin Id, Completeness, Contamination columns, values 0-100) or a fixed value

## Outputs

- output.tsv with trait partitioning (core vs accessory) and updated genome completeness estimates
- Bootstrap statistics: false-positive rate, recall, and 'lowest false' frequency

## How to apply

Load a genome-to-trait mapping (JSON or TAB-separated cog file, e.g. --cog_file example_files/example_genome2cog.tsv) together with a completeness prior (e.g. CheckM output with Bin Id, Completeness, Contamination columns, or a fixed value). For each trait, compute the probability of its observed presence/absence pattern under the core hypothesis and under the accessory hypothesis using the Bayesian estimator (Equations 1-5), and classify it as core or accessory by the ratio of these likelihoods. Re-estimate each genome's completeness from the resulting core/accessory partitioning (Equation 6) and recompute the partitioning iteratively until convergence, capped by --max_iter. Optionally run --boots bootstraps, rarefying artificial genomes built from the partitioning (core genes fixed in all genomes, others sampled from a frequency-conserved gene pool) to estimate false-positive rate, recall, and 'lowest false'. Write the trait partitioning and updated completeness estimates to output.tsv via mOTUpan.py.

## Related tools

- **mOTUpan.py** (Runs the iterative Bayesian core/accessory likelihood loop, completeness re-estimation, and optional bootstrapping; writes output.tsv) — github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan; provides mOTUlize.py for OTU clustering, mOTUconvert.py for converting external pangenome outputs to mOTUpan inputs) — github.com/moritzbuck/mOTUlizer
- **MMseqs2** (Computes gene clusters (mmseqs easy_cluster) from input proteomes before trait classification) — soedinglab/MMseqs2
- **CheckM** (Source of completeness/contamination priors used for the Bayesian estimator and iterative re-estimation) — ecogenomics.github.io/CheckM
- **mOTUconvert.py** (Converts Roary, PPanGGOLiN, eggNOGmapper, mmseqs2, and anvi'o pangenome outputs into --cog_file-compatible inputs for mOTUpan.py) — github.com/moritzbuck/mOTUlizer

## Examples

```
mOTUpan.py --cog_file example_files/example_genome2cog.tsv --checkm example_files/example_checkm.txt --max_iter 50 --boots 100 -o output.tsv
```

## Evaluation signals

- The iterative loop converges (or terminates at --max_iter) rather than oscillating between partitionings
- Bootstrap reports a low false-positive rate with a high 'lowest false' value, meaning candidate false positives are highly prevalent traits
- Recall from bootstrapping is high, i.e. most traits classified as core in bootstraps match the original core partitioning
- Core-genome estimates are comparable to Roary and PPanGGOLiN when run on high-quality genomes
- Updated completeness estimates from Equation 6 are within 0-100 and consistent with the input prior direction

## Limitations

- Genomes with very low completeness degrade estimates; the method is designed for 'highly diverse completeness range' but the completeness prior quality matters
- Bootstrapping cost scales with --boots count; higher numbers give standard deviations for the false-positive rate, recall, and 'lowest false' but take longer
- Default clustering requires mmseqs2 installed (or --cog_file), and mOTUlize.py needs fastANI in PATH if similarities are not provided

## Evidence

- [abstract] Bayesian likelihood ratio iteration: "The likelihood for each gene cluster to belong to core or accessory genome is estimated by computing the probability of its presence/absence pattern in the target genome set."
- [methods] Iterative completeness update: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] Bootstrap rationale: "these genomes are then rarefied according to the posterioir completeness estimates. The bootstrap will provide an estimate of the false positive rate"
- [readme] Trait classification rule: "For each of these gene-clusters it will then compute the likelihood of it being in the core vs the likelihood of it not being, the ratio of these likelihoods"
