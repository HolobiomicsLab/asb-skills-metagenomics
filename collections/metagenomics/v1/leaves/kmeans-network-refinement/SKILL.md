---
name: kmeans-network-refinement
description: Use when converting machine-learning-classified scaffold pairs into weighted
  network bins that must be refined and scored, e.g. to reconstruct vRhyme's v1.0
  MiniBatchKMeans bin-refinement and protein-redundancy iteration scoring.
when_to_use_negative:
- The goal is binning microbial (non-viral) genomes - vRhyme is explicitly not meant
  to bin microbes, and MetaBat2 performed better than vRhyme on a non-viral synthetic
  dataset
- The input contains multiple inactive (non-replicating) prophages from the same host
  genome, which vRhyme had difficulties separating
- Only a single input sample is available and robust coverage variance comparisons
  are not possible - vRhyme performs optimally with multiple input samples
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: vRhyme
  role: End-to-end implementation of the weighted-network construction, MiniBatchKMeans
    refinement, and protein-redundancy iteration scoring
  repo: https://github.com/AnantharamanLab/vRhyme
- name: networkx
  role: Builds and refines the weighted scaffold networks (clustering coefficients,
    Label Propagation in v1.1.0)
  repo: https://github.com/AnantharamanLab/vRhyme
- name: scikit-learn
  role: Supplies MiniBatchKMeans used for v1.0 bin refinement with n_clusters = s+1
- name: Mmseqs2
  role: Protein homology clustering to quantify per-bin protein redundancy for iteration
    scoring
  repo: https://github.com/soedinglab/MMseqs2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:9bfa7f5ce3e0a8c9defb9017eb66193350fae23baa68d4c21bb8d10aba8b5b08
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
  - Python
  - networkx
  - scikit-learn
  - vRhyme
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# kmeans-network-refinement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refine vRhyme's unrefined viral bins, represented as weighted scaffold networks (edge weight w = d'/p with Cohen's d coverage effect-size penalties), using MiniBatchKMeans and score 10-20 binning iterations by protein redundancy (Mmseqs2), scaffolds binned, and bins generated to select the best vMAGs.

## When to use

Use when converting machine-learning-classified scaffold pairs into weighted network bins that must be refined and scored, e.g. to reconstruct vRhyme's v1.0 MiniBatchKMeans bin-refinement and protein-redundancy iteration scoring.

## When NOT to use

- The goal is binning microbial (non-viral) genomes - vRhyme is explicitly not meant to bin microbes, and MetaBat2 performed better than vRhyme on a non-viral synthetic dataset
- The input contains multiple inactive (non-replicating) prophages from the same host genome, which vRhyme had difficulties separating
- Only a single input sample is available and robust coverage variance comparisons are not possible - vRhyme performs optimally with multiple input samples

## Inputs

- vRhyme_machine_distances.tsv (machine-learning pair classifications plus raw scaffold-pair distance/effect-size values)
- Predicted viral scaffolds (FASTA) from a viral predictor such as VIBRANT or VirSorter
- Single- or multi-sample read coverage effect-size comparisons between scaffolds
- Iteration preset thresholds (e.g. --iter 10-20, --mems 4, --red 50, penalty_w = 0.20)

## Outputs

- vRhyme_bin_scoring.tsv (per-iteration scores for protein redundancy, scaffolds binned, and bins generated)
- vRhyme_best_bins.#.membership.tsv (scaffold membership of the max-score iteration bins)
- vRhyme_best_bins.#.summary.tsv (summary statistics of the best bins)
- Refined viral metagenome-assembled genomes (vMAGs) as weighted-network-derived bins

## How to apply

Load the machine-learning pair classification outputs with raw scaffold-pair distance/effect-size values (vRhyme_machine_distances.tsv) and build a networkx weighted network where each node is a scaffold and each edge carries weight w = d'/p, penalizing Cohen's d coverage effect sizes above the iteration preset (penalty_w = 0.20, penalty_n default 10, capped at <30% of input scaffolds) and computing node clustering coefficients. Cap each node at max_edges = -0.001x + 6.125 (dynamic range 3-6 for x input scaffolds) to keep only the most reliable connections, then refine candidate networks of bins with at least 4 members (--mems 4) using MiniBatchKMeans with n_clusters = s+1 (the v1.0 refinement scheme; v1.1.0 replaced it with networkx Label Propagation). For each of the 10-20 binning iteration presets (--iter), report bins with at most 50 redundant proteins per bin (--red 50), computing protein redundancy via Mmseqs2 homology clustering on the rationale that viruses seldom encode redundant genes. Score each iteration on protein redundancy, number of scaffolds binned, and bins generated, select the maximum-score iteration, and write vRhyme_bin_scoring.tsv plus vRhyme_best_bins.#.membership.tsv and vRhyme_best_bins.#.summary.tsv.

## Related tools

- **vRhyme** (End-to-end implementation of the weighted-network construction, MiniBatchKMeans refinement, and protein-redundancy iteration scoring) — https://github.com/AnantharamanLab/vRhyme
- **networkx** (Builds and refines the weighted scaffold networks (clustering coefficients, Label Propagation in v1.1.0)) — https://github.com/AnantharamanLab/vRhyme
- **scikit-learn** (Supplies MiniBatchKMeans used for v1.0 bin refinement with n_clusters = s+1)
- **Mmseqs2** (Protein homology clustering to quantify per-bin protein redundancy for iteration scoring) — https://github.com/soedinglab/MMseqs2

## Examples

```
vRhyme -i predicted_viral_scaffolds.fasta -c coverage_table.tsv -o out_dir --mems 4 --red 50 --iter 10
```

## Evaluation signals

- vRhyme_bin_scoring.tsv contains a score per iteration (10-20 presets) and the best-bins files correspond to the maximum-score iteration
- Each refined bin contains at least 4 scaffold members (--mems 4) and at most 50 redundant proteins per bin (--red 50)
- Edge weights equal d'/p, with Cohen's d effect sizes above the preset penalized by penalty_w = 0.20 applied to at most penalty_n pairs (<30% of input scaffolds)
- Each scaffold node retains at most max_edges = -0.001x + 6.125 connections (3-6 edges), so network degrees stay within the dynamic range
- Best bins should be more complete and uncontaminated than generic binners on simulated viromes, per the paper's benchmark expectation

## Limitations

- v1.0's MiniBatchKMeans refinement was replaced by networkx Label Propagation in v1.1.0, so reproducing the published v1.0 behavior requires the older scheme (n_clusters = s+1)
- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- vRhyme's performance for a whole (unfiltered) metagenome as input has not been fully evaluated and it is not meant to bin microbes
- Optimal number of input coverage samples for robust coverage variance comparisons depends on how the dataset or metagenome was constructed

## Evidence
