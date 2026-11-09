---
name: genome-completeness-prior-simulation
description: Use when you need to benchmark or stress-test a core-genome estimator
  (e.g. mOTUpan) against simulated MAG/SAG incompleteness, or quantify how low-completeness
  genomes bias core-gene calls and bootstrap false-positive rates.
when_to_use_negative:
- The goal is a real core-genome estimate on empirical genomes (e.g. the 388-genome
  Prochlorococcus_A mOTU) — run mOTUpan directly on the faa/cog inputs instead of
  simulating
- Genomes are all high-quality (>95% complete, including closed genomes); incompleteness
  robustness is not the limiting factor and benchmarking against PPanGGOLiN/Roary
  is more informative
- Input is already a gene-cluster presence/absence table from another pangenome tool
  — use mOTUconvert.py rather than simulating new gene-trait sets
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan
  role: Runs the iterative Bayesian core/accessory partitioning on simulated cog_file
    and CheckM inputs; --boots supplies bootstrap FPR, recall and 'lowest false' metrics
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUpan.py
  role: CLI entry point invoked with --cog_file, --checkm, --max_iter and --boots
    for each simulated genome set
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan, installed via conda/pip or from source
  repo: github.com/moritzbuck/mOTUlizer
- name: CheckM
  role: Source of the three-column completeness/contamination file format (Bin Id,
    Completeness, Contamination) emulated by the simulated quality table
  repo: ecogenomics.github.io/CheckM
provenance:
  source_task_ids:
  - task_007
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:721c6e65cafb3ccab225b5625e8e1001f403da37f2012f38266d2c1484a4fc4c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - PPanGGOLiN
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
yes_rate: 0.625
n_claims: 8
---

# genome-completeness-prior-simulation

## Summary

Simulates idealized genome sets (one 'good' genome >45% completeness plus variable numbers of 'bad' genomes <45% completeness) as gene-trait presence/absence dictionaries in the mOTUpan --cog_file JSON format, to quantify how genome incompleteness and the number of low-quality genomes affect core-gene recovery and the false-positive rate of mOTUpan's Bayesian core-genome prediction.

## When to use

Use when you need to benchmark or stress-test a core-genome estimator (e.g. mOTUpan) against simulated MAG/SAG incompleteness, or quantify how low-completeness genomes bias core-gene calls and bootstrap false-positive rates.

## When NOT to use

- The goal is a real core-genome estimate on empirical genomes (e.g. the 388-genome Prochlorococcus_A mOTU) — run mOTUpan directly on the faa/cog inputs instead of simulating
- Genomes are all high-quality (>95% complete, including closed genomes); incompleteness robustness is not the limiting factor and benchmarking against PPanGGOLiN/Roary is more informative
- Input is already a gene-cluster presence/absence table from another pangenome tool — use mOTUconvert.py rather than simulating new gene-trait sets

## Inputs

- Simulated genome sets as gene-trait presence/absence dictionaries in mOTUpan --cog_file JSON format (one 'good' genome >45% completeness + variable number of 'bad' genomes <45%)
- CheckM-like three-column completeness/contamination table (Bin Id, Completeness, Contamination) with simulated completeness values, values 0-100

## Outputs

- simulated_core_genome_results.tsv — per-run table of recovered core genes vs good-genome completeness, effect of number of bad genomes on recovered core genes and false-positive rate, and bootstrapped FPR vs TPR
- figure_s2_reproduction.png — panels: core-gene recovery vs completeness of the good genome; core genes vs number of bad genomes; bootstrapped FPR vs TPR (Supplementary Figure S2 style)
- Per-run mOTUpan partitioning and bootstrap metrics (false-positive rate, recall, 'lowest false')

## How to apply

