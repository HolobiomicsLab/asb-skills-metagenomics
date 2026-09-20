---
name: bpe-sequence-tokenization
description: Use when deep-learning classification of 16S rRNA (or other nucleotide)
  sequences requires fixed-size tokenized inputs for a transformer trained on the
  DNABERT-2 BPE vocabulary, e.g. before HybridCNNBERTClassifier inference or training
  on Greengenes2 FASTA data.
when_to_use_negative:
- Input sequences are meant for the one-hot CNN ablation variant, which replaces DNABERT-2
  BPE tokenization with raw one-hot nucleotide encodings
- Input is a nucleotide count/feature table or non-16S data (e.g. 18S rRNA, shotgun
  metagenomes) outside DeepTaxa's FASTA-only scope
- Base-level quality scores must be used, since DeepTaxa operates on FASTA-formatted
  sequences and does not incorporate quality scores
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: DeepTaxa
  role: Framework whose predict/train pipeline applies the 512-token BPE tokenization
    before HybridCNNBERTClassifier forward passes on 16S rRNA FASTA inputs
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: Source of the BPE tokenizer and vocabulary (512-token fixed length) on which
    DeepTaxa's transformer pathway is trained from scratch
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:8fa2f96f2ba37a44fd386de3189aa70de065b110bef3c3ee7647817f1e19ccca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# BPE Sequence Tokenization

## Summary

Tokenizes 16S rRNA gene sequences into fixed-length inputs for DeepTaxa's transformer pathway using the DNABERT-2 byte-pair-encoding (BPE) vocabulary, truncating or padding each sequence to 512 tokens so the from-scratch BERT encoder and hybrid CNN-BERT heads receive uniform tensor shapes.

## When to use

Use when deep-learning classification of 16S rRNA (or other nucleotide) sequences requires fixed-size tokenized inputs for a transformer trained on the DNABERT-2 BPE vocabulary, e.g. before HybridCNNBERTClassifier inference or training on Greengenes2 FASTA data.

## When NOT to use

- Input sequences are meant for the one-hot CNN ablation variant, which replaces DNABERT-2 BPE tokenization with raw one-hot nucleotide encodings
- Input is a nucleotide count/feature table or non-16S data (e.g. 18S rRNA, shotgun metagenomes) outside DeepTaxa's FASTA-only scope
- Base-level quality scores must be used, since DeepTaxa operates on FASTA-formatted sequences and does not incorporate quality scores

## Inputs

- Nucleotide FASTA of 16S rRNA gene sequences (e.g. gg_2024_09_testing.fna.gz, 69,335 sequences)
- DeepTaxa PyTorch checkpoint (e.g. deeptaxa-full-length-v2.pt) specifying the DNABERT-2 BPE vocabulary
- Optional taxonomy TSV (gg_2024_09_testing.tsv.gz) for labeled evaluation

## Outputs

- 512-token fixed-length token-ID tensors per sequence, padded/truncated with attention masks
- Per-rank (domain-species) taxonomic predictions from the seven classification heads; per-rank accuracy, weighted F1, and ECE when labels are supplied

## How to apply

Obtain the nucleotide FASTA to be classified (e.g. gg_2024_09_testing.fna.gz with 69,335 held-out Greengenes2 2024.09 sequences). Tokenize each 16S rRNA gene sequence with the DNABERT-2 BPE tokenizer, truncating or padding to a fixed length of 512 tokens; this uniform length is required because DeepTaxa's HybridCNNBERTClassifier fuses a multiscale CNN with a transformer trained from scratch on the DNABERT-2 BPE vocabulary. Apply the same 512-token scheme identically across full-length (~1,500 bp), V3-V4 (~420 bp), and V4 (~253 bp) inputs so a single architecture serves all regions. Run tokenization inside the deeptaxa predict/train pipeline rather than as a detached preprocessing step, since the label encoders and checkpoint expect the matching representation. Verify correctness by checking that inference on the tokenized test FASTA reproduces published per-rank metrics (species accuracy ~92.96-92.95%, F1 ~0.9212, ECE 0.0242) when paired with the full-length v2 checkpoint.

## Related tools

- **DeepTaxa** (Framework whose predict/train pipeline applies the 512-token BPE tokenization before HybridCNNBERTClassifier forward passes on 16S rRNA FASTA inputs) — https://github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (Source of the BPE tokenizer and vocabulary (512-token fixed length) on which DeepTaxa's transformer pathway is trained from scratch)

## Examples

```
deeptaxa predict --fasta-file your_sequences.fna --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/predictions
```

## Evaluation signals

- Every tokenized example has exactly 512 tokens (padded or truncated), matching the published fixed input length
- Inference on the tokenized Greengenes2 2024.09 test set (69,335 sequences) with the full-length checkpoint yields species accuracy within tolerance of 92.96% and F1 0.9212
- Species-level expected calibration error ~0.0242 with 10 equal-width confidence bins confirms the tokenized inputs feed the correct model representation
- Per-rank F1 above 0.99 from domain through class on the tokenized test set indicates the vocabulary/label-encoder pairing is intact
- Cross-seed reproduction (species F1 std ≤0.0008 at every rank) after re-tokenizing and re-running inference

## Limitations

- The 512-token cap forces truncation of long full-length 16S reads (~1,500 bp), potentially discarding downstream sequence content
- Tokenization is tied to the DNABERT-2 BPE vocabulary; checkpoints retrained on other vocabularies are incompatible
- DeepTaxa's evaluation is limited to Greengenes2; tokenized-input performance on SILVA, RDP, or independent environmental/clinical datasets remains untested

## Evidence

- [methods] Fixed-length BPE tokenization to 512 tokens: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [abstract] DNABERT-2 BPE vocabulary for the transformer pathway: "We present DeepTaxa, a hybrid CNN-BERT framework that pairs a multiscale CNN with a transformer trained from scratch on the DNABERT-2 BPE vocabulary"
- [readme] DeepTaxa scope as 16S rRNA classifier: [evidence span withheld — anchor status: external_doc]
