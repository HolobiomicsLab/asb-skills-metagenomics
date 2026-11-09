---
name: core-size-trend-comparison-across-tools
description: Use when benchmarking core-genome estimation tools on genome sets of
  varying (especially low) completeness, to characterize how each tool's core estimate
  degrades along a completeness scale.
when_to_use_negative:
- Genome sets are high-quality/complete genomes, where Roary, PPanGGOLiN, and mOTUpan
  give comparable estimates and no crossover analysis is informative
- You only need a single core-genome estimate for one genome set rather than a trend
  across a completeness scale
- The clustering outputs are not from a format supported by mOTUconvert.py (mmseqs2,
  Roary, PPanGGOLiN, eggNOGmapper, anvi'o)
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Roary
  role: Produces presence/absence core-genome estimates at each completeness level
    using its default core-definition
  repo: https://sanger-pathogens.github.io/Roary/
- name: PPanGGOLiN
  role: Provides synteny-network-based gene clustering whose core estimates are converted
    and compared against mOTUpan along the completeness scale
  repo: https://github.com/labgem/PPanGGOLiN
- name: mOTUpan
  role: Bayesian iterative core-genome estimator run on converted gene-cluster files
    with a completeness prior, benchmarked against Roary and PPanGGOLiN
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUconvert.py
  role: Converts Roary and PPanGGOLiN outputs into mOTUpan-compatible gene-cluster
    input files
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUpan.py
  role: Executes mOTUpan with --cog_file and a completeness file, iterating the completeness
    prior to convergence
  repo: https://github.com/moritzbuck/mOTUlizer
- name: CheckM
  role: Source of the completeness/contamination file used as the mOTUpan completeness
    prior
  repo: https://ecogenomics.github.io/CheckM/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:321b3a5a5d5f79bb33e4a55327db4795ad68f7cd03313905485529a56cc6912c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - PPanGGOLiN
  - Roary
  - mOTUconvert.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# core-size-trend-comparison-across-tools

## Summary

Reproduce a completeness-scaled benchmark that tabulates core-genome size estimates from Roary, PPanGGOLiN, and mOTUpan across genome sets of decreasing average completeness, revealing a crossover in PPanGGOLiN vs mOTUpan estimates near 70% completeness.

## When to use

Use when benchmarking core-genome estimation tools on genome sets of varying (especially low) completeness, to characterize how each tool's core estimate degrades along a completeness scale.

## When NOT to use

- Genome sets are high-quality/complete genomes, where Roary, PPanGGOLiN, and mOTUpan give comparable estimates and no crossover analysis is informative
- You only need a single core-genome estimate for one genome set rather than a trend across a completeness scale
- The clustering outputs are not from a format supported by mOTUconvert.py (mmseqs2, Roary, PPanGGOLiN, eggNOGmapper, anvi'o)

## Inputs

- Input genome sets subsampled at each completeness level along a completeness scale
- Roary clustering output per genome set (converted via mOTUconvert.py)
- PPanGGOLiN clustering output per genome set (converted via mOTUconvert.py)
- checkm-style completeness file with Bin Id, Completeness, and Contamination columns (or fixed values)

## Outputs

- Tabulated core-genome size estimates for Roary, PPanGGOLiN, and mOTUpan at each average-completeness value
- Completeness-scaled comparison curves showing Roary's sharp decline and the PPanGGOLiN/mOTUpan crossover near 70% average completeness

## How to apply

Convert Roary and PPanGGOLiN clustering outputs into mOTUpan-compatible gene-cluster files using mOTUconvert.py with the appropriate --in_type for each tool's output format. Run Roary on each input genome set at every completeness level using its default core-definition, and run mOTUpan.py with --cog_file on the converted gene-cluster files plus a checkm-style completeness file (Bin Id, Completeness, Contamination columns), letting the Bayesian iterative re-estimation of the completeness prior converge with the default --max_iter. Extract core-genome sizes from Roary, PPanGGOLiN, and mOTUpan outputs at each average-completeness value and tabulate the three tools' estimates versus average completeness. The rationale is that Roary's presence/absence core-definition drops sharply with decreasing completeness, whereas mOTUpan's Bayesian estimator compensates for missing genes via a completeness prior.

## Related tools

- **Roary** (Produces presence/absence core-genome estimates at each completeness level using its default core-definition) — https://sanger-pathogens.github.io/Roary/
- **PPanGGOLiN** (Provides synteny-network-based gene clustering whose core estimates are converted and compared against mOTUpan along the completeness scale) — https://github.com/labgem/PPanGGOLiN
- **mOTUpan** (Bayesian iterative core-genome estimator run on converted gene-cluster files with a completeness prior, benchmarked against Roary and PPanGGOLiN) — https://github.com/moritzbuck/mOTUlizer
- **mOTUconvert.py** (Converts Roary and PPanGGOLiN outputs into mOTUpan-compatible gene-cluster input files) — https://github.com/moritzbuck/mOTUlizer
- **mOTUpan.py** (Executes mOTUpan with --cog_file and a completeness file, iterating the completeness prior to convergence) — https://github.com/moritzbuck/mOTUlizer
- **CheckM** (Source of the completeness/contamination file used as the mOTUpan completeness prior) — https://ecogenomics.github.io/CheckM/

## Examples

```
mOTUpan.py --cog_file ppanggolin_clusters.tsv --completeness checkm_output.tsv -o output.tsv
```

## Evaluation signals

- Roary's core estimate drops sharply with decreasing average completeness across the tabulated scale
- PPanGGOLiN's core estimates exceed mOTUpan's above ~70% average completeness, but mOTUpan's estimates become larger once average completeness drops below 70%
- mOTUpan's iterative completeness-prior re-estimation converges within the default --max_iter for each completeness level
- The tabulated core estimates per completeness level reproduce the reported qualitative trends (steep Roary decline; PPanGGOLiN/mOTUpan crossover near 70%)

## Limitations

- The PPanGGOLiN/mOTUpan crossover is specific to the completeness scale used; below ~70% average completeness the ranking of the two tools reverses
- mOTUpan assumes a meaningful completeness prior (e.g. checkm estimates or a fixed value); unreliable completeness values propagate into the core-genome partitioning
- Roary's default core-definition is very sensitive to missing genes and fragmented genomes, making it unsuitable as a reference at low completeness

## Evidence

- [results] PPanGGOLiN exceeds mOTUpan above ~70% completeness: [evidence span withheld — anchor status: not_found]
- [abstract] mOTUpan Bayesian method: "We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range."
- [methods] iterative completeness-prior convergence: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] mOTUconvert converts tool outputs: "* `mOTUconvert.py` converts the output of diverse programs into input files for `mOTUpan.py`, currently includes methods for [`mmseqs2`](https://github.com/soedinglab/MMseqs2),"
- [methods] --cog_file usage: "`--cog_file` : can be used as an alternative to `--faas`, you can use it to provide you own gene-clusters (or other genetically encoded traits)."
