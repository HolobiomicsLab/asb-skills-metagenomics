---
name: core-gene-recovery-analysis
description: Use when you need to quantify how genome incompleteness (e.g. CheckM
  completeness of MAGs) influences mOTUpan core-genome predictions, including false-positive
  and recall behaviour under bootstrap estimation.
when_to_use_negative:
- You need core-genome estimates from raw protein FASTA files without custom gene
  clusters — use mOTUpan's default --faas + mmseqs2 clustering workflow instead of
  --cog_file simulations
- Genome sets contain only high-quality complete genomes — mOTUpan's robustness-to-incompleteness
  analysis (bad genomes <45%) is not the relevant regime
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan.py
  role: Computes the Bayesian core/accessory partitioning from --cog_file gene-trait
    dictionaries and a --checkm completeness prior, iterated to convergence, with
    --boots for FPR/recall estimation
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan; installable via conda (bioconda motulizer)
    or pip (mOTUlizer)
  repo: github.com/moritzbuck/mOTUlizer
- name: CheckM
  role: Source of the genome completeness estimates supplied as the three-column (Bin
    Id, Completeness, Contamination) prior for mOTUpan
  repo: ecogenomics.github.io/CheckM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:9e851e12cd717cd3e6e36fca0c417917ea8a4c70f0c81e72e92f05724ca6400d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# core-gene-recovery-analysis

## Summary

Reproduce the mOTUpan idealized-runs analysis showing how one 'good' genome (>45% completeness) plus variable numbers of 'bad' genomes (<45%) affect core-gene recovery and the false-positive rate of the Bayesian core-genome prediction.

## When to use

Use when you need to quantify how genome incompleteness (e.g. CheckM completeness of MAGs) influences mOTUpan core-genome predictions, including false-positive and recall behaviour under bootstrap estimation.

## When NOT to use

- You need core-genome estimates from raw protein FASTA files without custom gene clusters — use mOTUpan's default --faas + mmseqs2 clustering workflow instead of --cog_file simulations
- Genome sets contain only high-quality complete genomes — mOTUpan's robustness-to-incompleteness analysis (bad genomes <45%) is not the relevant regime

## Inputs

- Simulated genome sets as gene-trait presence/absence dictionaries in mOTUpan --cog_file JSON format
- CheckM-like three-column completeness file (Bin Id, Completeness, Contamination)

## Outputs

- simulated_core_genome_results.tsv (per-run metrics: recovered core genes vs good-genome completeness, effect of number of bad genomes on core genes and FPR, bootstrapped FPR vs TPR)
- figure_s2_reproduction.png (Supplementary Figure S2-style panels)

## How to apply

Generate ~10000 simulated genome sets as gene-trait presence/absence dictionaries in the mOTUpan --cog_file JSON format, each with one good genome above 45% completeness and a variable number of bad genomes below 45%, sampling core genes as present in all genomes and accessory genes from a frequency-conserved pool. Write simulated completeness values in a CheckM-like three-column format (Bin Id, Completeness, Contamination) and run mOTUpan.py with --cog_file and --checkm so the Bayesian estimator computes, for each trait, the likelihood of being core versus not core and iterates the partitioning to convergence (set --max_iter suitably for small trait sets). Run the bootstrap option (--boots) so artificial genomes are built from the inferred gene-partitioning — core genes in all artificial genomes, others from a gene-pool with conserved frequency — and rarefied by posterior completeness estimates, extracting false-positive rate, recall and 'lowest false' metrics. Tabulate per-run metrics as simulated_core_genome_results.tsv and visualize core-gene recovery vs good-genome completeness, core genes vs number of bad genomes, and bootstrapped FPR vs TPR as Supplementary Figure S2-style panels (figure_s2_reproduction.png).

## Related tools

- **mOTUpan.py** (Computes the Bayesian core/accessory partitioning from --cog_file gene-trait dictionaries and a --checkm completeness prior, iterated to convergence, with --boots for FPR/recall estimation) — github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan; installable via conda (bioconda motulizer) or pip (mOTUlizer)) — github.com/moritzbuck/mOTUlizer
- **CheckM** (Source of the genome completeness estimates supplied as the three-column (Bin Id, Completeness, Contamination) prior for mOTUpan) — ecogenomics.github.io/CheckM

## Examples

```
mOTUpan.py --cog_file simulated_genomes. --checkm simulated_checkm.tsv --boots 100 --max_iter 50 -o output.tsv
```

## Evaluation signals

- Core-gene recovery of the good genome is controlled by its own completeness (number of recovered core genes scales with good-genome completeness)
- Increasing the number of bad genomes (<45% completeness) reduces the number of core genes retrieved by mOTUpan
- Increasing the number of bad genomes can also decrease the false-positive rate of the core-genome prediction
- The bootstrapped FPR predicts the true-positive rate (FPR vs TPR relationship reproduces Supplementary Figure S2C)
- Bootstrap outputs include the three documented metrics: false-positive rate, recall, and 'lowest false'

## Limitations

- Simulations rely on a frequency-conserved accessory-gene pool and CheckM-like completeness priors, so reproduced trends are qualitative, not exact quantitative matches to biological pangenomes
- The bootstrap FPR is an estimate — 'Higher number of bootstraps give you a standard deviation for these numbers' — so a single low --boots value yields noisy FPR/recall
- The Bayesian partitioning requires iterative recomputation until convergence; --max_iter must be set appropriately for small trait sets or the partitioning may not converge

## Evidence

- [abstract] We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range.: "We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range."
- [readme] bootstraps generate artificial genomes rarefied by posterior completeness: "runs `BOOTS` bootstraps, where artificial genomes are generated using the gene-partitioning obtained with `mOTUpan`"
- [readme] bootstrap metrics definition: "The bootstrap will provide an estimate of the false positive rate (e.g. fraction of core genes that might not be), the recall (fraction of core genes that have been classified as such in the"
- [readme] iterative recomputation until convergence: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] --cog_file accepts JSON gene-trait dictionaries: "`--cog_file` : can be used as an alternative to `--faas`, you can use it to provide you own gene-clusters (or other genetically encoded traits)."
