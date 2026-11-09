---
name: completeness-prior-iterative-refinement
description: Use when estimating core genomes from genome sets of mixed or low quality
  (e.g. CheckM-estimated MAG completeness along a completeness scale) and you need
  the completeness prior refined from the data rather than fixed.
when_to_use_negative:
- Genome set consists only of high-quality complete genomes where simpler presence/absence
  tools like Roary's default core-definition suffice
- You need OTU clustering and genome quality statistics rather than core-genome partitioning
  — use mOTUlize.py instead
- You have no completeness prior available (no CheckM output or fixed value) and cannot
  justify the Bayesian prior assumption
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan
  role: Performs the iterative Bayesian core/accessory likelihood computation with
    completeness-prior refinement
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUconvert.py
  role: Converts Roary, PPanGGOLiN, mmseqs2, eggNOGmapper or anvio clustering outputs
    into mOTUpan-compatible --cog_file inputs
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mmseqs2
  role: Computes gene clusters (easy_cluster) when mOTUpan is run from proteomes instead
    of a precomputed cog_file
  repo: https://github.com/soedinglab/MMseqs2
- name: CheckM
  role: Provides the completeness/contamination table used as the initial completeness
    prior
  repo: https://ecogenomics.github.io/CheckM/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:1ddb6e1a767d3befb9f4402104ab7a8067446fb65e3dfef2821e639b61812deb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/moritzbuck/mOTUlizer
  tools:
  - CheckM
  - PPanGGOLiN
  - Roary
  - mOTUconvert.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  - mmseqs2
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# completeness-prior-iterative-refinement

## Summary

An iterative Bayesian scheme in mOTUpan where the gene-cluster core/accessory partitioning is used to update each genome's completeness prior, which is then re-fed into the likelihood computation until convergence, making core-genome estimation robust to incomplete MAGs/SAGs.

## When to use

Use when estimating core genomes from genome sets of mixed or low quality (e.g. CheckM-estimated MAG completeness along a completeness scale) and you need the completeness prior refined from the data rather than fixed.

## When NOT to use

- Genome set consists only of high-quality complete genomes where simpler presence/absence tools like Roary's default core-definition suffice
- You need OTU clustering and genome quality statistics rather than core-genome partitioning — use mOTUlize.py instead
- You have no completeness prior available (no CheckM output or fixed value) and cannot justify the Bayesian prior assumption

## Inputs

- Gene-cluster file (JSON or TAB-separated) supplied via --cog_file, e.g. converted from Roary/PPanGGOLiN/mmseqs2/anvio output with mOTUconvert.py
- CheckM-style completeness/contamination table (columns: Bin Id, Completeness, Contamination) or a fixed completeness value
- Optional proteome FASTA files (.faa) if mOTUpan should cluster genes itself using mmseqs2

## Outputs

- Tab-separated output file with gene-cluster core/accessory likelihood-based partitioning (via -o)
- Per-genome posterior completeness estimates updated during iteration
- Core-genome size estimates comparable against Roary and PPanGGOLiN along a completeness scale

## How to apply

Provide mOTUpan.py with gene clusters (--cog_file, e.g. converted from Roary or PPanGGOLiN output via mOTUconvert.py) plus a checkm-style completeness file containing Bin Id, Completeness and Contamination columns. mOTUpan computes, for each gene cluster, the likelihood of its presence/absence pattern being core versus accessory, and the ratio of these likelihoods determines core membership. The resulting partitioning is then used to update the completeness prior, and the core-trait likelihoods are recomputed iteratively until convergence, controlled by the --max_iter flag. For reproducible comparisons along a completeness scale (e.g. testing where PPanGGOLiN and mOTUpan estimates cross over near 70% average completeness), keep iteration settings at defaults and extract core sizes per average-completeness level.

## Related tools

- **mOTUpan** (Performs the iterative Bayesian core/accessory likelihood computation with completeness-prior refinement) — https://github.com/moritzbuck/mOTUlizer
- **mOTUconvert.py** (Converts Roary, PPanGGOLiN, mmseqs2, eggNOGmapper or anvio clustering outputs into mOTUpan-compatible --cog_file inputs) — https://github.com/moritzbuck/mOTUlizer
- **mmseqs2** (Computes gene clusters (easy_cluster) when mOTUpan is run from proteomes instead of a precomputed cog_file) — https://github.com/soedinglab/MMseqs2
- **CheckM** (Provides the completeness/contamination table used as the initial completeness prior) — https://ecogenomics.github.io/CheckM/

## Examples

```
mOTUpan.py --cog_file converted_clusters.tsv --completeness checkm_output.tsv -o output.tsv
```

## Evaluation signals

- Iteration converges within --max_iter (default) without warning, indicating the completeness prior and gene partitioning reached a fixed point
- For high-quality genomes, mOTUpan core estimates are comparable to Roary and PPanGGOLiN estimates
- Along a completeness scale, Roary's core estimate drops sharply with decreasing completeness while mOTUpan estimates remain robust, with a PPanGGOLiN/mOTUpan crossover around 70% average completeness
- Bootstrap run (--boots BOOTS) reports low false-positive rate and high 'lowest false' frequency for core calls
- Output file parses as TAB-separated gene-cluster table with core/accessory likelihood ratios for every input cluster

## Limitations

- Requires mmseqs2 installed (or an externally provided cog_file) and a reasonable completeness prior; results depend on prior quality
- Genomes of very low completeness can still bias the core estimate if the prior badly underestimates missingness
- The iterative recomputation is recursive and may not converge for pathological gene-presence patterns; --max_iter caps iterations

## Evidence

- [readme] This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence.: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [abstract] We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range.: "We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range."
- [methods] --max_iter : maximum number of iterations for the recursive aspect of motupan.: "`--max_iter` : maximum number of iterations for the recursive aspect of motupan."
- [readme] --cog_file can be used as an alternative to --faas, to provide your own gene-clusters or other genetically encoded traits.: "`--cog_file` : can be used as an alternative to `--faas`, you can use it to provide you own gene-clusters (or other genetically encoded traits)."
- [readme] mOTUconvert.py converts the output of diverse programs into input files for mOTUpan.py: "`mOTUconvert.py` converts the output of diverse programs into input files for `mOTUpan.py`"
