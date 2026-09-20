---
name: species-level-accuracy-evaluation
description: Use when validating or benchmarking a 16S rRNA taxonomic classifier on
  short amplicon reads (V3–V4, ~420 bp, 341F/805R) against Greengenes2 ground-truth
  taxonomy at the species rank.
when_to_use_negative:
- Input sequences are full-length ~1,500 bp 16S reads — use the full-length checkpoint
  (92.95% 5-seed mean) instead of the V3–V4 amplicon checkpoint
- Input sequences are ~253 bp V4 amplicons (515F/806R) — the V4 checkpoint (82.84%
  5-seed mean, 16,909-species vocabulary) is the appropriate model
- No ground-truth taxonomy labels are available — the evaluation mode (per-rank accuracy/F1/ECE)
  requires --taxonomy-file; run plain prediction mode instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: DeepTaxa
  role: HybridCNNBERTClassifier inference engine; deeptaxa predict CLI with --checkpoint
    and --taxonomy-file emits per-rank accuracy, F1, and ECE
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: BPE tokenizer used to encode each ~420 bp amplicon sequence before model input
    (512-token fixed length)
- name: Greengenes2 2024.09
  role: Reference database supplying the ~420 bp in-silico V3–V4 test amplicons and
    ground-truth seven-rank taxonomy
  repo: https://huggingface.co/datasets/systems-genomics-lab/greengenes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:ce554ceffcecea88cc3502acb1e20ec1855aa60bd27dc6d1857d52799c96b34c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - Greengenes2 2024.09
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# species-level-accuracy-evaluation

## Summary

Reproduce the species-level top-1 accuracy of DeepTaxa's dedicated V3–V4 amplicon checkpoint (87.55% reported; 87.54% 5-seed mean in README) on ~420 bp in-silico V3–V4 hypervariable-region test sequences derived from Greengenes2 2024.09.

## When to use

Use when validating or benchmarking a 16S rRNA taxonomic classifier on short amplicon reads (V3–V4, ~420 bp, 341F/805R) against Greengenes2 ground-truth taxonomy at the species rank.

## When NOT to use

- Input sequences are full-length ~1,500 bp 16S reads — use the full-length checkpoint (92.95% 5-seed mean) instead of the V3–V4 amplicon checkpoint
- Input sequences are ~253 bp V4 amplicons (515F/806R) — the V4 checkpoint (82.84% 5-seed mean, 16,909-species vocabulary) is the appropriate model
- No ground-truth taxonomy labels are available — the evaluation mode (per-rank accuracy/F1/ECE) requires --taxonomy-file; run plain prediction mode instead

## Inputs

- In-silico V3–V4 hypervariable-region test FASTA (~420 bp amplicons, 341F/805R, derived from Greengenes2 2024.09)
- Ground-truth taxonomy TSV (per-sequence seven-rank labels, e.g. gg_2024_09_testing.tsv.gz)
- Dedicated V3–V4 checkpoint deeptaxa-v3v4-v2.pt (75.8 M parameters, 8,347-species label vocabulary)

## Outputs

- Per-rank top-1 accuracy (Domain through Species; species target 87.55% / 87.54% 5-seed mean)
- Per-rank F1 (species F1 ≈ 0.8592–0.8588) and expected calibration error (ECE ≈ 0.0278)
- Per-rank softmax prediction files written to the output directory (v3v4_species_accuracy metrics)

## How to apply

Download the dedicated V3–V4 checkpoint (deeptaxa-v3v4-v2.pt, 75.8 M parameters, 8,347-species label vocabulary) from huggingface.co/systems-genomics-lab/deeptaxa. Load the in-silico V3–V4 hypervariable-region test FASTA (~420 bp amplicons from Greengenes2 2024.09, primers 341F/805R) plus its taxonomy TSV, and tokenize sequences with the DNABERT-2 BPE tokenizer truncated/padded to 512 tokens. Run deeptaxa predict with the HybridCNNBERTClassifier under PyTorch mixed precision on GPU; the tool emits per-rank softmax predictions and top-1 labels across the seven Linnean ranks. Compute per-rank top-1 accuracy by exactly matching the highest-probability predicted label to the ground-truth taxonomy at each rank, and additionally record F1 and expected calibration error (reported: F1 = 0.8592, ECE = 0.0278 for seed 42). Validation succeeds if species-level accuracy reproduces 87.55% for the seed-42 checkpoint (README 5-seed mean: 87.54%, cross-seed std ≤0.0004 F1); a zero-shot full-length baseline instead yields ~26.9 percentage points lower accuracy, so using the wrong checkpoint is the main failure mode.

## Related tools

- **DeepTaxa** (HybridCNNBERTClassifier inference engine; deeptaxa predict CLI with --checkpoint and --taxonomy-file emits per-rank accuracy, F1, and ECE) — https://github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (BPE tokenizer used to encode each ~420 bp amplicon sequence before model input (512-token fixed length))
- **Greengenes2 2024.09** (Reference database supplying the ~420 bp in-silico V3–V4 test amplicons and ground-truth seven-rank taxonomy) — https://huggingface.co/datasets/systems-genomics-lab/greengenes

## Examples

```
deeptaxa predict --fasta-file gg_2024_09_testing.fna.gz --taxonomy-file gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-v3v4-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- Species-level top-1 accuracy reproduces the reported 87.55% for the dedicated V3–V4 checkpoint (seed 42)
- Species F1 falls in the reported range ~0.8588–0.8592 and species ECE ≈ 0.0278–0.0242-range calibration
- The gain over a zero-shot full-length baseline is approximately +26.9 percentage points at species rank
- Predicted species labels are always members of the checkpoint's 8,347-species label vocabulary (V3–V4, not 16,909)
- Cross-seed variation stays within reported bounds (species std 0.0004 F1 / 0.03 percentage points across seeds 42, 123, 456, 789, 1011)

## Limitations

- Single stratified train/validation/test split rather than k-fold cross-validation; evaluation is limited to Greengenes2 and not validated on SILVA, RDP, or independent environmental/clinical datasets
- Species labels absent from the 8,347-species training vocabulary cannot be predicted ('Unseen' labels reduce achievable accuracy)
- Model operates on FASTA only and does not incorporate base-level quality scores, which may matter for noisy amplicon data

## Evidence

- [abstract] Dedicated V3–V4 checkpoint species accuracy: "a dedicated V3–V4 amplicon checkpoint reaches 87.55% species accuracy from an approximately 420 bp window"
- [methods] Tokenization method: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [results] Amplicon extraction with primer mismatches: "up to two mismatches per primer"
- [readme] V3–V4 checkpoint specification and 5-seed mean: [evidence span withheld — anchor status: external_doc]
- [readme] Evaluation mode output contract: [evidence span withheld — anchor status: not_found]
