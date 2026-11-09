---
name: protein-redundancy-bin-scoring
description: Use when binning viral scaffolds into vMAGs from metagenomes and needing
  to pick the binning iteration among candidate weighted-network refinements based
  on protein redundancy rather than bacterial marker genes.
when_to_use_negative:
- 'Binning microbial (non-viral) genomes: vRhyme is not meant to bin microbes, and
  viruses lack universal single-copy marker genes so redundancy logic differs'
- Datasets expected to contain many redundant genes (e.g., non-viral genomes), where
  the protein-redundancy assumption behind the scoring breaks down
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: vRhyme
  role: Runs the weighted-network construction, bin refinement iterations, and protein-redundancy
    scoring/selection pipeline
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Mmseqs2
  role: Homology clustering of predicted proteins to detect redundant proteins per
    bin for iteration scoring
  repo: https://github.com/soedinglab/MMseqs2
- name: scikit-learn
  role: Provides MiniBatchKMeans used to refine candidate bin networks in the v1.0
    scheme
- name: networkx
  role: Builds the weighted bin networks and provides Label Propagation, which replaced
    MiniBatchKMeans refinement in v1.1.0
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:28289608e666e7132757795878f23f0d28771dd26697b2b71d9e78b7c166d27b
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# protein-redundancy-bin-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

vRhyme's mechanism for selecting the best genome bins by scoring candidate binning iterations on protein redundancy (via Mmseqs2 homology clustering), number of scaffolds binned, and bins generated, exploiting the observation that viruses seldom encode redundant genes.

## When to use

Use when binning viral scaffolds into vMAGs from metagenomes and needing to pick the best binning iteration among candidate weighted-network refinements based on protein redundancy rather than bacterial marker genes.

## When NOT to use

- Binning microbial (non-viral) genomes: vRhyme is not meant to bin microbes, and viruses lack universal single-copy marker genes so redundancy logic differs
- Datasets expected to contain many redundant genes (e.g., non-viral genomes), where the protein-redundancy assumption behind the scoring breaks down

## Inputs

- vRhyme_machine_distances.tsv (scaffold-pair distance / coverage effect-size values)
- Machine-learning pair classification outputs for scaffold pairs
- Predicted proteins per scaffold (Prodigal ORF calls) for Mmseqs2 redundancy clustering
- Viral scaffolds FASTA from a virus predictor (e.g., VIBRANT or VirSorter)

## Outputs

- vRhyme_bin_scoring.tsv (per-iteration redundancy, scaffolds binned, bins generated scores)
- vRhyme_best_bins.#.membership.tsv (scaffold-to-bin membership of the best iteration)
- vRhyme_best_bins.#.summary.tsv (summary statistics of best bins)

## How to apply

After converting machine-learning-classified scaffold pairs into weighted networks (edges weighted w = d'/p from Cohen's d coverage effect sizes, edges capped at max_edges = -0.001x + 6.125), refine candidate networks of bins with at least 4 members (--mems 4) using MiniBatchKMeans with n_clusters = s+1 in the v1.0 scheme. Run 10-20 binning iteration presets (--iter) and for each iteration count redundant proteins per bin capped at --red 50, detected by Mmseqs2 homology clustering, since few viruses encode redundant proteins. Score each iteration using protein redundancy, number of scaffolds binned, and bins generated, then select the maximum-score iteration as the best bins. Write the scoring table vRhyme_bin_scoring.tsv together with vRhyme_best_bins.#.membership.tsv and vRhyme_best_bins.#.summary.tsv.

## Related tools

- **vRhyme** (Runs the weighted-network construction, bin refinement iterations, and protein-redundancy scoring/selection pipeline) — https://github.com/AnantharamanLab/vRhyme
- **Mmseqs2** (Homology clustering of predicted proteins to detect redundant proteins per bin for iteration scoring) — https://github.com/soedinglab/MMseqs2
- **scikit-learn** (Provides MiniBatchKMeans used to refine candidate bin networks in the v1.0 scheme)
- **networkx** (Builds the weighted bin networks and provides Label Propagation, which replaced MiniBatchKMeans refinement in v1.1.0)

## Evaluation signals

- vRhyme_bin_scoring.tsv exists and contains a score row for each of the 10-20 --iter presets
- The selected best iteration corresponds to the maximum score computed from protein redundancy, scaffolds binned, and bins generated
- Each best bin reports at most --red 50 redundant proteins per bin (default 50) as clustered by Mmseqs2
- Best bins contain at least --mems 4 member scaffolds (default 4)
- vRhyme_best_bins.#.membership.tsv and vRhyme_best_bins.#.summary.tsv are emitted alongside the scoring table for the winning iteration

## Limitations

- The refinement scheme is version-dependent: v1.0 used MiniBatchKMeans with n_clusters = s+1, but v1.1.0 replaced MiniBatchKMeans bin refinement with Label Propagation (networkx)
- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome and binning non-viral genomes
- The --penalty_n maximum is <30% of the number of input samples, so small input sample counts constrain network edge penalization

## Evidence
