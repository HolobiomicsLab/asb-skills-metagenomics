---
name: bayesian-core-genome-partitioning
description: Use when computing a core genome from genome sets containing incomplete
  MAGs/SAGs of mixed quality (e.g. 45–95% complete), where standard pangenome tools
  like Roary are sensitive to missing genes.
when_to_use_negative:
- Genome sets consist only of high-quality complete genomes and a simple presence/absence
  tool like Roary is already adequate — though PPanGGOLiN remains the state-of-the-art
  comparator
- The analysis goal is species-level genome clustering rather than core-genome estimation;
  use mOTUlize.py (fastANI, 95% ANI cutoff) instead
- The input is raw reads or a metagenome OTU table — mOTUpan works on genome-encoded
  traits (gene clusters), not reads; note mOTUlizer is unrelated to the read-based
  mOTUs tool
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan
  role: 'Core Bayesian partitioning tool: computes likelihood of each gene cluster
    being core vs accessory and iterates with updated completeness prior until convergence'
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan, installed via conda/pip or from the
    GitHub repository
  repo: github.com/moritzbuck/mOTUlizer
- name: mmseqs2
  role: Computes gene clusters from input proteomes via easy_cluster when --faas is
    used
  repo: https://github.com/soedinglab/MMseqs2
- name: CheckM
  role: Source of genome completeness/contamination priors supplied via --checkm
  repo: https://ecogenomics.github.io/CheckM/
- name: mOTUconvert.py
  role: Converts Roary, PPanGGOLiN, mmseqs2, eggNOGmapper or anvi'o outputs into --cog_file
    compatible inputs
  repo: github.com/moritzbuck/mOTUlizer
- name: PPanGGOLiN
  role: Benchmark comparator; its persistent/shell partitioning can be converted with
    mOTUconvert.py and re-scored by mOTUpan via --cog_file
  repo: https://github.com/labgem/PPanGGOLiN
- name: Roary
  role: Benchmark comparator; its gene-cluster output can be converted with mOTUconvert.py
    and re-scored by mOTUpan via --cog_file
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:abb73b78b8e0a3b5af152137899d657e4998b9fa6d9d79966b7fcbabe87a48ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - PPanGGOLiN
  - Roary
  - fastANI
  - mOTUconvert.py
  - mOTUlize.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  - mmseqs2
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# bayesian-core-genome-partitioning

## Summary

Iterative Bayesian core/accessory genome partitioning via mOTUpan (mOTUlizer package), which estimates for each gene cluster the likelihood of being present in all genomes versus a subset, using completeness priors (e.g. CheckM) to robustly handle MAGs/SAGs of diverse and low completeness.

## When to use

Use when computing a core genome from genome sets containing incomplete MAGs/SAGs of mixed quality (e.g. 45–95% complete), where standard pangenome tools like Roary are sensitive to missing genes.

## When NOT to use

- Genome sets consist only of high-quality complete genomes and a simple presence/absence tool like Roary is already adequate — though PPanGGOLiN remains the state-of-the-art comparator
- The analysis goal is species-level genome clustering rather than core-genome estimation; use mOTUlize.py (fastANI, 95% ANI cutoff) instead
- The input is raw reads or a metagenome OTU table — mOTUpan works on genome-encoded traits (gene clusters), not reads; note mOTUlizer is unrelated to the read-based mOTUs tool

## Inputs

