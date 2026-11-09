---
name: short-read-clustering-benchmarking
description: 'Use when evaluating clustering accuracy (ACC) of a DNA sequence clustering
  tool on synthetic FASTA datasets, especially regimes excluded from the iDeLUCS paper:
  average read length <500 bp or expected cluster counts above 200 (e.g. ~5000).'
when_to_use_negative:
- Input is real biological data with unknown cluster structure and no ground-truth
  labels — the workflow requires GT cluster_id columns for ACC computation
- The question is about ACC results reported in the iDeLUCS paper itself — such results
  do not exist for short reads (<500 bp) or >200 clusters; the paper explicitly deferred
  these to future work
- Sequences require alignment-based clustering or taxonomic identifiers — iDeLUCS
  is explicitly alignment-free and identifier-free
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: iDeLUCS
  role: Deep learning CLI that clusters DNA sequences via genomic signatures; run
    on each dataset with --sequence_file, --GT_file, --n_clusters, --k=6, --lambda=2.8
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: pip
  role: Installs the idelucs package from PyPI
- name: PyTorch
  role: Machine learning framework underlying iDeLUCS training
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:89fe614d6cbfaf97e488a720c18c2a2046ea44b2b1600ae77bff50e239354643
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - PyTorch
  - Python
  - iDeLUCS
  - pip
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# short-read-clustering-benchmarking

## Summary

Benchmark alignment-free clustering of DNA sequences (including short reads <500 bp and large cluster counts >200) using iDeLUCS, a deep learning tool that clusters genomic k-mer signatures without alignment, and compute clustering accuracy (ACC) against ground-truth labels held out from training.

## When to use

Use when evaluating clustering accuracy (ACC) of a DNA sequence clustering tool on synthetic FASTA datasets, especially regimes excluded from the iDeLUCS paper: average read length <500 bp or expected cluster counts above 200 (e.g. ~5000).

## When NOT to use

- Input is real biological data with unknown cluster structure and no ground-truth labels — the workflow requires GT cluster_id columns for ACC computation
- The question is about ACC results reported in the iDeLUCS paper itself — such results do not exist for short reads (<500 bp) or >200 clusters; the paper explicitly deferred these to future work
- Sequences require alignment-based clustering or taxonomic identifiers — iDeLUCS is explicitly alignment-free and identifier-free

## Inputs

- Synthetic DNA sequence datasets in single FASTA files (e.g. Girgis 2022 short-read <500 bp and large-cluster-count ~5000 regimes)
- Tab-separated ground-truth label files with columns sequence_id and cluster_id (identifiers matching the FASTA file)
- Expected cluster count per dataset (n_true_clusters)

## Outputs

- Per-dataset clustering accuracy (ACC) values computed against the ground-truth cluster_id column
- Aggregated TSV benchmark table with one ACC value per excluded regime (short-read and large-cluster-count datasets)
- Cluster identifier assignments for every sequence in each input FASTA file

## How to apply

Install iDeLUCS via pip (pip install idelucs), which pulls in Python and PyTorch as the ML framework. Load the excluded Girgis (2022) synthetic FASTA sequence files together with their tab-separated ground-truth files containing sequence_id and cluster_id columns. Run the idelucs CLI per dataset with --sequence_file and --GT_file set, k=6 (within the supported [4,5,6] range), --n_clusters equal to the expected cluster count for the regime, --lambda=2.8, --n_epochs within the recommended 50-150 range, and --batch_sz chosen as a multiple of --n_mimics. Ground-truth labels are only used post hoc, never during training, so use them after clustering to compute ACC per dataset and aggregate the values into a single TSV benchmark table. Note the paper itself never reported ACC for these regimes: its authors explicitly deferred short-read (<500 bp) and >200-cluster testing to future work, so these results fill a documented evaluation gap.

## Related tools

- **iDeLUCS** (Deep learning CLI that clusters DNA sequences via genomic signatures; run on each dataset with --sequence_file, --GT_file, --n_clusters, --k=6, --lambda=2.8) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **pip** (Installs the idelucs package from PyPI)
- **PyTorch** (Machine learning framework underlying iDeLUCS training)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- Output TSV table exists and includes one row per excluded regime (short-read <500 bp and large-cluster-count ~5000)
- Every reported ACC was computed post hoc against the GT cluster_id column, which was never used during training
- n_clusters parameter is within the tool's supported range for each run and at least n_true_clusters when GT_file is provided
- k is within the supported [4,5,6] range; n_epochs within the recommended [50,150]; batch_sz is a multiple of n_mimics
- idelucs -h runs successfully to confirm installation before benchmarking

## Limitations

- The --n_clusters parameter supports a range of 0 and 2-199, so regimes with ~5000 expected clusters fall outside the tool's documented range and may require workarounds or fail
- The iDeLUCS paper excluded short-read datasets and datasets with more than 200 clusters from its evaluation; authors flag systematic testing on these as future work
- PyTorch version compatibility with the local CUDA installation is not guaranteed
- batch_sz is limited by machine capacity; a larger batch may improve convergence but may harm accuracy

## Evidence

- [readme] post hoc labels: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id`"
- [readme] k-mer options: "--k| integer|k-mer length|  Default = 6, Options:`[4,5,6]`"
- [readme] cluster range: "Default = 5 ; Range: 0, 2-199."
- [abstract] alignment-free scope: "that detects genomic signatures and uses them to cluster DNA sequences, without the need for sequence alignment or taxonomic identifiers"
- [readme] PyTorch framework: "iDeLUCS uses PyTorch as the machine learning development framework and its latest stable release might not be compatible with your version of CUDA"
