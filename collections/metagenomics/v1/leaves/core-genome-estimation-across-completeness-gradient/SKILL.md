---
name: core-genome-estimation-across-completeness-gradient
description: Use when comparing core-genome estimates from Roary, PPanGGOLiN, and
  mOTUpan over genome sets of varying (e.g. CheckM-estimated) completeness to locate
  where the tools' relative ordering reverses.
when_to_use_negative:
- The genome set is uniformly high quality (near-complete, low contamination) — for
  such high-quality genomes mOTUpan merely gives estimates comparable to Roary and
  PPanGGOLiN, so the completeness-gradient comparison adds little
- You need a tool-agnostic completeness correction only for one method — use mOTUpan's
  own --MAG-completeness/--SUB-completeness filters rather than the full three-tool
  benchmark
- The goal is metagenomic read-based OTU profiling — mOTUpan (mOTUlizer) is not the
  read-based mOTUs tool that creates OTU tables directly from reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: mOTUpan
  role: Iterative Bayesian core-genome estimator; run per completeness level with
    --cog_file on converted gene-cluster files plus a completeness prior, iterating
    until convergence
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan and mOTUconvert.py used to convert Roary/PPanGGOLiN
    outputs
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUconvert.py
  role: Converts Roary and PPanGGOLiN clustering outputs into input files for mOTUpan.py
    with --in_type set per source format
  repo: github.com/moritzbuck/mOTUlizer
- name: Roary
  role: Reference pangenome tool providing core-gene estimates with its default core-definition
    at each completeness level
  repo: sanger-pathogens.github.io/Roary
- name: PPanGGOLiN
  role: Synteny-network-based pangenome tool whose core estimates are compared against
    mOTUpan across the completeness scale
  repo: https://github.com/labgem/PPanGGOLiN
- name: mmseqs2
  role: Gene-clustering engine used by mOTUpan.py when proteomes rather than pre-computed
    gene clusters are supplied
  repo: https://github.com/soedinglab/MMseqs2
- name: CheckM
  role: Source of the completeness/contamination file format used as mOTUpan's completeness
    prior
  repo: https://ecogenomics.github.io/CheckM/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:d2302425400827c5047336c39da8eabcff9c1619ea2eaeece80eae9010660735
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
  - mmseqs2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# core-genome-estimation-across-completeness-gradient

## Summary

Reproduces the completeness-scaled benchmark in which Roary, PPanGGOLiN, and mOTUpan core-genome estimates are tabulated against average genome completeness, locating the ~70% completeness point where PPanGGOLiN's and mOTUpan's estimates cross over and mOTUpan's become larger.

## When to use

Use when comparing core-genome estimates from Roary, PPanGGOLiN, and mOTUpan over genome sets of varying (e.g. CheckM-estimated) completeness to locate where the tools' relative ordering reverses.

## When NOT to use

- The genome set is uniformly high quality (near-complete, low contamination) — for such high-quality genomes mOTUpan merely gives estimates comparable to Roary and PPanGGOLiN, so the completeness-gradient comparison adds little
- You need a tool-agnostic completeness correction only for one method — use mOTUpan's own --MAG-completeness/--SUB-completeness filters rather than the full three-tool benchmark
- The goal is metagenomic read-based OTU profiling — mOTUpan (mOTUlizer) is not the read-based mOTUs tool that creates OTU tables directly from reads

## Inputs

- Genome (MAG/SAG) sets at multiple average-completeness levels (e.g. derived by rarefying complete genome sets)
- Roary clustering output (gene-cluster presence/absence per genome)
- PPanGGOLiN clustering output (synteny-network-based gene clusters)
- Proteome FASTA files (.faa) for mOTUpan's internal mmseqs2 clustering if clusters are not pre-computed
- CheckM-style completeness/contamination file with Bin Id, Completeness, and Contamination columns (or a fixed completeness value)

## Outputs

- mOTUpan-compatible gene-cluster files produced by mOTUconvert.py
- Per-completeness-level core-genome size estimates from Roary, PPanGGOLiN, and mOTUpan
- Tabulated curves of core-genome estimate versus average completeness showing Roary's steep decline and the PPanGGOLiN/mOTUpan crossover near 70% completeness

