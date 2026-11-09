---
name: coverage-effect-size-edge-weighting
description: Use when binning predicted viral scaffolds from metagenomes into vMAGs
  and classified scaffold pairs plus coverage effect-size values (vRhyme_machine_distances.tsv)
  must be converted into refined, redundancy-scored genome bins.
when_to_use_negative:
- Input is an entire metagenome rather than predicted viral scaffolds — vRhyme is
  not meant to bin microbes and performance on whole metagenomes has not been fully
  evaluated
- Goal is binning bacteria, archaea, or eukaryotes, where marker-gene-based tools
  such as MetaBat2 may perform better (MetaBat2 outperformed vRhyme on a non-viral
  synthetic dataset)
- Circular scaffolds (complete viral genomes) are present and should be extracted
  pre-binning rather than networked
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_0622
tools:
- name: vRhyme
  role: Primary tool implementing weighted-network construction, MiniBatchKMeans/Label
    Propagation refinement, and iteration scoring
  repo: https://github.com/AnantharamanLab/vRhyme
- name: networkx
  role: Python library used to build the weighted bin networks (added as a dependency
    in v1.1.0; also provides Label Propagation refinement)
- name: scikit-learn
  role: Provides MiniBatchKMeans used for v1.0 bin refinement with n_clusters = s+1
- name: Mmseqs2
  role: Homology clustering used to determine protein redundancy for bin scoring
  repo: https://github.com/soedinglab/MMseqs2
- name: Python
  role: Implementation language for vRhyme
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:8ae6128ed8a20640ed39af4c48c26802fa484f6ef5cee6bda5995b5541ecad6a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# coverage-effect-size-edge-weighting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts machine-learning-classified scaffold pairs from vRhyme into weighted networks of unrefined bins (edge weight w = d'/p from Cohen's d coverage effect sizes), refines them with MiniBatchKMeans, and scores iterations by protein redundancy computed via Mmseqs2.

## When to use

Use when binning predicted viral scaffolds from metagenomes into vMAGs and classified scaffold pairs plus coverage effect-size values (vRhyme_machine_distances.tsv) must be converted into refined, redundancy-scored genome bins.

## When NOT to use

- Input is an entire metagenome rather than predicted viral scaffolds — vRhyme is not meant to bin microbes and performance on whole metagenomes has not been fully evaluated
- Goal is binning bacteria, archaea, or eukaryotes, where marker-gene-based tools such as MetaBat2 may perform better (MetaBat2 outperformed vRhyme on a non-viral synthetic dataset)
- Circular scaffolds (complete viral genomes) are present and should be extracted pre-binning rather than networked

## Inputs

- Machine-learning pair classification outputs for scaffold pairs
- Raw scaffold-pair coverage distance/effect-size values (vRhyme_machine_distances.tsv)
- Viral scaffold FASTA (predicted e.g. with VIBRANT or VirSorter)
- Read coverage samples (BAM files from Bowtie2/BWA alignment)

## Outputs

- Weighted networks of unrefined bins (networkx graphs; nodes = scaffolds, edges = weighted paired-scaffold connections)
- vRhyme_bin_scoring.tsv (iteration scores by protein redundancy, scaffolds binned, bins generated)
- vRhyme_best_bins.#.membership.tsv (best-iteration bin memberships)
- vRhyme_best_bins.#.summary.tsv (best-bin summaries)

## How to apply

Load the machine-learning pair classification outputs and raw scaffold-pair distance/effect-size values, then build a networkx weighted network where each node is a scaffold and each edge carries weight w = d'/p; edges with Cohen's d coverage effect sizes above the preset threshold receive a penalty (penalty_w = 0.20, penalty_n default 10, capped at <30% of input scaffolds). Cap each node's degree via max_edges = -0.001x + 6.125 (dynamic range 3-6 for x input scaffolds), keeping only the most reliable connections, and compute clustering coefficients per node. Refine candidate networks of bins with at least 4 members (--mems 4) using MiniBatchKMeans with n_clusters = s+1 (the v1.0 scheme; v1.1.0 replaced this with Label Propagation). Sweep the 10-20 binning iteration presets (--iter), reporting bins with at most 50 redundant proteins per bin (--red 50), and score each iteration by Mmseqs2 protein redundancy, number of scaffolds binned, and bins generated, exploiting the observation that viruses seldom encode redundant genes. Select the maximum-score iteration and write vRhyme_bin_scoring.tsv with vRhyme_best_bins.#.membership.tsv and vRhyme_best_bins.#.summary.tsv.

## Related tools

- **vRhyme** (Primary tool implementing weighted-network construction, MiniBatchKMeans/Label Propagation refinement, and iteration scoring) — https://github.com/AnantharamanLab/vRhyme
- **networkx** (Python library used to build the weighted bin networks (added as a dependency in v1.1.0; also provides Label Propagation refinement))
- **scikit-learn** (Provides MiniBatchKMeans used for v1.0 bin refinement with n_clusters = s+1)
- **Mmseqs2** (Homology clustering used to determine protein redundancy for bin scoring) — https://github.com/soedinglab/MMseqs2
- **Python** (Implementation language for vRhyme)

## Examples

```
vRhyme -i viral_scaffolds.fna -s s1.bam -s s2.bam -s s3.bam -t 15 -o vRhyme_out --mems 4 --red 50 --iter 10
```

## Evaluation signals

- Edge weights in the network equal d'/p, with penalty_w = 0.20 applied only to pairs whose Cohen's d exceeds the iteration preset threshold
- No node exceeds max_edges = -0.001x + 6.125 connections (dynamic range 3-6); penalty_n candidates stay below 30% of input scaffolds
- Reported best bins each contain at most 50 redundant proteins (--red 50) as determined by Mmseqs2 clustering
- vRhyme_bin_scoring.tsv scores for the 10-20 --iter presets exist and the selected maximum-score iteration matches the membership/summary files emitted
- Every scaffold in output bins belongs to a connected component of the weighted network of at least 4 members (--mems 4)

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- Binning non-viral genomes is an anticipated limitation; vRhyme is not meant to bin microbes
- The v1.0 refinement scheme (MiniBatchKMeans, n_clusters = s+1) was replaced in v1.1.0 by Label Propagation (networkx), so results differ across versions
- vRhyme performs optimally with multiple input coverage samples, but the optimal value depends on how the dataset or metagenome was constructed

## Evidence

- [abstract] vRhyme utilizes single- or multi-sample coverage effect size comparisons between scaffolds: "vRhyme utilizes single- or multi-sample coverage effect size comparisons between scaffolds"
- [abstract] iterations of weighted networks and refined bins: "which are compiled into iterations of weighted networks and refined bins"
- [abstract] protein redundancy scoring rationale: "a protein redundancy scoring mechanism based on the observation that viruses seldom encode redundant genes"
