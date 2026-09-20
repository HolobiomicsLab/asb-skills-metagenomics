---
name: bayesian-core-trait-likelihood-partitioning
description: Use when estimating core genomes from genome sets (MAGs, SAGs, or complete
  genomes) of highly variable or low completeness, where presence/absence patterns
  of gene-encoded traits must be evaluated against a completeness prior instead of
  simple occurrence thresholds.
when_to_use_negative:
- All input genomes are high-quality/complete — Roary and PPanGGOLiN give comparable
  estimates to mOTUpan on high-quality genomes, so the Bayesian incompleteness correction
  adds little
- The goal is OTU/species binning of genomes by ANI similarity network rather than
  core-genome partitioning — use mOTUlize.py (fastANI, 95% ANI default --similarity-cutoff)
  instead
- Input is raw reads with no proteomes or precomputed gene clusters — the method needs
  protein sequences or gene-cluster/trait files as input
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: mOTUpan.py
  role: Runs the iterative Bayesian core vs accessory likelihood partitioning on proteomes
    or provided gene-cluster files
  repo: github.com/moritzbuck/mOTUlizer
- name: mmseqs2
  role: Computes the gene clusters (easy_cluster) when mOTUpan is given .faa proteomes;
    --genome2cog_only returns a JSON compatible with --cog_file
  repo: https://github.com/soedinglab/MMseqs2
- name: mOTUconvert.py
  role: Converts Roary, PPanGGOLiN, mmseqs2, eggNOGmapper, and anvio outputs into
    input files for mOTUpan.py
  repo: https://github.com/moritzbuck/mOTUlizer
- name: CheckM
  role: Source of the completeness/contamination prior supplied to mOTUpan for iterative
    re-estimation
  repo: https://ecogenomics.github.io/CheckM/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:00079a4672ce104ff3271de36efe766b7278cc5917981716be3fb3dcd43aeb86
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# bayesian-core-trait-likelihood-partitioning

## Summary

mOTUpan computes the likelihood of each gene cluster belonging to the core vs accessory genome from its presence/absence pattern, using a completeness prior that is iteratively updated until convergence. This makes core-genome estimation robust for MAGs/SAGs of highly diverse completeness where tools like Roary collapse.

## When to use

Use when estimating core genomes from genome sets (MAGs, SAGs, or complete genomes) of highly variable or low completeness, where presence/absence patterns of gene-encoded traits must be evaluated against a completeness prior instead of simple occurrence thresholds.

## When NOT to use

- All input genomes are high-quality/complete — Roary and PPanGGOLiN give comparable estimates to mOTUpan on high-quality genomes, so the Bayesian incompleteness correction adds little
- The goal is OTU/species binning of genomes by ANI similarity network rather than core-genome partitioning — use mOTUlize.py (fastANI, 95% ANI default --similarity-cutoff) instead
- Input is raw reads with no proteomes or precomputed gene clusters — the method needs protein sequences or gene-cluster/trait files as input

## Inputs

- Proteome FASTA files (.faa) for the genome set, one per genome
- Gene-cluster files (JSON or TAB-separated) from Roary, PPanGGOLiN, mmseqs2, eggNOGmapper, or anvio pangenome databases, optionally converted with mOTUconvert.py
- Completeness prior: CheckM-style output table with Bin Id, Completeness, Contamination columns, or a fixed completeness value
- Optional pairwise similarity file from fastANI for mOTUlize-based mOTU definition

## Outputs

- Core/accessory partitioning table (output.tsv) giving the core-genome estimate for the genome set
- Posterior completeness estimates per genome after iterative prior re-estimation
- Bootstrap quality metrics: false-positive rate, recall, and 'lowest false' frequency, with standard deviations at higher bootstrap counts

## How to apply

Provide mOTUpan.py with either the set of proteomes (--faas), which triggers gene clustering with mmseqs2 easy_cluster, or precomputed gene clusters via --cog_file (JSON or TAB-separated, e.g. produced from Roary, PPanGGOLiN, mmseqs2, eggNOGmapper, or anvio outputs with mOTUconvert.py). Supply a completeness prior as CheckM-style output (Bin Id, Completeness, Contamination columns) or a fixed value via --MAG-completeness/--MAG-contamination thresholds for high-quality genomes. For each gene cluster, mOTUpan computes the ratio of the likelihood of the observed presence/absence pattern under the core hypothesis vs the accessory hypothesis, and the resulting partitioning is used to update the completeness prior and recomputed iteratively until convergence (bounded by --max_iter). Wrap the run with --boots to generate artificial rarefied genomes from the partitioning, yielding false-positive rate, recall, and 'lowest false' quality estimates; the 'lowest false' should be high, meaning possible false positives are highly prevalent. Compare against Roary and PPanGGOLiN on the same sets: Roary's core estimate drops sharply with decreasing completeness, whereas mOTUpan stays stable and PPanGGOLiN/mOTUpan estimates cross over near 70% average completeness.

## Related tools

- **mOTUpan.py** (Runs the iterative Bayesian core vs accessory likelihood partitioning on proteomes or provided gene-cluster files) — github.com/moritzbuck/mOTUlizer
- **mmseqs2** (Computes the gene clusters (easy_cluster) when mOTUpan is given .faa proteomes; --genome2cog_only returns a JSON compatible with --cog_file) — https://github.com/soedinglab/MMseqs2
- **mOTUconvert.py** (Converts Roary, PPanGGOLiN, mmseqs2, eggNOGmapper, and anvio outputs into input files for mOTUpan.py) — https://github.com/moritzbuck/mOTUlizer
- **CheckM** (Source of the completeness/contamination prior supplied to mOTUpan for iterative re-estimation) — https://ecogenomics.github.io/CheckM/

## Examples

```
mOTUpan.py --faas *.faa -o output.tsv
```

## Evaluation signals

- The recursive re-estimation converges (partitioning stops changing) within --max_iter iterations
- Bootstrap quality metrics from --boots show a high 'lowest false' frequency, meaning candidate false-positive core genes are highly prevalent in the genome set
- Core-genome estimate remains stable as average completeness drops, whereas Roary's default core-definition estimate declines sharply on the same completeness scale
- At high completeness (>70%), mOTUpan estimates are comparable to Roary and PPanGGOLiN, validating the likelihood-ratio partitioning
- Output file is a core/accessory table whose partitioning can be regenerated from a --cog_file JSON/TAB input converted by mOTUconvert.py

## Limitations

- Requires mmseqs2 installed when running from proteomes (--faas); otherwise precomputed gene clusters must be provided via --cog_file
- Quality of the initial partitioning depends on the supplied completeness prior (CheckM output or fixed value); the prior is only corrected iteratively, not de novo
- The anvi'o wrapper anvi-run-motupan.py is experimental with fewer options than mOTUpan.py
- Bootstrapping gives per-run quality estimates, but higher bootstrap counts are needed to obtain standard deviations

## Evidence

- [abstract] novel iterative Bayesian method: "We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range."
- [abstract] presence/absence likelihood: "The likelihood for each gene cluster to belong to core or accessory genome is estimated by computing the probability of its presence/absence pattern in the target genome set."
- [readme] iterative prior update: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] custom gene clusters: "`--cog_file` : can be used as an alternative to `--faas`, you can use it to provide you own gene-clusters (or other genetically encoded traits)."
- [readme] bootstrap quality: "`--boots BOOTS` : runs `BOOTS` bootstraps, where artificial genomes are generated using the gene-partitioning obtained with `mOTUpan`"