## How to apply

Convert Roary and PPanGGOLiN clustering outputs into mOTUpan-compatible gene-cluster files with mOTUconvert.py, setting --in_type to match each tool's output format. Run Roary with its default core-definition on each genome set at every completeness level, and run mOTUpan.py with --cog_file on the converted files plus a CheckM-style completeness file (Bin Id, Completeness, Contamination columns), letting the iterative Bayesian re-estimation of the completeness prior converge under the default --max_iter. Extract core-genome sizes from all three tools' outputs at each average-completeness value and tabulate estimates versus completeness. Judge success by the qualitative validation: Roary's core estimate declines steeply as completeness drops, while the PPanGGOLiN and mOTUpan curves cross over near 70% average completeness — mOTUpan exceeding PPanGGOLiN below and PPanGGOLiN exceeding mOTUpan above that point.

## Related tools

- **mOTUpan** (Iterative Bayesian core-genome estimator; run per completeness level with --cog_file on converted gene-cluster files plus a completeness prior, iterating until convergence) — github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan and mOTUconvert.py used to convert Roary/PPanGGOLiN outputs) — github.com/moritzbuck/mOTUlizer
- **mOTUconvert.py** (Converts Roary and PPanGGOLiN clustering outputs into input files for mOTUpan.py with --in_type set per source format) — github.com/moritzbuck/mOTUlizer
- **Roary** (Reference pangenome tool providing core-gene estimates with its default core-definition at each completeness level) — sanger-pathogens.github.io/Roary
- **PPanGGOLiN** (Synteny-network-based pangenome tool whose core estimates are compared against mOTUpan across the completeness scale) — https://github.com/labgem/PPanGGOLiN
- **mmseqs2** (Gene-clustering engine used by mOTUpan.py when proteomes rather than pre-computed gene clusters are supplied) — https://github.com/soedinglab/MMseqs2
- **CheckM** (Source of the completeness/contamination file format used as mOTUpan's completeness prior) — https://ecogenomics.github.io/CheckM/

## Examples

```
mOTUpan.py --cog_file ppanggolin_clusters.tsv -o motupan_core.tsv
```

## Evaluation signals

- Roary's core estimate drops sharply with decreasing average completeness, reproducing its known sensitivity to missing genes
- The tabulated PPanGGOLiN and mOTUpan curves cross over at approximately 70% average completeness, with mOTUpan larger below and PPanGGOLiN larger above
- For high-quality (high-completeness) genome sets, mOTUpan's estimates are comparable to Roary and PPanGGOLiN
- The mOTUpan iterative re-estimation of the completeness prior converges within the default --max_iter for every completeness level
- mOTUconvert.py output files are accepted by mOTUpan.py --cog_file without schema errors for both Roary and PPanGGOLiN input types

## Limitations

- mOTUpan requires a completeness prior (CheckM-style file or fixed value); the crossover behaviour is assessed relative to that prior's accuracy
- fastANI similarity computation is the slowest part of the mOTUlizer workflow and is memory-hungry for large genome sets
- anvi-run-motupan.py is explicitly experimental and has fewer options than mOTUpan.py
- The bootstrap quality estimates (--boots) should be used to check that core calls at low completeness are not dominated by false positives

## Evidence

- [abstract] mOTUpan as iterative Bayesian core estimator for diverse-completeness sets: "We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range."
- [abstract] Comparability with Roary and PPanGGOLiN for high-quality genomes: "It has shown comparable estimates to state-of-the-art tools Roary and PPanGGOLiN for high-quality genomes and is capable of using genomes at lower completeness thresholds."
- [methods] mOTUconvert.py role in the benchmark workflow: "mOTUconvert.py` converts the output of diverse programs into input files for `mOTUpan.py"
- [methods] --cog_file option for providing external gene clusters: "`--cog_file` : can be used as an alternative to `--faas`, you can use it to provide you own gene-clusters (or other genetically encoded traits)."
- [readme] Iterative convergence of the completeness prior: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] CheckM output as the completeness prior format: "well as a completeness prior of these genomes (for example [`checkm`](https://ecogenomics.github.io/CheckM/) output or a fixed value)"
