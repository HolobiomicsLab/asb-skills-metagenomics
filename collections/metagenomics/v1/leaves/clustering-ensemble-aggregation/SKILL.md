---
name: clustering-ensemble-aggregation
description: Use when multiple trained networks have emitted cluster assignments for
  the same DNA sequence set (e.g. iDeLUCS mimic-based training with n_mimics augmentations)
  and a single final consensus cluster label per sequence is required.
when_to_use_negative:
- Only a single trained network is available — there is no ensemble to aggregate and
  majority voting or the ensemble is undefined
- The sequences require alignment-based comparison or taxonomic identifiers; iDeLUCS
  is explicitly alignment-free
- Labels are needed during training — GT files in iDeLUCS are used only for post-hoc
  analysis, not supervised learning
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3307
tools:
- name: iDeLUCS
  role: Trains multiple PyTorch networks whose per-network cluster assignments feed
    the information-theoretic clustering ensemble
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: PyTorch
  role: Machine learning framework used to train the networks that generate the individual
    cluster assignments being aggregated
- name: DeLUCS
  role: Predecessor algorithm whose majority-voting aggregation serves as the baseline
    being replaced by the ensemble
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:ad8c4b599cde98097487ba80fdb924c9ac725fb875d3efb65f0a305fc15429a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - DeLUCS
  - PyTorch
  - Python
  - iDeLUCS
  - pip
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# clustering-ensemble-aggregation

## Summary

Aggregate per-network cluster assignments produced by multiple iDeLUCS deep-learning runs using an information-theoretic clustering ensemble instead of the majority voting scheme of its predecessor (DeLUCS), reducing variance and boosting unsupervised clustering accuracy.

## When to use

Use when multiple trained networks have emitted cluster assignments for the same DNA sequence set (e.g. iDeLUCS mimic-based training with n_mimics augmentations) and a single final consensus cluster label per sequence is required.

## When NOT to use

- Only a single trained network is available — there is no ensemble to aggregate and majority voting or the ensemble is undefined
- The sequences require alignment-based comparison or taxonomic identifiers; iDeLUCS is explicitly alignment-free
- Labels are needed during training — GT files in iDeLUCS are used only for post-hoc analysis, not supervised learning

## Inputs

- Per-network cluster assignments from multiple independently trained iDeLUCS networks
- Encoded k-mer frequency vectors of DNA sequences from a single FASTA file (e.g. Example/Vertebrata.fas, k=6)
- Optional ground-truth labels TSV with sequence_id and cluster_id columns (e.g. Example/Vertebrata_GT.tsv) for post hoc ACC only

## Outputs

- Final consensus cluster identifier per input DNA sequence via the information-theoretic clustering ensemble
- Majority-voting baseline assignments for comparison
- ACC values for ensemble vs. baseline (e.g. written to acc_comparison.tsv)

## How to apply

After training several PyTorch networks on k-mer frequency vectors (k=6) with mimic-sequence augmentations and the contrastive consistency term (lambda=2.8), collect each network's cluster assignments on the input sequences. Instead of aggregating by majority voting (the DeLUCS baseline), combine the per-network co-association information with a clustering ensemble based on information theory, which reduces variance across runs. Run both aggregation strategies so the ensemble can be validated post hoc against the majority-voting baseline. If ground-truth labels are available (a --GT_file with sequence_id and cluster_id columns, used only post hoc, never during training), compute unsupervised clustering ACC for both the ensemble and the baseline; on Example/Vertebrata the ensemble with the contrastive term is expected to exceed the majority-voting baseline ACC.

## Related tools

- **iDeLUCS** (Trains multiple PyTorch networks whose per-network cluster assignments feed the information-theoretic clustering ensemble) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **PyTorch** (Machine learning framework used to train the networks that generate the individual cluster assignments being aggregated)
- **DeLUCS** (Predecessor algorithm whose majority-voting aggregation serves as the baseline being replaced by the ensemble)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- Unsupervised clustering ACC of the information-theoretic ensemble, computed post hoc against Example/Vertebrata_GT.tsv labels, must exceed the majority-voting baseline ACC
- Every input sequence in the FASTA file receives exactly one final cluster identifier from the ensemble
- Ensemble aggregation must not use GT labels during aggregation — they enter only in post hoc ACC computation
- Effective parameters match the documented Vertebrata run: n_epochs=35, lambda=2.8, k=6, n_clusters=5, n_mimics=3, batch_sz=512
- Comparison of ensemble vs. baseline ACC is reported (e.g. in acc_comparison.tsv) rather than a single aggregate value

## Limitations

- iDeLUCS' latest stable PyTorch release might not be compatible with the user's version of CUDA, affecting reproducibility of the networks whose assignments are aggregated
- The information-theoretic ensemble replaces majority voting but the source cards provide no standalone command to run aggregation in isolation; it is embedded in the iDeLUCS pipeline
- n_clusters must be at least the true number of clusters when GT_file is provided, otherwise post hoc ACC comparisons are invalid

## Evidence

- [other] The pipeline's final stage replaces the majority voting scheme with a clustering ensemble based on information theory, and training adds a contrastive consistency term to the loss function: [evidence span withheld — anchor status: not_found]
- [readme] idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512"
- [readme] These labels won't be used during training, just for posthoc analysis.: "These labels won't be used during training, just for posthoc analysis."
- [methods] iDeLUCS uses PyTorch as the machine learning development framework: "iDeLUCS uses PyTorch as the machine learning development framework"
