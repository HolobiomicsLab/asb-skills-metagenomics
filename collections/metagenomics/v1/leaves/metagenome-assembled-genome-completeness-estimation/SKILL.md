---
name: metagenome-assembled-genome-completeness-estimation
description: Use when analyzing genome sets of mixed or dubious quality (MAGs, SAGs,
  bins, SUBs) where completeness varies widely, and a completeness prior must feed
  into iterative Bayesian core/accessory partitioning.
when_to_use_negative:
- Genome sets consist of complete, high-quality isolate genomes only, where standard
  pangenome tools like Roary or PPanGGOLiN suffice without a completeness prior
- You need read-level OTU tables rather than genome/bin-level clustering — this is
  not the read-based mOTUs tool
- No completeness estimates (CheckM or fixed values) are available and genome quality
  is unknown/unassessable
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUlizer / mOTUpan.py
  role: Implements the iterative Bayesian core-genome estimation that consumes completeness
    priors and updates them until convergence
  repo: https://github.com/moritzbuck/mOTUlizer
- name: CheckM
  role: Provides the completeness and contamination values per genome used as the
    completeness prior
  repo: https://ecogenomics.github.io/CheckM/
- name: fastANI
  role: Computes pairwise ANI similarities for mOTU clustering; needs to be in PATH
    when no --similarities file is provided
  repo: https://github.com/ParBLiSS/FastANI
- name: mmseqs2
  role: Builds gene clusters (easy_cluster) whose presence/absence patterns are scored
    against the completeness prior
  repo: https://github.com/soedinglab/MMseqs2
- name: mOTUlize.py
  role: Clusters genomes into mOTUs at the 95% ANI cutoff, using --MAG-completeness/--SUB-completeness
    to bin genomes by quality
  repo: https://github.com/moritzbuck/mOTUlizer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:74bcf218b83d6a12956a85661152bf5be32d38c84b60c0c0ace50cc9985dceff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/moritzbuck/mOTUlizer
  tools:
  - CheckM
  - PPanGGOLiN
  - Roary
  - fastANI
  - mOTUconvert.py
  - mOTUlize.py
  - mOTUlizer
  - mOTUlizer / mOTUpan.py
  - mOTUpan
  - mOTUpan.py
  - mmseqs2
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# metagenome-assembled-genome-completeness-estimation

## Summary

Estimate and use genome completeness priors (e.g. CheckM completeness/contamination values or fixed values) for MAG/SAG genome sets of highly diverse quality, so that core-genome partitioning methods like mOTUpan can correct for missing genes instead of being misled by incompleteness.

## When to use

Use when analyzing genome sets of mixed or dubious quality (MAGs, SAGs, bins, SUBs) where completeness varies widely, and a completeness prior must feed into iterative Bayesian core/accessory partitioning.

## When NOT to use

- Genome sets consist of complete, high-quality isolate genomes only, where standard pangenome tools like Roary or PPanGGOLiN suffice without a completeness prior
- You need read-level OTU tables rather than genome/bin-level clustering — this is not the read-based mOTUs tool
- No completeness estimates (CheckM or fixed values) are available and genome quality is unknown/unassessable

## Inputs

- Nucleotide FASTA files of genomes/MAGs/SAGs (--fnas) for mOTU clustering
- Proteome FASTA files (--faas *.faa) for gene clustering
- CheckM output file (or fixed values) with completeness and contamination per genome
- Optional precomputed pairwise similarity file from fastANI (--similarities)
- Optional gene-cluster file (JSON or TAB-separated) via --cog_file

## Outputs

- mOTU clustering table (genome-to-mOTU assignments and stats) from mOTUlize.py
- Core/accessory gene-cluster partitioning with per-trait core-likelihood ratios from mOTUpan.py (output.tsv)
- Posterior completeness estimates per genome used to update the iterative prior
- Bootstrap quality estimates (false positive rate, recall, 'lowest false') when --boots is used

## How to apply

First compute per-genome completeness (and contamination) with CheckM, or provide fixed values, formatted as the file passed to mOTUlizer's --checkM-completeness/--MAG-completeness options. Feed these values into mOTUlize.py so that --MAG-completeness/--MAG-contamination controls which genomes are treated as high-quality 'MAGs' for building the mOTU network, and --SUB-completeness controls recruitment of lower-quality satellite bins. Then run mOTUpan.py with --faas *.faa plus the completeness file: mmseqs2 easy_cluster builds gene clusters, and the Bayesian likelihood of each presence/absence pattern (core vs accessory) yields a completeness prior that is iteratively updated until convergence (--max_iter). The rationale is that MAG incompleteness makes naive presence/absence counting unreliable; the iterative prior re-estimation lets low-completeness genomes still contribute signal rather than being discarded.

## Related tools

- **mOTUlizer / mOTUpan.py** (Implements the iterative Bayesian core-genome estimation that consumes completeness priors and updates them until convergence) — https://github.com/moritzbuck/mOTUlizer
- **CheckM** (Provides the completeness and contamination values per genome used as the completeness prior) — https://ecogenomics.github.io/CheckM/
- **fastANI** (Computes pairwise ANI similarities for mOTU clustering; needs to be in PATH when no --similarities file is provided) — https://github.com/ParBLiSS/FastANI
- **mmseqs2** (Builds gene clusters (easy_cluster) whose presence/absence patterns are scored against the completeness prior) — https://github.com/soedinglab/MMseqs2
- **mOTUlize.py** (Clusters genomes into mOTUs at the 95% ANI cutoff, using --MAG-completeness/--SUB-completeness to bin genomes by quality) — https://github.com/moritzbuck/mOTUlizer

## Examples

```
mOTUpan.py --faas *.faa --checkm checkm_output.tsv -o output.tsv --boots 100
```

## Evaluation signals

- Every genome in the input set has a completeness value (from CheckM or fixed) and appears in the output partitioning; no genomes silently dropped
- Iterative recomputation converges within --max_iter, with the completeness prior stabilizing between iterations
- Bootstrapping (--boots) yields low false positive rate for core genes and high recall, with 'lowest false' high (false positives highly prevalent)
- On benchmark GTDB (11570 genomes, 301 species) and StratFreshDB (8955 genomes, 258 mOTUs) sets, mOTUpan core estimates stay comparable to PPanGGOLiN even as completeness decreases
- Simulated rarefaction of high-quality genomes recovers true core genomes at low completeness thresholds, unlike Roary which drops away

## Limitations

- fastANI is memory-greedy; mOTUlize.py runs it in blocks to bypass this, and it is the slowest step for large genome sets
- Existing core-genome methods (without completeness priors) are very sensitive to missing genes and fragmented genomes, limiting them to high-quality inputs
- MAG/SAG incompleteness inherently hinders large-scale subpopulation diversity, core-genome structure and genome evolution studies if not modeled explicitly

## Evidence

- [abstract] mOTUpan is a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range: "We present mOTUpan, a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range."
- [methods] This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence.: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] --MAG-completeness/--MAG-contamination controls which genomes are considered high quality, used for creating the mOTU-network: "`--MAG-completeness`/`--MAG-contamination`: controls which genomes are considered high quality, used for creating the mOTU-network"
- [readme] for example checkm output or a fixed value completeness prior of these genomes: "as well as a completeness prior of these genomes (for example [`checkm`](https://ecogenomics.github.io/CheckM/) output or a fixed value)"
- [abstract] capable of using genomes at lower completeness thresholds: "It has shown comparable estimates to state-of-the-art tools Roary and PPanGGOLiN for high-quality genomes and is capable of using genomes at lower completeness thresholds."
