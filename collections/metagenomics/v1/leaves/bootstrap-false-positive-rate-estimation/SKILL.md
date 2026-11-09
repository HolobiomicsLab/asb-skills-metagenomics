---
name: bootstrap-false-positive-rate-estimation
description: Use when a core-genome prediction from incomplete genomes (MAGs/SAGs)
  needs a quality estimate — e.g. to flag problematic genome combinations or to validate
  a bootstrapped core call against an empirical 'true' core.
when_to_use_negative:
- The genome set consists of high-quality complete genomes where established tools
  like Roary or PPanGGOLiN suffice and completeness priors are unnecessary.
- You only need gene clustering or OTU grouping (use mOTUlize.py or mmseqs2 alone)
  without a core-genome quality assessment.
- The input is a raw similarity/ANI file not yet converted into mOTUpan trait format
  — convert first with mOTUconvert.py before bootstrapping.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan
  role: Performs the iterative Bayesian core/accessory partitioning and, with --boots,
    generates artificial rarefied genomes and the false-positive-rate, recall, and
    'lowest false' metrics.
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUpan.py
  role: Command-line entry point invoked with --faas/--cog_file, --checkm, and --boots
    to run the bootstrapping quality estimation.
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan and its bootstrapping procedure.
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mmseqs2
  role: Computes gene clusters (easy_cluster) from input proteomes as the traits that
    the bootstrapping evaluates.
  repo: https://github.com/soedinglab/MMseqs2
- name: mOTUconvert.py
  role: Converts outputs of mmseqs2, Roary, PPanGGOLiN, eggNOGmapper, or anvi'o into
    --cog_file input files for mOTUpan.py.
  repo: https://github.com/moritzbuck/mOTUlizer
- name: CheckM
  role: Source of completeness (and contamination) priors supplied via --checkm, used
    to rarefy artificial bootstrap genomes by posterior completeness estimates.
  repo: https://ecogenomics.github.io/CheckM/
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:275dac103368ac8bed6c72a7565e2eacc1ddbb4ced45be0f8c7eec8c69e5774c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# bootstrap-false-positive-rate-estimation

## Summary

Estimate the quality of an mOTUpan core-genome prediction by generating artificial rarefied genomes from the inferred core/accessory partitioning, re-classifying them, and deriving a bootstrapped false-positive rate, recall, and 'lowest false' statistic that predicts the true accuracy of the core-genome call.

## When to use

Use when a core-genome prediction from incomplete genomes (MAGs/SAGs) needs a quality estimate — e.g. to flag problematic genome combinations or to validate a bootstrapped core call against an empirical 'true' core.

## When NOT to use

- The genome set consists of high-quality complete genomes where established tools like Roary or PPanGGOLiN suffice and completeness priors are unnecessary.
- You only need gene clustering or OTU grouping (use mOTUlize.py or mmseqs2 alone) without a core-genome quality assessment.
- The input is a raw similarity/ANI file not yet converted into mOTUpan trait format — convert first with mOTUconvert.py before bootstrapping.

## Inputs

- mOTUpan gene-cluster partitioning (core vs non-core gene clusters), supplied as proteomes via --faas with mmseqs2 clustering or a pre-computed --cog_file (JSON presence/absence dictionary, e.g. from mOTUconvert.py)
- Per-genome completeness priors as a CheckM-like three-column file (Bin Id, Completeness, Contamination) via --checkm, or fixed values via --MAG-completeness
- Number of bootstrap replicates (BOOTS integer)

## Outputs

- Bootstrap quality statistics per replicate: false-positive rate, recall, and 'lowest false' frequency, with standard deviations over replicates (e.g. written to bootstrap_stats.tsv)
- For validation studies, a table joining empirical and bootstrapped false-positive-rate columns per run (Supplementary Table S3 / Figure S1 style)

## How to apply

