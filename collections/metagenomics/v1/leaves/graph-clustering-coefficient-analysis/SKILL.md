---
name: graph-clustering-coefficient-analysis
description: Use when metagenomic viral scaffolds have already been pair-classified
  by machine learning (vRhyme_machine_distances.tsv with distance/effect-size values)
  and you need to convert those pairs into unrefined weighted-network bins and refine
  them into vMAGs.
when_to_use_negative:
- Input is a whole metagenome rather than predicted viral scaffolds — vRhyme is built
  to run on viral sequences and is not meant to bin microbes
- Scaffolds are already complete circular viral genomes — vRhyme removes circular
  scaffolds pre-binning, so they should be handled separately
- Binning bacterial/archaeal genomes where single-copy marker genes and tools like
  MetaBat2 are more appropriate (MetaBat2 outperformed vRhyme on a non-viral synthetic
  dataset)
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: vRhyme
  role: Implements the weighted-network construction, MiniBatchKMeans/Label Propagation
    refinement, and protein-redundancy iteration scoring pipeline
  repo: https://github.com/AnantharamanLab/vRhyme
- name: networkx
  role: Python library used to build the weighted scaffold-pair networks and (v1.1.0)
    perform Label Propagation bin refinement
- name: scikit-learn
  role: Provides MiniBatchKMeans used in the v1.0 bin refinement scheme (n_clusters
    = s+1)
- name: Mmseqs2
  role: Homology clustering of predicted proteins to quantify redundant proteins per
    bin for iteration scoring
  repo: https://github.com/soedinglab/MMseqs2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:87bdd39be71f9e251a7f3b60d8d3e86f93aae3b3103ef00a008323645cb20a34
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# graph-clustering-coefficient-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs weighted networks of candidate viral bins where nodes are scaffolds and edges are weighted scaffold-pair connections (w = d'/p with Cohen's d coverage-effect-size penalties), computes per-node clustering coefficients, and refines the networks into genome bins — the core of vRhyme's vMAG construction.

## When to use

Use when metagenomic viral scaffolds have already been pair-classified by machine learning (vRhyme_machine_distances.tsv with distance/effect-size values) and you need to convert those pairs into unrefined weighted-network bins and refine them into vMAGs.

## When NOT to use

- Input is a whole metagenome rather than predicted viral scaffolds — vRhyme is built to run on viral sequences and is not meant to bin microbes
- Scaffolds are already complete circular viral genomes — vRhyme removes circular scaffolds pre-binning, so they should be handled separately
- Binning bacterial/archaeal genomes where single-copy marker genes and tools like MetaBat2 are more appropriate (MetaBat2 outperformed vRhyme on a non-viral synthetic dataset)

## Inputs

- Machine-learning pair classification outputs (scaffold-pair feature similarity classifications)
- vRhyme_machine_distances.tsv (raw scaffold-pair distance / coverage effect-size values)
- Viral scaffolds FASTA predicted by a virus caller (e.g., VIBRANT or VirSorter)
- Per-sample read coverage of scaffolds (BAM via Bowtie2/BWA alignment, or coverage tables)
- Predicted proteins (Prodigal ORF calls) for Mmseqs2 redundancy scoring

## Outputs

- Weighted networks of unrefined bins (nodes = scaffolds, edges = weighted paired-scaffold connections with clustering coefficients per node)
- vRhyme_bin_scoring.tsv (per-iteration redundancy/scaffold/bin scores)
- vRhyme_best_bins.#.membership.tsv (scaffold-to-bin assignments for the top-scoring iteration)
- vRhyme_best_bins.#.summary.tsv (per-bin statistics)

## How to apply

Load the machine-learning pair classification outputs together with raw scaffold-pair distance and effect-size values, then build a networkx weighted network where classified pairs become edges with weight w = d'/p; Cohen's d coverage effect sizes exceeding the iteration preset threshold are penalized (penalty_w = 0.20, penalty_n default 10, capped below 30% of input scaffolds). Compute clustering coefficients for each node, then cap each node at max_edges given by -0.001x + 6.125 (a dynamic range of 3-6 edges for x input scaffolds) to retain only the most reliable connections. Refine candidate networks of bins with at least 4 members (--mems 4) using MiniBatchKMeans with n_clusters = s+1 (the v1.0.0 scheme; v1.1.0 replaced MiniBatchKMeans with networkx Label Propagation). Score each of the 10-20 iteration presets (--iter) by protein redundancy — at most 50 redundant proteins per bin (--red 50) determined via Mmseqs2 homology clustering — plus number of scaffolds binned and bins generated, then report the maximum-score iteration as vRhyme_best_bins.#.membership.tsv and summary files. The rationale is viral genomes seldom encode redundant proteins, so redundancy scoring selects iterations that best assemble near-complete viral genomes.

## Related tools

- **vRhyme** (Implements the weighted-network construction, MiniBatchKMeans/Label Propagation refinement, and protein-redundancy iteration scoring pipeline) — https://github.com/AnantharamanLab/vRhyme
- **networkx** (Python library used to build the weighted scaffold-pair networks and (v1.1.0) perform Label Propagation bin refinement)
- **scikit-learn** (Provides MiniBatchKMeans used in the v1.0 bin refinement scheme (n_clusters = s+1))
- **Mmseqs2** (Homology clustering of predicted proteins to quantify redundant proteins per bin for iteration scoring) — https://github.com/soedinglab/MMseqs2

## Examples

```
conda create -c bioconda -n vRhyme python=3 networkx pandas numpy numba scikit-learn pysam samtools mash mummer mmseqs2 prodigal bowtie2 bwa
```

## Evaluation signals

- Selected iteration has at most 50 redundant proteins per bin (--red 50) as measured by Mmseqs2 clustering, and achieves the maximum score in vRhyme_bin_scoring.tsv across the 10-20 --iter presets
- Every node (scaffold) in each weighted network respects the max_edges cap of -0.001x + 6.125 (3-6 edges), and penalty_n never exceeds 30% of the number of input samples
- All refined bins contain at least 4 member scaffolds (--mems 4) and each refined network is connected (v1.1.0 fixed non-connected network generation bugs)
- membership.tsv scaffolds map 1:1 to the input FASTA scaffold names and each scaffold appears in exactly one best bin
- Test installation reports 'Success' for the Python dependencies (networkx, pandas, numpy, scikit-learn, numba, pysam) and both machine learning models

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- Binning non-viral genomes is an anticipated limitation; MetaBat2 performed better than vRhyme on a synthetic dataset of predominately non-viral sequences
- The refinement algorithm is version-dependent: v1.0.0 used MiniBatchKMeans (n_clusters = s+1) while v1.1.0 replaced it with networkx Label Propagation, so results are not directly comparable across versions
- Performance depends on dataset construction; vRhyme performs optimally with multiple input coverage samples for robust coverage variance comparisons

## Evidence
