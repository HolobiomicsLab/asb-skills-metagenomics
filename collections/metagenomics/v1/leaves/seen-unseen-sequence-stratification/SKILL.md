---
name: seen-unseen-sequence-stratification
description: Use when evaluating a taxonomic classifier's generalization to species
  not represented in its training label vocabulary, e.g. on Greengenes2 test sequences
  held out from training.
when_to_use_negative:
- You only need overall test-set metrics (species accuracy/F1/ECE) — use full-test-set
  evaluation instead
- You need to evaluate calibration (ECE) or benchmark against DADA2/QIIME 2 — stratification
  by Seen/Unseen is orthogonal to those analyses
- Your test species labels are all present in the training vocabulary, leaving an
  empty Unseen subset
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3301
tools:
- name: DeepTaxa
  role: Classifier whose predictions and per-rank label heads are evaluated on the
    Unseen subset
  repo: github.com/systems-genomics-lab/deeptaxa
- name: Greengenes2 2024.09
  role: Source of training/test taxonomy labels used to define the Seen/Unseen partition
  repo: https://huggingface.co/datasets/systems-genomics-lab/greengenes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:a3dfd45d1f40552c75de5b5d6b34a16ec06060ca8a6398b2af771f6611d9cc61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepTaxa
  - Greengenes2 2024.09
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# seen-unseen-sequence-stratification

## Summary

Stratifies held-out test sequences by whether their species label is present ('Seen') or absent ('Unseen') from the training vocabulary, then computes per-rank top-1 accuracy on the Unseen subset to quantify generalization to novel taxa.

## When to use

Use when evaluating a taxonomic classifier's generalization to species not represented in its training label vocabulary, e.g. on Greengenes2 test sequences held out from training.

## When NOT to use

- You only need overall test-set metrics (species accuracy/F1/ECE) — use full-test-set evaluation instead
- You need to evaluate calibration (ECE) or benchmark against DADA2/QIIME 2 — stratification by Seen/Unseen is orthogonal to those analyses
- Your test species labels are all present in the training vocabulary, leaving an empty Unseen subset

## Inputs

- Greengenes2 2024.09 test-set taxonomy labels (gg_2024_09_testing.tsv.gz, 69,335 sequences)
- Training-set species label vocabulary (gg_2024_09_training.tsv.gz)
- Per-sequence taxonomic predictions from the DeepTaxa model (e.g. deeptaxa-full-length-v2.pt)

## Outputs

- Binary Seen/Unseen assignment flag for each test sequence
- Per-rank top-1 accuracy table (domain through genus) on the Unseen subset
- Validation check that accuracies reproduce the reported monotonic decline from 99.8% (domain) to 48.9% (genus)

## How to apply

Load the Greengenes2 2024.09 taxonomy labels (e.g. gg_2024_09_testing.tsv.gz) together with model predictions for the 69,335-sequence held-out test set. Classify each sequence as 'Seen' if its species label appears in the training vocabulary, else 'Unseen' (about 1,300 sequences). Compute top-1 accuracy at each rank from domain to genus on the Unseen subset, evaluating only up to the rank preceding the first missing rank in each lineage. Tabulate per-rank accuracies and check for the expected monotonic decline — 99.8% at domain down to 48.9% at genus — which reflects how DeepTaxa degrades smoothly on unseen taxa rather than collapsing.

## Related tools

- **DeepTaxa** (Classifier whose predictions and per-rank label heads are evaluated on the Unseen subset) — github.com/systems-genomics-lab/deeptaxa
- **Greengenes2 2024.09** (Source of training/test taxonomy labels used to define the Seen/Unseen partition) — https://huggingface.co/datasets/systems-genomics-lab/greengenes

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- Unseen-subset top-1 accuracy at domain is ~99.8%
- Per-rank accuracies decline monotonically from domain to genus (99.8% → 48.9%)
- Unseen subset size is ~1,300 of 69,335 test sequences under the Seen/Unseen partition rule
- Evaluation is restricted to ranks preceding the first missing rank in each lineage (no propagation through unresolvable gaps)

## Limitations

- Results depend on the single stratified 80/20 train/test split (no k-fold cross-validation), with three training seeds
- Analysis is specific to Greengenes2 2024.09; performance on other reference databases (SILVA, RDP) or independent environmental/clinical datasets is untested
- Rare species labels mean small Unseen subsets (~1,300 sequences), so fine-rank estimates carry sampling uncertainty

## Evidence

- [methods] Seen/Unseen partition definition: "We classified sequences into two categories: “Seen” (species label present in the training vocabulary) and “Unseen” (species label absent from the training vocabulary)."
- [methods] Dataset scale: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling to preserve label distributions."
- [abstract] Smooth degradation on low-similarity sequences: "Performance degrades smoothly with decreasing training-set similarity (species F1 from 0.95 to 0.45)"
- [readme] Test-set size in README: [evidence span withheld — anchor status: external_doc]