After running mOTUpan.py's iterative Bayesian core/accessory likelihood estimation to convergence, invoke the --boots BOOTS flag: BOOTS artificial genomes are generated with all core gene clusters placed in every artificial genome and remaining genes sampled from a gene-pool conserving their observed frequencies. Each artificial genome is rarefied according to the posterior completeness estimates (e.g. from a CheckM --checkm file) so incomplete MAGs are mimicked, then re-classified with the core-likelihood classifier. For each bootstrap replicate compute the false-positive rate (fraction of core genes that might not be core), the recall (fraction of core genes reclassified as core), and 'lowest false' (lowest frequency of any false-positive gene cluster, expected to be high). Aggregate over replicates for standard deviations; the bootstrapped FPR predicts the empirical FPR obtained by classifying genes against a liberal 'true' core (e.g. the union of core genes over 10 runs on 100 sampled Prochlorococcus_A genomes), so a high bootstrapped FPR flags unreliable genome-set combinations.

## Related tools

- **mOTUpan** (Performs the iterative Bayesian core/accessory partitioning and, with --boots, generates artificial rarefied genomes and the false-positive-rate, recall, and 'lowest false' metrics.) — https://github.com/moritzbuck/mOTUlizer
- **mOTUpan.py** (Command-line entry point invoked with --faas/--cog_file, --checkm, and --boots to run the bootstrapping quality estimation.) — https://github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan and its bootstrapping procedure.) — https://github.com/moritzbuck/mOTUlizer
- **mmseqs2** (Computes gene clusters (easy_cluster) from input proteomes as the traits that the bootstrapping evaluates.) — https://github.com/soedinglab/MMseqs2
- **mOTUconvert.py** (Converts outputs of mmseqs2, Roary, PPanGGOLiN, eggNOGmapper, or anvi'o into --cog_file input files for mOTUpan.py.) — https://github.com/moritzbuck/mOTUlizer
- **CheckM** (Source of completeness (and contamination) priors supplied via --checkm, used to rarefy artificial bootstrap genomes by posterior completeness estimates.) — https://ecogenomics.github.io/CheckM/

## Examples

```
mOTUpan.py --cog_file cog_clusters. --checkm checkm_output.tsv --boots 100 -o bootstrap_stats.tsv
```

## Evaluation signals

- All BOOTS replicates complete and yield finite, interpretable values for false-positive rate, recall, and 'lowest false' per bootstrap.
- All bootstrapped FPR values fall in the range [0,1].
- 'lowest false' is high, indicating candidate false positives are highly prevalent in the genome set.
- In validation mode, the output table contains both empirical and bootstrapped false-positive-rate columns per run, and the bootstrapped FPR correlates with the empirical FPR (Supplementary Table S3 / Figure S1 relationship).
- In simulated idealized runs, increasing the number of 'bad' (<45% completeness) genomes reduces recovered core genes while potentially lowering the FPR, and bootstrapped FPR predicts true-positive rate (Supplementary Figure S2 trends).

## Limitations

- The bootstrap provides a quality estimate, not a guarantee — a low bootstrapped FPR is needed but the empirical accuracy still depends on the reference ('true') core definition, e.g. the liberal union-of-10-runs core.
- The 'lowest false' statistic should be high; low values indicate candidate false positives that are not highly prevalent, weakening confidence in the partitioning.
- Genome-set composition matters: adding more low-completeness (<45%) 'bad' genomes reduces the number of core genes retrieved, even though it can paradoxically decrease the false-positive rate.
- Higher numbers of bootstraps are needed to obtain standard deviations for the three quality metrics.

## Evidence

- [abstract] mOTUpan wraps a bootstrapping procedure to estimate the quality of a specific core-genome prediction: "mOTUpan wraps a bootstrapping procedure to estimate the quality of a specific core-genome prediction"
- [readme] bootstrapping builds synthetic genomes and estimates FPR and sensitivity: "`--boots BOOTS` : runs `BOOTS` bootstraps, where artificial genomes are generated using the gene-partitioning obtained with `mOTUpan`"
- [readme] bootstrap metrics definition from README: "The bootstrap will provide an estimate of the false positive rate (e.g. fraction of core genes that might not be), the recall (fraction of core genes that have been classified as such in the"
- [readme] iterative convergence of the Bayesian partitioning: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [results] bootstrapped FPR predicts true accuracy (source card finding): [evidence span withheld — anchor status: not_found]
