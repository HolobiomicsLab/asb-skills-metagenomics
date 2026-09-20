---
name: pangenome-partitioning-quality-bootstrapping
description: Use when computing core-genome partitioning from presence/absence patterns
  of gene clusters or traits in genome sets of highly diverse completeness (e.g. MAGs/SAGs
  with CheckM completeness/contamination estimates), and when the quality of that
  partitioning must be quantified via bootstrapping.
when_to_use_negative:
- You are looking for mOTUs, the read-based OTU-table tool (motu-tool.org) — mOTUlizer
  is a different tool for MAG/bin pangenomics
- Your goal is only gene clustering, not core/accessory classification — use --genome2cog_only
  (mmseqs easy_cluster) or Roary/PPanGGOLiN on high-quality complete genomes instead
- The genomes are complete/high-quality isolates where standard pangenome tools like
  Roary or PPanGGOLiN already perform comparably
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan.py
  role: Runs the iterative Bayesian core/accessory partitioning with completeness
    re-estimation and optional bootstraps, writing the partitioning to output.tsv
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUconvert.py
  role: Converts outputs of mmseqs2, Roary, PPanGGOLiN, eggNOGmapper, and anvi'o pangenome
    databases into input files for mOTUpan.py
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUlize.py
  role: Optionally clusters genomes into mOTUs first (via fastANI similarities, 95%
    default cutoff) so mOTUpan.py can run per-mOTU
  repo: github.com/moritzbuck/mOTUlizer
- name: MMseqs2
  role: Computes gene clusters with easy_cluster when proteomes are supplied via --faas
    instead of --cog_file
  repo: soedinglab/MMseqs2
- name: CheckM
  role: Supplies the completeness/contamination prior (e.g. example_files/example_checkm.txt)
    that seeds the iterative completeness re-estimation
  repo: ecogenomics.github.io/CheckM
- name: anvi-run-motupan.py
  role: Experimental anvi'o-compatible wrapper that runs mOTUpan directly on an anvi'o
    pangenome database
  repo: github.com/moritzbuck/mOTUlizer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:7a60a650f2c26c36bfd9ce24081e9aa527c7298fbf003308c2b0f939791e5038
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - MMseqs2
  - anvi-run-motupan.py
  - mOTUconvert.py
  - mOTUlize.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pangenome-partitioning-quality-bootstrapping

## Summary

An iterative Bayesian core/accessory genome partitioning procedure (mOTUpan in the mOTUlizer package) that classifies genome-encoded traits by core-vs-accessory likelihood ratios, re-estimates genome completeness from the partitioning until convergence, and bootstraps artificial genomes to estimate false-positive rate, recall, and 'lowest false'.

## When to use

Use when computing core-genome partitioning from presence/absence patterns of gene clusters or traits in genome sets of highly diverse completeness (e.g. MAGs/SAGs with CheckM completeness/contamination estimates), and when the quality of that partitioning must be quantified via bootstrapping.

## When NOT to use

- You are looking for mOTUs, the read-based OTU-table tool (motu-tool.org) — mOTUlizer is a different tool for MAG/bin pangenomics
- Your goal is only gene clustering, not core/accessory classification — use --genome2cog_only (mmseqs easy_cluster) or Roary/PPanGGOLiN on high-quality complete genomes instead
- The genomes are complete/high-quality isolates where standard pangenome tools like Roary or PPanGGOLiN already perform comparably

## Inputs

- Proteome FASTA files (*.faa) of the genomes, or a precomputed gene-cluster file via --cog_file (JSON or TAB-separated genome2cog mapping)
- Completeness/contamination prior table (e.g. CheckM output with Bin Id, Completeness, Contamination columns, values 0-100) or a fixed completeness value
- Optional pairwise similarity file for --similarities (three TAB-separated columns: genome, genome, 0-100 value, e.g. fastANI output kept with --keep-simi-file)

## Outputs

- output.tsv containing the core/accessory trait partitioning and updated genome completeness estimates
- Bootstrap quality metrics: false-positive rate, recall, and 'lowest false' (lowest frequency of any false positive), with standard deviations across bootstraps

