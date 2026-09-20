---
name: dna-sequence-clustering-benchmark-evaluation
description: 'Decision support for choosing a clustering-quality metric when ground-truth
  labels are unavailable: apply when benchmark tables exist for multiple clustering
  algorithms/datasets and you need to know if silhouette or Davies-Bouldin can proxy
  ACC.'
when_to_use_negative:
- Ground-truth labels are available for training — iDeLUCS is unsupervised; GT files
  are used for posthoc analysis only, so a supervised classifier should be used instead
- The task is running a single clustering job rather than evaluating/validating clustering
  quality across algorithms and metrics
- Datasets lack any ground-truth assignments at all — ACC cannot be computed, and
  the intrinsic-vs-external comparison degenerates to intrinsic metrics alone
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: iDeLUCS
  role: Deep-learning tool that detects genomic signatures and clusters DNA sequences;
    produces clustering assignments evaluated with ACC and intrinsic metrics
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: MeShClust v3.0
  role: Specialized DNA-sequence clustering algorithm benchmarked against iDeLUCS
    in the ACC-vs-intrinsic-metric comparison
- name: DeLUCS
  role: Specialized deep-learning clustering algorithm benchmarked against iDeLUCS
    in the ACC-vs-intrinsic-metric comparison
- name: Python
  role: Environment for loading benchmark tables with pandas and computing silhouette,
    Davies-Bouldin, ACC, and Spearman correlation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:ec3b79fdd67e9c0e38245aa694ac44745b0dba592531e8be9a90e15409add324
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - DeLUCS
  - MeShClust v3.0
  - Python
  - iDeLUCS
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# dna-sequence-clustering-benchmark-evaluation

## Summary

A benchmark-evaluation skill for DNA-sequence clustering that reproduces per-algorithm, per-dataset results (iDeLUCS, MeShClust v3.0, DeLUCS) and tests whether intrinsic metrics (silhouette coefficient, Davies-Bouldin Index) rank-correlate with the external ACC metric computed against ground-truth cluster_id labels.

## When to use

Decision support for choosing a clustering-quality metric when ground-truth labels are unavailable: apply when benchmark tables exist for multiple clustering algorithms/datasets and you need to know if silhouette or Davies-Bouldin can proxy ACC.

## When NOT to use

- Ground-truth labels are available for training — iDeLUCS is unsupervised; GT files are used for posthoc analysis only, so a supervised classifier should be used instead
- The task is running a single clustering job rather than evaluating/validating clustering quality across algorithms and metrics
- Datasets lack any ground-truth assignments at all — ACC cannot be computed, and the intrinsic-vs-external comparison degenerates to intrinsic metrics alone

## Inputs

- Per-algorithm, per-dataset benchmark results tables from iDeLUCS, MeShClust v3.0, and DeLUCS
- Ground-truth label TSV files (sequence_id and cluster_id columns, optional phylogeny column), e.g. Example/Vertebrata_GT.tsv
- Clustering outputs assigning a cluster identifier to each sequence in the input FASTA file

## Outputs

- Comparison table of Spearman rank correlation and percentage agreement between each intrinsic metric (silhouette, Davies-Bouldin) and ACC for every (algorithm, dataset) pair
- Summary figure indicating which intrinsic metric best tracks ACC
- ACC values per clustering result from optimal matching to ground-truth cluster_id assignments

## How to apply

Load the per-algorithm, per-dataset benchmark results produced by iDeLUCS, MeShClust v3.0, and DeLUCS using Python/pandas, plus the ground-truth TSV files (e.g. Example/Vertebrata_GT.tsv with sequence_id and cluster_id columns) that iDeLUCS uses only for posthoc analysis. For each clustering result, compute the silhouette coefficient and Davies-Bouldin Index over the learned genomic-signature representations, and compute external ACC by optimally matching predicted cluster assignments to the ground-truth cluster_id assignments. Then compute Spearman rank correlation and percentage agreement between each intrinsic metric and ACC across all (algorithm, dataset) pairs, assembling the values into a single comparison table and a summary figure. Follow the article's decision rationale: ACC — not the intrinsic metrics — is treated as the best indicator of clustering performance because ACC alone reflects correspondence with ground-truth labels, so intrinsic metrics should be reported as proxies only where their rank correlation with ACC is demonstrated.

## Related tools

- **iDeLUCS** (Deep-learning tool that detects genomic signatures and clusters DNA sequences; produces clustering assignments evaluated with ACC and intrinsic metrics) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **MeShClust v3.0** (Specialized DNA-sequence clustering algorithm benchmarked against iDeLUCS in the ACC-vs-intrinsic-metric comparison)
- **DeLUCS** (Specialized deep-learning clustering algorithm benchmarked against iDeLUCS in the ACC-vs-intrinsic-metric comparison)
- **Python** (Environment for loading benchmark tables with pandas and computing silhouette, Davies-Bouldin, ACC, and Spearman correlation)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- Output table contains a correlation value and agreement value for every (algorithm, dataset) pair analyzed
- All Spearman correlation values fall within the range [-1, 1]
- The summary figure clearly indicates which intrinsic metric best tracks ACC
- ACC values are computed against the GT file's cluster_id column matched via sequence_id identifiers from the FASTA file
- Intrinsic metrics are computed over the learned genomic-signature representations, not raw sequences

## Limitations

- The authors treat ACC, not the intrinsic metrics, as the best indicator of clustering performance because ACC alone reflects correspondence with ground-truth labels; intrinsic metrics may not reliably proxy ACC
- GT labels are not used during training — they serve only for posthoc analysis, so evaluation requires ground truth that real unlabeled datasets lack
- iDeLUCS's latest stable PyTorch release may not be compatible with your CUDA version, affecting reproducibility of benchmark runs
- n_clusters should be at least the number of true clusters when a GT file is provided; mis-specification distorts ACC comparisons

## Evidence

- [other] The authors treat ACC, not the intrinsic metrics, as the best indicator of clustering performance, because ACC alone reflects the correspondence between cluster assignments and the ground-truth labels.: [evidence span withheld — anchor status: not_found]
- [abstract] iDeLUCS outperforms two classical clustering algorithms by ~20% and two specialized algorithms by ~12% in unsupervised clustering accuracy on real DNA sequence datasets: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets"
- [readme] GT file labels are posthoc only: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id`"
- [methods] iDeLUCS clustering command with ground truth: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35"
