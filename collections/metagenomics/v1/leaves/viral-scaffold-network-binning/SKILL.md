---
name: viral-scaffold-network-binning
description: Use when predicted viral scaffolds (e.g., VIBRANT/VirSorter output) plus
  single- or multi-sample coverage data must be binned into vMAGs, and you need weighted-network
  construction, KMeans/Label Propagation refinement, and redundancy-based iteration
  selection.
when_to_use_negative:
- Input is an entire metagenome expected to contain microbial genomes — vRhyme is
  not meant to bin microbes and MetaBat2 outperformed it on a non-viral synthetic
  dataset
- Sequences are multiple inactive (non-replicating) prophages from the same host genome
  — vRhyme had difficulties separating these
- Input scaffolds are already circular complete viral genomes — vRhyme v1.1.0 removes
  these pre-binning instead of binning them
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: vRhyme
  role: 'Primary tool: builds weighted networks of scaffold pairs, refines bins, scores
    iterations by protein redundancy'
  repo: https://github.com/AnantharamanLab/vRhyme
- name: networkx
  role: Constructs weighted scaffold-pair networks; Label Propagation bin refinement
    in v1.1.0
- name: scikit-learn
  role: MiniBatchKMeans bin refinement (n_clusters = s+1) in the v1.0 scheme
- name: Mmseqs2
  role: Homology clustering of predicted proteins to determine per-bin protein redundancy
    for iteration scoring
  repo: https://github.com/soedinglab/MMseqs2
- name: Prodigal
  role: ORF prediction supplying proteins for the redundancy check
  repo: https://github.com/hyattpd/Prodigal
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:c23e50143bfbf79cce3ae0de00fb93b48a8f37da9aae1fa6473d03735fbfddc6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - Mmseqs2
  - Prodigal
  - Python
  - networkx
  - scikit-learn
  - vRhyme
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# viral-scaffold-network-binning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts machine-learning-classified viral scaffold pairs into weighted networks of unrefined bins, refines them iteratively with MiniBatchKMeans (vRhyme v1.0) or Label Propagation (v1.1.0), and selects the best iteration by protein redundancy scoring via Mmseqs2, exploiting the observation that viruses seldom encode redundant genes.

## When to use

Use when predicted viral scaffolds (e.g., VIBRANT/VirSorter output) plus single- or multi-sample coverage data must be binned into vMAGs, and you need weighted-network construction, KMeans/Label Propagation refinement, and redundancy-based iteration selection.

## When NOT to use

- Input is an entire metagenome expected to contain microbial genomes — vRhyme is not meant to bin microbes and MetaBat2 outperformed it on a non-viral synthetic dataset
- Sequences are multiple inactive (non-replicating) prophages from the same host genome — vRhyme had difficulties separating these
- Input scaffolds are already circular complete viral genomes — vRhyme v1.1.0 removes these pre-binning instead of binning them

## Inputs

- Predicted viral scaffolds in FASTA (e.g., VIBRANT or VirSorter output)
- Machine-learning pair classification outputs with scaffold-pair distance/effect-size values (vRhyme_machine_distances.tsv)
- Single- or multi-sample read coverage data (BAM from Bowtie2/BWA, or coverage TSV via -c)
- Protein predictions (Prodigal ORFs) for redundancy checks
- Binning iteration presets (--iter, 10-20) and member threshold (--mems 4)

## Outputs

- vRhyme_bin_scoring.tsv (per-iteration redundancy, scaffold count, and bin count scores)
- vRhyme_best_bins.#.membership.tsv (scaffold-to-bin assignments for the best iteration)
- vRhyme_best_bins.#.summary.tsv (per-bin statistics for the best iteration)

## How to apply

Load machine-learning pair classifications and raw scaffold-pair distance/effect-size values (vRhyme_machine_distances.tsv), then build a networkx weighted network where each node is a scaffold and each edge is a classified pair with weight w = d'/p; penalize Cohen's d coverage effect sizes exceeding the iteration preset (penalty_w = 0.20, penalty_n default 10, capped at <30% of input scaffolds) and compute per-node clustering coefficients. Cap each node at max_edges = -0.001x + 6.125 (dynamic range 3-6 for x input scaffolds in v1.1.0; default 6 in v1.0.0), keeping only the most reliable connections. Refine candidate networks of bins with at least 4 members (--mems 4) using MiniBatchKMeans with n_clusters = s+1 (v1.0 refinement scheme; v1.1.0 replaced this with networkx Label Propagation). For each of the 10-20 iteration presets (--iter), report bins with at most 50 redundant proteins per bin (--red 50), scoring iterations by Mmseqs2-based protein redundancy, number of scaffolds binned, and bins generated; select the maximum-score iteration as vRhyme_best_bins.

## Related tools

- **vRhyme** (Primary tool: builds weighted networks of scaffold pairs, refines bins, scores iterations by protein redundancy) — https://github.com/AnantharamanLab/vRhyme
- **networkx** (Constructs weighted scaffold-pair networks; Label Propagation bin refinement in v1.1.0)
- **scikit-learn** (MiniBatchKMeans bin refinement (n_clusters = s+1) in the v1.0 scheme)
- **Mmseqs2** (Homology clustering of predicted proteins to determine per-bin protein redundancy for iteration scoring) — https://github.com/soedinglab/MMseqs2
- **Prodigal** (ORF prediction supplying proteins for the redundancy check) — https://github.com/hyattpd/Prodigal

## Examples

```
git clone https://github.com/AnantharamanLab/vRhyme; cd vRhyme; gunzip vRhyme/models/vRhyme_machine_model_ET.sav.gz; pip install .
```

## Evaluation signals

- vRhyme_bin_scoring.tsv reports, for each of the 10-20 iterations, protein redundancy (≤50 redundant proteins per bin, --red 50), number of scaffolds binned, and bins generated; the maximum-score iteration must be selected as best
- Edge weights must equal d'/p with Cohen's d penalties (penalty_w = 0.20, penalty_n ≤ 30% of input scaffolds) applied before max_edges capping
- Each node's edge count must not exceed max_edges = -0.001x + 6.125 (range 3-6), and refined bins must contain at least 4 members (--mems 4)
- In v1.0, MiniBatchKMeans refinement must use n_clusters = s+1; in v1.1.0, Label Propagation replaces it and networks must be connected (a v1.1.0 bug fix targeted non-connected refinement networks)
- Best-bin outputs vRhyme_best_bins.#.membership.tsv and summary.tsv must be consistent with the maximum-score iteration in vRhyme_bin_scoring.tsv

## Limitations

- vRhyme is built to run on viral sequences/scaffolds; whole-metagenome performance has not been fully evaluated and it is not meant to bin microbes
- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- Performance depends on coverage variance: it performs optimally with multiple input samples, though the optimal number depends on dataset construction
- v1.0's MiniBatchKMeans refinement was replaced by Label Propagation in v1.1.0, and a bug generating non-connected networks of bins during refinement was fixed — version choice affects refinement behavior

## Evidence