- Proteome FASTA files (.faa) per genome, via --faas
- Gene-cluster mapping as JSON or TAB-separated file via --cog_file (e.g. produced by mOTUconvert.py from Roary/PPanGGOLiN/mmseqs2/anvi'o output)
- CheckM output or fixed completeness values: TAB-separated file with Bin Id, Completeness, Contamination columns, via --checkm

## Outputs

- Core/accessory partitioning of gene clusters per genome set (e.g. output.tsv; e.g. 1637 core clusters for the 388-genome Prochlorococcus_A mOTU)
- Bootstrapped quality metrics: false-positive rate, recall, and 'lowest false' when --boots is used

## How to apply

Provide proteomes (.faa) to mOTUpan.py, which clusters genes with mmseqs2 easy_cluster, or supply precomputed gene clusters via --cog_file (JSON or TAB-separated genome-to-trait mapping, convertible from Roary/PPanGGOLiN/mmseqs2/anvi'o with mOTUconvert.py). Supply a CheckM-style completeness file (Bin Id, Completeness, Contamination) or fixed values so the Bayesian estimator can use genome completeness as a prior. For each gene cluster, mOTUpan computes the likelihood ratio of being core versus not core; the resulting partitioning updates the completeness prior and is recomputed iteratively until convergence (cap iterations with --max_iter). Optionally run --boots to generate artificial genomes rarefied by posterior completeness estimates, yielding a bootstrapped false-positive rate, recall, and 'lowest false' statistic that predicts the true accuracy of the core-genome call. Benchmarks show mOTUpan estimates comparable to PPanGGOLiN on complete GTDB sets, while Roary's estimates diverge as completeness decreases.

## Related tools

- **mOTUpan** (Core Bayesian partitioning tool: computes likelihood of each gene cluster being core vs accessory and iterates with updated completeness prior until convergence) — github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan, installed via conda/pip or from the GitHub repository) — github.com/moritzbuck/mOTUlizer
- **mmseqs2** (Computes gene clusters from input proteomes via easy_cluster when --faas is used) — https://github.com/soedinglab/MMseqs2
- **CheckM** (Source of genome completeness/contamination priors supplied via --checkm) — https://ecogenomics.github.io/CheckM/
- **mOTUconvert.py** (Converts Roary, PPanGGOLiN, mmseqs2, eggNOGmapper or anvi'o outputs into --cog_file compatible inputs) — github.com/moritzbuck/mOTUlizer
- **PPanGGOLiN** (Benchmark comparator; its persistent/shell partitioning can be converted with mOTUconvert.py and re-scored by mOTUpan via --cog_file) — https://github.com/labgem/PPanGGOLiN
- **Roary** (Benchmark comparator; its gene-cluster output can be converted with mOTUconvert.py and re-scored by mOTUpan via --cog_file) — https://sanger-pathogens.github.io/Roary/

## Examples

```
mOTUpan.py --faas *.faa --checkm checkm_output.tsv --boots 100 -o output.tsv
```

## Evaluation signals

- Output table contains a core/accessory partitioning for every gene cluster and the iterative Bayesian estimation converged within --max_iter
- Bootstrapped FPR values from --boots lie in [0,1] and the bootstrapped false-positive rate tracks the empirical error against a liberal union-of-runs 'true' core
- On benchmark GTDB species sets, mOTUpan's normalized residue core estimates are comparable to PPanGGOLiN's, while Roary's drift as completeness decreases
- On the 388-genome Prochlorococcus_A mOTU, core estimate (~1637 clusters) is close to, and below, PPanGGOLiN's persistent+shell (1853, considered an overestimate)
- Simulated/idealized runs show recovered core-gene counts scale with good-genome completeness and low-completeness 'bad' genomes reduce core recovery

## Limitations

- Quality of the partitioning depends on completeness priors; with few genomes (e.g. one good genome plus many <45% complete ones), core recovery drops, though more bad genomes can lower the false-positive rate
- fastANI used by mOTUlize.py is memory-hungry on large genome sets (mOTUlize runs it in blocks), and the similarity clustering step is the slowest part
- Bootstrapped metrics are estimates; more bootstraps are needed to obtain standard deviations on FPR, recall and 'lowest false'

## Evidence

- [abstract] mOTUpan is a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range: "We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range."
- [methods] Iterative partitioning updates completeness prior until convergence: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] Bootstrap option description: "`--boots BOOTS` : runs `BOOTS` bootstraps, where artificial genomes are generated using the gene-partitioning obtained with `mOTUpan`"
- [readme] cog_file option description: "`--cog_file` : can be used as an alternative to `--faas`, you can use it to provide you own gene-clusters (or other genetically encoded traits)."
- [readme] Default 95% ANI similarity cutoff: "`--similarity-cutoff`: similarity cutoff used to generate the mOTU network. By default 95, as in 95% ANI cutoff"
