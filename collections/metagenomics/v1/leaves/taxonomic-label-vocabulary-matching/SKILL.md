---
name: taxonomic-label-vocabulary-matching
description: Use when a hierarchical 16S rRNA classifier (e.g. DeepTaxa on Greengenes2
  2024.09) must be evaluated on test sequences whose species labels are absent from
  the training vocabulary, to measure parent-rank robustness.
when_to_use_negative:
- All test-set species labels are already present in the training vocabulary (no Unseen
  subset exists to analyze)
- The goal is generalization across reference databases such as SILVA or RDP — the
  article notes such evaluation 'remains to be tested' for DeepTaxa
- Input sequences lack paired ground-truth taxonomy labels, so Seen/Unseen classification
  of lineages is impossible
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: DeepTaxa
  role: Produces the per-rank predictions on the 69,335-sequence test set that are
    compared against the training species vocabulary for Seen/Unseen partitioning
    and rank-wise accuracy computation
  repo: https://github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:2fcbcfe8b1489ca7583f33564b9a6a1794fd67caf9825273cc33c053e3f9f3e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DeepTaxa
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# taxonomic-label-vocabulary-matching

## Summary

Partitions held-out test sequences into Seen vs Unseen by matching each species label against the training label vocabulary, then computes per-rank (domain–genus) top-1 accuracy on the Unseen subset to quantify how DeepTaxa degrades when the exact species is absent from training.

## When to use

Use when a hierarchical 16S rRNA classifier (e.g. DeepTaxa on Greengenes2 2024.09) must be evaluated on test sequences whose species labels are absent from the training vocabulary, to measure parent-rank robustness.

## When NOT to use

- All test-set species labels are already present in the training vocabulary (no Unseen subset exists to analyze)
- The goal is generalization across reference databases such as SILVA or RDP — the article notes such evaluation 'remains to be tested' for DeepTaxa
- Input sequences lack paired ground-truth taxonomy labels, so Seen/Unseen classification of lineages is impossible

## Inputs

- Greengenes2 2024.09 held-out test-set taxonomy label table (gg_2024_09_testing.tsv.gz, 69,335 sequences)
- Greengenes2 2024.09 held-out test-set sequences (gg_2024_09_testing.fna.gz)
- Training-set taxonomy label table used to build the Seen-species vocabulary (gg_2024_09_training.tsv.gz)
- DeepTaxa per-rank predictions from a trained checkpoint (e.g. deeptaxa-full-length-v2.pt)

## Outputs

- Seen/Unseen partition flags per test sequence (~1,300 Unseen)
- Per-rank (domain through genus) top-1 accuracy table on the Unseen subset
- Comparison against the reported monotonic decline 99.8% (domain) to 48.9% (genus)

## How to apply

Load the Greengenes2 2024.09 test taxonomy labels (e.g. gg_2024_09_testing.tsv.gz for the 69,335-sequence held-out test set) and the DeepTaxa per-rank predictions produced by `deeptaxa predict`. Apply the Seen/Unseen partition rule: flag a sequence as Unseen if its species label is absent from the training vocabulary, yielding roughly 1,300 Unseen sequences. On the Unseen subset, compute top-1 accuracy at each rank from domain to genus, evaluating only up to the rank preceding the first missing rank in each lineage so that unassignable finer ranks do not contaminate coarser-rank scores. Tabulate the per-rank accuracies and validate them against the paper's reported monotonic decline from 99.8% at domain to 48.9% at genus. The rationale is that species-level vocabulary presence dominates fine-rank scores; parent-rank accuracy on Unseen labels isolates the model's hierarchical generalization from simple label recall.

## Related tools

- **DeepTaxa** (Produces the per-rank predictions on the 69,335-sequence test set that are compared against the training species vocabulary for Seen/Unseen partitioning and rank-wise accuracy computation) — https://github.com/systems-genomics-lab/deeptaxa

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- The Unseen partition flags roughly 1,300 of the 69,335 test sequences, matching the paper's Seen/Unseen split
- Computed Unseen-subset top-1 accuracies decline monotonically from 99.8% at domain to 48.9% at genus, matching the reported values
- Accuracy is only evaluated up to the rank preceding the first missing rank in each lineage — verify no rank beyond a missing lineage rank contributes a score
- Species-level accuracy on Unseen labels should fall far below the overall 92.96% species accuracy, consistent with the reported degradation from 0.95 to 0.45 species F1 as training-set similarity decreases
- Per-rank accuracies on the Seen subset should remain high (e.g. family-level F1 above 0.90 through the medium similarity bucket), confirming the partition, not label noise, drives the gap

## Limitations

- Results are specific to the Greengenes2 2024.09 dataset; performance across other reference databases (SILVA, RDP) and independent environmental/clinical datasets remains to be tested
- DeepTaxa cannot predict a species label absent from its training vocabulary, so species-rank accuracy on the Unseen subset is structurally bounded and only parent ranks are interpretable
- Evaluation uses a single stratified 80/20 train/test split with three training seeds rather than k-fold cross-validation, so the ~1,300-sequence Unseen subset size and accuracies may vary across resplits

## Evidence

- [methods] Seen/Unseen partition rule: "We classified sequences into two categories: “Seen” (species label present in the training vocabulary) and “Unseen” (species label absent from the training vocabulary)."
- [methods] train/test split defining the 69,335-sequence test set: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling to preserve label distributions."
- [abstract] degradation with training-set similarity: "Performance degrades smoothly with decreasing training-set similarity (species F1 from 0.95 to 0.45)"
- [other] Unseen-subset evaluation outputs and accuracy pattern: [evidence span withheld — anchor status: not_found]