Generate thousands (e.g. 10000) of simulated genome sets as gene-trait presence/absence dictionaries in the mOTUpan --cog_file JSON format, each containing one 'good' genome with completeness above 45% and a variable number of 'bad' genomes below 45%, sampling core genes as present in all genomes and accessory genes from a frequency-conserved pool. Write a CheckM-like three-column file (Bin Id, Completeness, Contamination) with the simulated completeness values and run mOTUpan with --cog_file and --checkm so the iterative Bayesian estimator computes each trait's core-vs-non-core likelihood and re-estimates the completeness prior until convergence, setting --max_iter appropriately for the small trait sets. Run the bootstrap option (--boots) per simulation, where artificial genomes are built from the inferred gene-partitioning (core genes in all artificial genomes, others from a gene-pool with conserved frequency) and rarefied by posterior completeness estimates, extracting false-positive rate, recall, and 'lowest false' metrics. Tabulate per-run metrics — recovered core genes vs good-genome completeness, core genes vs number of bad genomes, and bootstrapped FPR vs true-positive rate — and visualize as panels matching the paper's Supplementary Figure S2. The rationale is that mOTUpan's Bayesian estimator is 'very robust to incompleteness', and simulation isolates the marginal effect of each bad genome on partitioning and FPR.

## Related tools

- **mOTUpan** (Runs the iterative Bayesian core/accessory partitioning on simulated cog_file and CheckM inputs; --boots supplies bootstrap FPR, recall and 'lowest false' metrics) — github.com/moritzbuck/mOTUlizer
- **mOTUpan.py** (CLI entry point invoked with --cog_file, --checkm, --max_iter and --boots for each simulated genome set) — github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan, installed via conda/pip or from source) — github.com/moritzbuck/mOTUlizer
- **CheckM** (Source of the three-column completeness/contamination file format (Bin Id, Completeness, Contamination) emulated by the simulated quality table) — ecogenomics.github.io/CheckM

## Examples

```
mOTUpan.py --cog_file simulated_genome_set. --checkm simulated_checkm.txt --max_iter 100 --boots 100 -o output.tsv
```

## Evaluation signals

- Core-gene recovery of the 'good' genome is controlled primarily by its completeness (Supplementary Figure S2A trend reproduced)
- Increasing the number of 'bad' genomes (<45% completeness) reduces the number of core genes retrieved by mOTUpan (Supplementary Figure S2B trend)
- Increasing the number of 'bad' genomes can also decrease the false-positive rate of the core-genome prediction (Supplementary Figure S2C trend)
- Bootstrapped FPR predicts the true-positive rate: bootstrap reports a low false-positive rate with a high 'lowest false' value, meaning candidate false positives are highly prevalent traits
- The iterative Bayesian loop converges (or hits --max_iter) for each of the 10000 simulated runs

## Limitations

- Simulation results are qualitative trends (Supplementary Figure S2), not absolute core-gene counts transferable to empirical genome sets
- Traits are simulated from a frequency-conserved accessory-gene pool, so real gene-frequency distributions may deviate
- Small trait sets require a manually set --max_iter since convergence is capped rather than guaranteed

## Evidence

- [abstract] The likelihood for each gene cluster to belong to core or accessory genome is estimated by computing the probability of its presence/absence pattern in the target genome set.: "The likelihood for each gene cluster to belong to core or accessory genome is estimated by computing the probability of its presence/absence pattern in the target genome set."
- [abstract] mOTUpan wraps a bootstrapping procedure to estimate the quality of a specific core-genome prediction: "mOTUpan wraps a bootstrapping procedure to estimate the quality of a specific core-genome prediction"
- [methods] This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence.: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] --cog_file : can be used as an alternative to --faas, you can use it to provide you own gene-clusters (or other genetically encoded traits).: "--cog_file : can be used as an alternative to --faas, you can use it to provide you own gene-clusters (or other genetically encoded traits)."
- [readme] --boots BOOTS : runs BOOTS bootstraps, where artificial genomes are generated using the gene-partitioning obtained with mOTUpan: "--boots BOOTS : runs BOOTS bootstraps, where artificial genomes are generated using the gene-partitioning obtained with `mOTUpan`"
