---
name: generalization-evaluation-on-unseen-labels
description: Use when quantifying how a hierarchical taxonomic classifier generalizes
  to species labels absent from its training vocabulary — e.g., after training on
  Greengenes2 2024.09 and evaluating on the 69,335-sequence held-out test set.
when_to_use_negative:
- When the goal is overall accuracy/F1 on the full held-out test set rather than label-novelty
  stratification — the Seen/Unseen split would discard the majority (Seen) signal.
- When evaluating on reference databases outside Greengenes2 (e.g., SILVA or RDP),
  since the Seen/Unseen rule is defined against the Greengenes2 2024.09 training vocabulary.
- When inputs are quality-scored reads rather than FASTA sequences, since DeepTaxa
  currently operates on FASTA and does not incorporate base-level quality scores.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3474
tools:
- name: DeepTaxa
  role: Hybrid CNN-BERT classifier whose per-rank predictions on the Unseen subset
    are the object of the generalization analysis
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: Hugging Face Greengenes dataset repository
  role: Source of the Greengenes2 2024.09 training/testing FASTA and taxonomy files
    used to build the Seen/Unseen partition
  repo: https://huggingface.co/datasets/systems-genomics-lab/greengenes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:9dd1fc76d433d5e2ec9d5a230979ee947fcb21681acfd64f392e3be93d739fde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DeepTaxa
  - Hugging Face Greengenes dataset repository
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# generalization-evaluation-on-unseen-labels

## Summary

A stratified evaluation protocol that partitions held-out test sequences into Seen (species label present in training vocabulary) and Unseen (label absent) subsets, then measures how per-rank accuracy degrades on the Unseen subset, evaluated only up to the rank preceding each lineage's first missing rank.

## When to use

Use when quantifying how a hierarchical taxonomic classifier generalizes to species labels absent from its training vocabulary — e.g., after training on Greengenes2 2024.09 and evaluating on the 69,335-sequence held-out test set.

## When NOT to use

- When the goal is overall accuracy/F1 on the full held-out test set rather than label-novelty stratification — the Seen/Unseen split would discard the majority (Seen) signal.
- When evaluating on reference databases outside Greengenes2 (e.g., SILVA or RDP), since the Seen/Unseen rule is defined against the Greengenes2 2024.09 training vocabulary.
- When inputs are quality-scored reads rather than FASTA sequences, since DeepTaxa currently operates on FASTA and does not incorporate base-level quality scores.

## Inputs

- Greengenes2 2024.09 test-set taxonomy labels (gg_2024_09_testing.tsv.gz, 69,335 sequences)
- Greengenes2 2024.09 test-set FASTA (gg_2024_09_testing.fna.gz)
- Training-set taxonomy labels (gg_2024_09_training.tsv.gz) defining the species vocabulary
- Per-rank predictions from a trained DeepTaxa checkpoint (e.g., deeptaxa-full-length-v2.pt)

## Outputs

- Seen/Unseen partition flags for each test sequence (~1,300 Unseen)
- Per-rank top-1 accuracy table on the Unseen subset (domain → genus, evaluated up to the first missing rank per lineage)
- Generalization curve confirming monotonic decline from 99.8% (domain) to 48.9% (genus)

## How to apply

Load the Greengenes2 2024.09 test-set taxonomy labels (gg_2024_09_testing.tsv.gz) and the classifier's per-rank predictions for the 69,335-sequence held-out test set. Apply the Seen/Unseen partition rule: flag sequences whose species label is absent from the training vocabulary (roughly 1,300 Unseen sequences). On the Unseen subset, compute top-1 accuracy at each rank from domain to genus, evaluating each lineage only up to the rank preceding its first missing rank so that absent parent labels do not penalize finer ranks. Validation requires the per-rank accuracies to match the paper's reported monotonic decline (99.8% at domain falling to 48.9% at genus), which demonstrates that DeepTaxa retains parent-rank signal for genuinely novel species rather than collapsing outright.

## Related tools

- **DeepTaxa** (Hybrid CNN-BERT classifier whose per-rank predictions on the Unseen subset are the object of the generalization analysis) — https://github.com/systems-genomics-lab/deeptaxa
- **Hugging Face Greengenes dataset repository** (Source of the Greengenes2 2024.09 training/testing FASTA and taxonomy files used to build the Seen/Unseen partition) — https://huggingface.co/datasets/systems-genomics-lab/greengenes

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- Unseen-subset top-1 accuracies reproduce the reported monotonic decline: 99.8% at domain down to 48.9% at genus.
- Unseen partition yields roughly 1,300 sequences out of 69,335 test sequences.
- Every lineage is scored only up to the rank preceding its first missing rank, so no lineage contributes a score for a rank whose parent label is absent.
- Species-level F1 degradation across similarity buckets is consistent with the smooth trend from 0.95 (high-similarity) to 0.45 (low-similarity).
- Cross-seed variance of the underlying classifier is negligible (cross-seed std ≤0.0008 F1 at every rank), so Unseen-subset results should be stable across seed checkpoints.

## Limitations

- The evaluation uses a single stratified train/validation/test split with three (later five) training seeds rather than k-fold cross-validation, so the Seen/Unseen split is tied to one partition of Greengenes2.
- Results are specific to Greengenes2 2024.09; generalization on other reference databases (SILVA, RDP) or independent environmental/clinical datasets remains untested.
- The paper reports a smooth species-F1 decline from 0.95 to 0.45 across similarity buckets, so parent-rank accuracy on Unseen labels does not imply reliable species-level novelty classification.

## Evidence

- [methods] Seen/Unseen partition rule: "We classified sequences into two categories: “Seen” (species label present in the training vocabulary) and “Unseen” (species label absent from the training vocabulary)."
- [abstract] Smooth degradation with similarity: "Performance degrades smoothly with decreasing training-set similarity (species F1 from 0.95 to 0.45)"
- [discussion] Single-split caveat: "the use of a single stratified train/validation/test split with three training seeds rather than k-fold cross-validation"
- [readme] Cross-seed reproducibility of underlying classifier: [evidence span withheld — anchor status: external_doc]
