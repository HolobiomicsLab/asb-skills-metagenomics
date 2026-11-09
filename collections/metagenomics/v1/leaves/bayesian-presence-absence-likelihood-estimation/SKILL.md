---
name: bayesian-presence-absence-likelihood-estimation
description: Use when computing core-genome partitioning for a set of closely related
  genomes of mixed quality (complete genomes plus incomplete/contaminated MAGs, SAGs,
  SUBs) from presence/absence patterns of gene clusters.
when_to_use_negative:
- You need OTU clustering of genomes into mOTUs or ANI network construction from nucleotide
  sequences — use mOTUlize.py with fastANI similarities instead
- Your goal is read-based OTU-table construction (the unrelated 'mOTUs' tool at motu-tool.org),
  not core-genome estimation from genome sets
- The genomes are not closely related (mOTUlizer targets 'a group of closely related
  MAGs/Genomes/bins/SUBs'); the 95% ANI species cutoff defines the expected relatedness
  scale
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan
  role: Iterative Bayesian method that estimates core vs accessory membership of gene
    clusters from presence/absence patterns under a completeness prior
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUpan.py
  role: Command-line entry point; runs mmseqs2 easy_cluster on --faas inputs, iteratively
    recomputes likelihoods, and writes the partitioning to -o output.tsv
  repo: https://github.com/moritzbuck/mOTUlizer
- name: MMseqs2
  role: Computes the gene clusters (easy_cluster) from protein FASTA files that are
    the presence/absence traits for the Bayesian estimation
  repo: https://github.com/soedinglab/MMseqs2
- name: mOTUconvert.py
  role: Converts the output of diverse programs (e.g. Roary, PPanGGOLiN, eggNOGmapper,
    anvi'o pangenomes) into input files for mOTUpan.py
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package in which mOTUpan is implemented
  repo: https://github.com/moritzbuck/mOTUlizer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:e740d85780336b5f6a5ea057ca26fff93b4e8bfe1e5f32a17df473815f987630
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/moritzbuck/mOTUlizer
  tools:
  - MMseqs2
  - mOTUconvert.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# bayesian-presence-absence-likelihood-estimation

## Summary

Iterative Bayesian estimation of whether each gene cluster belongs to the core or accessory genome by computing the probability of its presence/absence pattern across genome sets that include incomplete MAGs/SAGs, implemented in mOTUpan (mOTUlizer package).

## When to use

Use when computing core-genome partitioning for a set of closely related genomes of mixed quality (complete genomes plus incomplete/contaminated MAGs, SAGs, SUBs) from presence/absence patterns of gene clusters.

## When NOT to use

- You need OTU clustering of genomes into mOTUs or ANI network construction from nucleotide sequences — use mOTUlize.py with fastANI similarities instead
- Your goal is read-based OTU-table construction (the unrelated 'mOTUs' tool at motu-tool.org), not core-genome estimation from genome sets
- The genomes are not closely related (mOTUlizer targets 'a group of closely related MAGs/Genomes/bins/SUBs'); the 95% ANI species cutoff defines the expected relatedness scale

## Inputs

- Protein FASTA files (*.faa) for each genome in the set (via --faas)
- Optionally, user-provided gene clusters or other genome-encoded traits as a JSON-compatible structure (via --cog_file)
- Optional completeness/contamination values for genomes (e.g. checkm output or a fixed value)

## Outputs

- TAB-separated file (output.tsv via -o) assigning each gene cluster a core or accessory label
- Optional bootstrap-based quality estimates of the core-genome prediction (--boots)

## How to apply

First compute gene clusters with mmseqs2 easy_cluster over the protein FASTA files (*.faa) of the genome set, e.g. via 'mOTUpan.py --faas *.faa -o output.tsv'; alternatively supply pre-computed gene clusters with --cog_file. mOTUpan then estimates, for each gene cluster, the likelihood of belonging to the core versus accessory genome by computing the probability of its presence/absence pattern in the target genome set, accounting for each genome's completeness (a completeness prior can be supplied, e.g. from checkm output or a fixed value). The resulting partitioning is used to update the completeness prior and the likelihoods are recomputed iteratively until convergence (bounded by --max_iter). Optionally run --boots bootstraps, where artificial genomes are generated from the obtained gene-partitioning, to estimate the quality of the core-genome prediction. The final TAB-separated per-gene-cluster core/accessory partitioning is written to the file given via -o.

## Related tools

- **mOTUpan** (Iterative Bayesian method that estimates core vs accessory membership of gene clusters from presence/absence patterns under a completeness prior) — https://github.com/moritzbuck/mOTUlizer
- **mOTUpan.py** (Command-line entry point; runs mmseqs2 easy_cluster on --faas inputs, iteratively recomputes likelihoods, and writes the partitioning to -o output.tsv) — https://github.com/moritzbuck/mOTUlizer
- **MMseqs2** (Computes the gene clusters (easy_cluster) from protein FASTA files that are the presence/absence traits for the Bayesian estimation) — https://github.com/soedinglab/MMseqs2
- **mOTUconvert.py** (Converts the output of diverse programs (e.g. Roary, PPanGGOLiN, eggNOGmapper, anvi'o pangenomes) into input files for mOTUpan.py) — https://github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package in which mOTUpan is implemented) — https://github.com/moritzbuck/mOTUlizer

## Examples

```
mOTUpan.py --faas *.faa -o output.tsv
```

## Evaluation signals

- Run completes within the reported resource envelope: ~4 min gene clustering on 24 threads and ~2 h 15 min single-threaded mOTUpan at ~3 GB RAM on 9443 S. aureus GTDB genomes (Ryzen 9 3900X)
- output.tsv contains a core/accessory assignment for every gene cluster and is TAB-separated
- Iterative recomputation terminates within --max_iter iterations (convergence of the completeness prior and partitioning)
- For high-quality genomes, core-genome estimates are comparable to Roary and PPanGGOLiN (documented benchmark correspondence)
- Bootstrapping (--boots) yields a quality estimate of the specific core-genome prediction

## Limitations

- Iterative recomputation is bounded by --max_iter; non-convergence within the maximum iterations is a possible failure mode
- Results depend on genome completeness values; an external estimate (e.g. checkm) or a fixed value must be supplied as the completeness prior
- Benchmarked/comparable only on closely related genome sets (species-scale, ~95% ANI); the method is validated against Roary and PPanGGOLiN for high-quality genomes but its advantage lies in using lower-completeness MAGs/SAGs
- Gene clustering step requires mmseqs2 installed; no README changelog is available

## Evidence

- [abstract] The likelihood for each gene cluster to belong to core or accessory genome is estimated by computing the probability of its presence/absence pattern in the target genome set.: "The likelihood for each gene cluster to belong to core or accessory genome is estimated by computing the probability of its presence/absence pattern"
- [methods] Iterative recomputation until convergence.: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [methods] Simplest mOTUpan.py invocation with mmseqs2.: "Simplest command to run (needs mmseqs2 installed), but many options:"
- [methods] Bootstrapping quality estimation.: "`--boots BOOTS` : runs `BOOTS` bootstraps, where artificial genomes are generated using the gene-partitioning obtained with `mOTUpan`"
- [methods] Gene clustering with mmseqs2.: "it computes gene clusters using [`mmseqs2`](https://github.com/soedinglab/MMseqs2)"
- [results] Reported runtime and memory on 9443 S. aureus genomes.: [evidence span withheld — anchor status: not_found]
