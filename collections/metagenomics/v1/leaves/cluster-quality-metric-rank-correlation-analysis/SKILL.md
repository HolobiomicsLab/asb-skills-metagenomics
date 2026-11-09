---
name: cluster-quality-metric-rank-correlation-analysis
description: Use when you have per-algorithm, per-dataset clustering results from
  DNA-sequence clustering tools (iDeLUCS, DeLUCS, MeShClust v3.0) plus ground-truth
  cluster_id assignments, and need to decide whether unsupervised intrinsic metrics
  reliably track label-based clustering accuracy.
when_to_use_negative:
- When ground-truth labels are available for the dataset of interest — ACC alone is
  the preferred performance indicator per the article, so no proxy analysis is needed
- When the goal is clustering itself rather than metric agreement analysis — use iDeLUCS/MeShClust
  v3.0/DeLUCS clustering workflows instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: iDeLUCS
  role: Produces the clustering results and learned genomic-signature representations
    on which silhouette, Davies-Bouldin, and ACC are computed; also provides Example
    GT TSV files
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: DeLUCS
  role: Specialized DNA-sequence clustering algorithm whose benchmark results are
    included as one (algorithm, dataset) axis in the correlation analysis
- name: MeShClust v3.0
  role: Specialized DNA-sequence clustering algorithm whose benchmark results are
    included as one (algorithm, dataset) axis in the correlation analysis
- name: Python
  role: Used with pandas to load benchmark tables and GT TSVs and compute Spearman
    correlation and agreement statistics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:29a9f93c7f46ae431e300f54190720abc4270aee95350a5ed7cd43a4b2878932
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - DeLUCS
  - MeShClust v3.0
  - Python
  - iDeLUCS
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cluster-quality-metric-rank-correlation-analysis

## Summary

A benchmark-analysis skill that quantifies whether intrinsic clustering quality metrics (silhouette coefficient, Davies-Bouldin Index) computed over learned genomic-signature representations can serve as proxies for the external ACC metric when ground-truth labels are unavailable, using Spearman rank correlation and percentage agreement across per-algorithm, per-dataset results from iDeLUCS, MeShClust v3.0, and DeLUCS.

## When to use

Use when you have per-algorithm, per-dataset clustering results from DNA-sequence clustering tools (iDeLUCS, DeLUCS, MeShClust v3.0) plus ground-truth cluster_id assignments, and need to decide whether unsupervised intrinsic metrics reliably track label-based clustering accuracy.

## When NOT to use

- When ground-truth labels are available for the dataset of interest — ACC alone is the preferred performance indicator per the article, so no proxy analysis is needed
- When the goal is clustering itself rather than metric agreement analysis — use iDeLUCS/MeShClust v3.0/DeLUCS clustering workflows instead

## Inputs

- Per-algorithm, per-dataset benchmark results tables from iDeLUCS, MeShClust v3.0, and DeLUCS (pandas-readable)
- Ground-truth label TSV files (e.g. Example/Vertebrata_GT.tsv) with sequence_id and cluster_id columns
- Learned genomic-signature representations of the clustered DNA sequences (computed from k-mer signatures, k in {4,5,6})

## Outputs

- Comparison table of Spearman rank correlation and percentage agreement values for each (algorithm, dataset) pair
- Summary figure indicating which intrinsic metric (silhouette vs Davies-Bouldin) best tracks ACC

## How to apply

Load the per-algorithm, per-dataset benchmark result tables (iDeLUCS, MeShClust v3.0, DeLUCS) with pandas, then load the ground-truth TSV files (e.g. Example/Vertebrata_GT.tsv with sequence_id and cluster_id columns) that iDeLUCS uses only for posthoc analysis. For each clustering result, compute the silhouette coefficient and Davies-Bouldin Index over the learned genomic-signature representations, and compute ACC by matching predicted cluster assignments to the ground-truth cluster_id labels. Compute the Spearman rank correlation and percentage agreement between each intrinsic metric and ACC across all (algorithm, dataset) pairs, and assemble these into a single comparison table with a summary figure. Judge correctness by checking that every (algorithm, dataset) pair has both a correlation value in [-1, 1] and an agreement value, and that the figure clearly indicates which intrinsic metric best tracks ACC. Rationale: the paper treats ACC, not the intrinsic metrics, as the best indicator of clustering performance because only ACC reflects correspondence with ground-truth labels; this skill empirically quantifies how far intrinsic metrics deviate from that standard.

## Related tools

- **iDeLUCS** (Produces the clustering results and learned genomic-signature representations on which silhouette, Davies-Bouldin, and ACC are computed; also provides Example GT TSV files) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **DeLUCS** (Specialized DNA-sequence clustering algorithm whose benchmark results are included as one (algorithm, dataset) axis in the correlation analysis)
- **MeShClust v3.0** (Specialized DNA-sequence clustering algorithm whose benchmark results are included as one (algorithm, dataset) axis in the correlation analysis)
- **Python** (Used with pandas to load benchmark tables and GT TSVs and compute Spearman correlation and agreement statistics)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- Output table contains a correlation value and an agreement value for every (algorithm, dataset) pair analyzed
- All correlation values fall within [-1, 1]
- The summary figure clearly identifies which intrinsic metric best tracks ACC
- ACC values are computed against the GT TSV cluster_id column with sequence_id matching the FASTA sequence identifiers
- Spearman correlations are computed across the full set of (algorithm, dataset) pairs, not within a single algorithm

## Limitations

- Ground-truth labels in the GT file are used only for posthoc analysis, not training, so this analysis is possible only where GT TSVs exist
- The article treats ACC as the best performance indicator; intrinsic metrics are not substitutes for ACC when labels are available
- k-mer length is restricted to k in [4, 5, 6] for genomic-signature computation in iDeLUCS
- PyTorch version compatibility with CUDA is unspecified and may affect reproduction of the underlying clustering results

## Evidence

- [readme] The GT file must contain the columns: `sequence_id` with the sequence identifiers, these identifiers must correspond to the identifiers provided in the `<sequence_file>`. The column `cluster_id` with the "ground truth" assignments for each sequence.: "The GT file must contain the columns: `sequence_id` with the sequence identifiers, these identifiers must correspond to the identifiers provided in the `<sequence_file>`."
- [readme] These labels won't be used during training, just for posthoc analysis.: "These labels won't be used during training, just for posthoc analysis."
- [abstract] iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [abstract] that detects genomic signatures and uses them to cluster DNA sequences, without the need for sequence alignment or taxonomic identifiers: "that detects genomic signatures and uses them to cluster DNA sequences, without the need for sequence alignment or taxonomic identifiers"
