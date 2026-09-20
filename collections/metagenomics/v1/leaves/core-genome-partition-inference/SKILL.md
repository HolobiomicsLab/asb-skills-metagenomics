---
name: core-genome-partition-inference
description: Use when you need a core-genome partitioning for a set of genomes or
  MAGs/SAGs of varying and possibly low completeness, and you can supply proteomes
  (.faa) plus a completeness prior such as CheckM output.
when_to_use_negative:
- Your genomes are highly contaminated, not merely incomplete — mOTUpan's model treats
  incompleteness, and contamination handling is the domain of mOTUlize.py's mOTU-clustering
  step
- You only need OTU/mOTU clustering of genomes by ANI similarity — that is mOTUlize.py,
  not mOTUpan
- You want a read-to-OTU table from metagenomic reads — that is the unrelated mOTUs
  tool (motu-tool.org), not mOTUlizer
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: mOTUpan
  role: Core iterative Bayesian classifier that computes the core-genome partitioning
    from gene-cluster presence/absence patterns; run via mOTUpan.py
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan (also provides mOTUlize.py, mOTUconvert.py,
    anvi-run-motupan.py)
  repo: https://github.com/moritzbuck/mOTUlizer
- name: MMseqs2
  role: Computes the gene clusters (easy_cluster) from the input proteomes that mOTUpan
    then partitions into core/accessory
  repo: https://github.com/soedinglab/MMseqs2
- name: mOTUconvert.py
  role: Converts output of mmseqs2, Roary, PPanGGOLiN, eggNOGmapper or anvi'o pangenomes
    into input files for mOTUpan.py
  repo: https://github.com/moritzbuck/mOTUlizer
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:7c4cc292f9e17c61d0dd6db0f964e51da906414748f42be8841d19d91beea72c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MMseqs2
  - mOTUconvert.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# core-genome-partition-inference

## Summary

Iterative Bayesian method (mOTUpan, part of mOTUlizer) that computes the core vs accessory genome partitioning for sets of genomes/MAGs of highly diverse completeness by likelihood-ratio tests on gene-cluster presence/absence patterns. Gene clustering is done with mmseqs2 easy_cluster and completeness priors (e.g. from CheckM) are updated iteratively until convergence.

## When to use

Use when you need a core-genome partitioning for a set of genomes or MAGs/SAGs of varying and possibly low completeness, and you can supply proteomes (.faa) plus a completeness prior such as CheckM output.

## When NOT to use

- Your genomes are highly contaminated, not merely incomplete — mOTUpan's model treats incompleteness, and contamination handling is the domain of mOTUlize.py's mOTU-clustering step
- You only need OTU/mOTU clustering of genomes by ANI similarity — that is mOTUlize.py, not mOTUpan
- You want a read-to-OTU table from metagenomic reads — that is the unrelated mOTUs tool (motu-tool.org), not mOTUlizer

## Inputs

- Protein FASTA files (*.faa) for each genome/MAG (via --faas)
- Completeness (and contamination) prior, e.g. CheckM output file or a fixed value (--checkm / --MAG-completeness)
- Alternative: user-provided gene clusters or genome-encoded traits as JSON or TAB-separated file (--cog_file)
- Optional pairwise genome similarities (e.g. from fastANI) instead of recomputing

## Outputs

- TAB-separated per-gene-cluster core/accessory partitioning written to output.tsv via -o
- Posterior per-genome completeness estimates (used to rarefy artificial genomes in bootstrapping)
- Optional bootstrap quality statistics (false-positive rate, recall, 'lowest false') from --boots

## How to apply

Run mOTUpan.py with the proteome FASTA files via --faas (mmseqs2 must be installed); it computes gene clusters using mmseqs2 easy_cluster, or supply your own clusters/traits via --cog_file (JSON or TAB-separated). Provide a completeness prior with --checkm (CheckM output or a fixed value); if omitted, genome quality assumptions must be handled explicitly. For each gene cluster mOTUpan computes the likelihood of the observed presence/absence pattern under core vs non-core models, and the likelihood ratio decides core vs accessory; the resulting partitioning updates the completeness prior and is recomputed iteratively until convergence (cap iterations with --max_iter). Write the per-gene-cluster partitioning with -o (e.g. output.tsv, TAB-separated). The method scales: on 9443 S. aureus GTDB genomes gene clustering took ~4 min on 24 threads and the mOTUpan run ~2 h 15 min single-threaded at ~3 GB RAM. Optionally validate a specific prediction with the --boots BOOTS bootstrapping, which estimates false-positive rate, recall and 'lowest false'.

## Related tools

- **mOTUpan** (Core iterative Bayesian classifier that computes the core-genome partitioning from gene-cluster presence/absence patterns; run via mOTUpan.py) — github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan (also provides mOTUlize.py, mOTUconvert.py, anvi-run-motupan.py)) — https://github.com/moritzbuck/mOTUlizer
- **MMseqs2** (Computes the gene clusters (easy_cluster) from the input proteomes that mOTUpan then partitions into core/accessory) — https://github.com/soedinglab/MMseqs2
- **mOTUconvert.py** (Converts output of mmseqs2, Roary, PPanGGOLiN, eggNOGmapper or anvi'o pangenomes into input files for mOTUpan.py) — https://github.com/moritzbuck/mOTUlizer

## Examples

```
mOTUpan.py --faas *.faa -o output.tsv
```

## Evaluation signals

- output.tsv contains a core/accessory assignment for every gene cluster
- The iterative recomputation of the completeness prior converges (or hits --max_iter as a documented cap)
- On large sets the run stays within the reported resource envelope (e.g. ~4 min mmseqs2 clustering on 24 threads, ~2 h 15 min single-threaded, ~3 GB RAM for 9443 S. aureus genomes)
- Running with --boots yields finite, interpretable false-positive rate, recall and 'lowest false' values; 'lowest false' should be high, indicating candidate false positives are highly prevalent
- For high-quality genomes, core-genome estimates are comparable to Roary and PPanGGOLiN

## Limitations

- mmseqs2 must be installed for the default --faas path (mOTUlize.py likewise needs fastANI in PATH if --similarities is not provided)
- mOTUlize's fastANI step is memory-greedy and the slowest part for large genome sets (mitigated by running in blocks and --keep-simi-file)
- Accuracy on incomplete MAGs/SAGs depends on the supplied completeness prior; the iterative procedure is bounded by --max_iter
- The iterative partitioning may require --max_iter to terminate if convergence is not reached

## Evidence

- [abstract] mOTUpan is a novel iterative Bayesian method for computing the core genome for sets of genomes of highly diverse completeness range: [evidence span withheld — anchor status: not_found]
- [abstract] The likelihood for each gene cluster to belong to core or accessory genome is estimated by computing the probability of its presence/absence pattern in the target genome set.: "The likelihood for each gene cluster to belong to core or accessory genome is estimated by computing the probability of its presence/absence pattern"
- [methods] it computes gene clusters using mmseqs2: "it computes gene clusters using [`mmseqs2`](https://github.com/soedinglab/MMseqs2)"
- [readme] This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence.: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] Simplest command to run (needs mmseqs2 installed): "Simplest command to run (needs mmseqs2 installed), but many options:"