## How to apply

Load a genome-to-trait mapping (JSON or TAB-separated cog file such as example_files/example_genome2cog.tsv, or supply proteomes with --faas so mmseqs2 easy_cluster builds gene clusters) and a completeness/contamination prior (e.g. CheckM output with Bin Id, Completeness, Contamination columns with values 0-100, or a fixed value). For each trait, compute the probability of its presence/absence pattern under the core and accessory hypotheses with the Bayesian estimator (Equations 1-5) and classify by the ratio of these likelihoods. Re-estimate each genome's completeness from the resulting core partition (Equation 6) and recompute the partitioning iteratively until convergence, capped by --max_iter iterations. Optionally run --boots bootstraps in which artificial genomes are built from the partitioning (core genes in all, others as a frequency-conserved gene pool) and rarefied according to posterior completeness estimates to yield false-positive rate, recall, and 'lowest false' values with standard deviations.

## Related tools

- **mOTUpan.py** (Runs the iterative Bayesian core/accessory partitioning with completeness re-estimation and optional bootstraps, writing the partitioning to output.tsv) — github.com/moritzbuck/mOTUlizer
- **mOTUconvert.py** (Converts outputs of mmseqs2, Roary, PPanGGOLiN, eggNOGmapper, and anvi'o pangenome databases into input files for mOTUpan.py) — github.com/moritzbuck/mOTUlizer
- **mOTUlize.py** (Optionally clusters genomes into mOTUs first (via fastANI similarities, 95% default cutoff) so mOTUpan.py can run per-mOTU) — github.com/moritzbuck/mOTUlizer
- **MMseqs2** (Computes gene clusters with easy_cluster when proteomes are supplied via --faas instead of --cog_file) — soedinglab/MMseqs2
- **CheckM** (Supplies the completeness/contamination prior (e.g. example_files/example_checkm.txt) that seeds the iterative completeness re-estimation) — ecogenomics.github.io/CheckM
- **anvi-run-motupan.py** (Experimental anvi'o-compatible wrapper that runs mOTUpan directly on an anvi'o pangenome database) — github.com/moritzbuck/mOTUlizer

## Examples

```
mOTUpan.py --faas *.faa --checkm example_files/example_checkm.txt --boots 100 --max_iter 100 -o output.tsv
```

## Evaluation signals

- The iterative recomputation converges (or hits --max_iter) rather than oscillating without stabilization
- Bootstrap reports a low false-positive rate and a high recall (fraction of core genes re-classified as core in the bootstrap)
- 'lowest false' value is high, indicating candidate false positives are actually highly prevalent traits in the genome set
- Re-estimated completeness values from the core partition are consistent with (and refine) the input CheckM completeness/contamination prior
- For high-quality genomes, core-genome estimates are comparable to Roary and PPanGGOLiN

## Limitations

- Requires a completeness/contamination prior (CheckM output or fixed value); without --checkm for mOTUlize.py all genomes are assumed high-quality 'MAGs'
- mOTUpan.py needs mmseqs2 installed if gene clusters are not pre-supplied; mOTUlize.py needs fastANI in the PATH unless --similarities is provided
- fastANI is memory-greedy (mitigated by running in blocks) and is the slowest part of the pipeline; the similarity parser ignores any line containing the word 'query'
- anvi-run-motupan.py is experimental and has fewer options than mOTUpan.py

## Evidence

- [abstract] iterative Bayesian classification: "We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range."
- [abstract] bootstrap procedure: "mOTUpan wraps a bootstrapping procedure to estimate the quality of a specific core-genome prediction"
- [methods] iteration to convergence: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] bootstrap rarefaction detail: "artificial genomes are generated using the gene-partitioning obtained with `mOTUpan` (e.g. the core genes are in all artificial genomes, the others are a gene-pool with their frequency conserved)"
- [readme] core vs accessory likelihood ratio: "For each of these gene-clusters it will then compute the likelihood of it being in the core vs the likelihood of it not being, the ratio of these likelihoods will determine if a trait is considered"
