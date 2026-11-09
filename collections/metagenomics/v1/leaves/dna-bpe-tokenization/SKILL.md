---
name: dna-bpe-tokenization
description: Use when a FASTA of 16S rRNA sequences (full-length ~1,500 bp or in-silico
  amplicons such as ~420 bp V3-V4) must be converted into fixed-length token inputs
  for a DNABERT-2-vocabulary transformer classifier.
when_to_use_negative:
- Input is already model-ready (pre-tokenized tensor batches or checkpoint logits),
  so re-tokenizing would duplicate work
- The downstream classifier is k-mer based (SINTAX, Kraken 2) or alignment based (BLAST,
  MMseqs2, VSEARCH), which expects raw nucleotide sequences rather than BPE subword
  IDs
- Sequences are not DNA (e.g., protein or 18S/metagenomic reads) and were not tokenized
  on the DNA BPE vocabulary used to train the released DeepTaxa checkpoints
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: DNABERT-2
  role: Supplies the BPE tokenizer and DNA vocabulary on which the DeepTaxa transformer
    is trained from scratch
- name: DeepTaxa
  role: Consumes the 512-token BPE representations as input to the HybridCNNBERTClassifier
    during training and inference
  repo: https://github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:6c113aaa9f8cd365eebe9ec3be105b49019f0bd2387b67c0424585e08a0afdc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# DNA BPE tokenization

## Summary

Encodes 16S rRNA gene sequences as subword tokens using the DNABERT-2 byte-pair-encoding (BPE) tokenizer so that the DeepTaxa HybridCNNBERTClassifier can consume them; from-scratch training on this vocabulary lets the transformer learn DNA-specific subword patterns rather than amino-acid or word-level units.

## When to use

Use when a FASTA of 16S rRNA sequences (full-length ~1,500 bp or in-silico amplicons such as ~420 bp V3-V4) must be converted into fixed-length token inputs for a DNABERT-2-vocabulary transformer classifier.

## When NOT to use

- Input is already model-ready (pre-tokenized tensor batches or checkpoint logits), so re-tokenizing would duplicate work
- The downstream classifier is k-mer based (SINTAX, Kraken 2) or alignment based (BLAST, MMseqs2, VSEARCH), which expects raw nucleotide sequences rather than BPE subword IDs
- Sequences are not DNA (e.g., protein or 18S/metagenomic reads) and were not tokenized on the DNA BPE vocabulary used to train the released DeepTaxa checkpoints

## Inputs

- FASTA file of 16S rRNA gene sequences (e.g., gg_2024_09_training.fna.gz, ~1,500 bp)
- FASTA of in-silico amplicons (~420 bp V3-V4 from Greengenes2 2024.09)
- Paired taxonomy TSV (e.g., gg_2024_09_training.tsv.gz) if tokenized sequences feed supervised training

## Outputs

- Per-sequence token ID arrays truncated/padded to 512 tokens on the DNABERT-2 BPE vocabulary
- Tokenized tensors ready for the DeepTaxa HybridCNNBERTClassifier embedding layer

## How to apply

Take each nucleotide sequence from the Greengenes2 2024.09 training/testing FASTA (277,336 training / 69,335 test sequences, or the 273,003 in-silico V3-V4 amplicons) and tokenize it with the DNABERT-2 BPE tokenizer, then truncate or pad the token list to a fixed length of 512 tokens. The BPE vocabulary is learned DNA subwords, which lets the transformer capture both local motifs and long-range dependencies without hand-crafted k-mer encoding. Feed the resulting token IDs directly into the embedding layer of the HybridCNNBERTClassifier (or CNN/BERT ablation variants); the same tokenization is required at both training and inference time so checkpoints remain compatible. Rationale: DeepTaxa pairs its multiscale CNN with a transformer trained from scratch on the DNABERT-2 BPE vocabulary, and consistent 512-token truncation/padding is what produced the 92.96% species accuracy on Greengenes2 and the 87.55% V3-V4 checkpoint accuracy.

## Related tools

- **DNABERT-2** (Supplies the BPE tokenizer and DNA vocabulary on which the DeepTaxa transformer is trained from scratch)
- **DeepTaxa** (Consumes the 512-token BPE representations as input to the HybridCNNBERTClassifier during training and inference) — https://github.com/systems-genomics-lab/deeptaxa

## Evaluation signals

- Every tokenized sequence has exactly 512 tokens after truncation/padding (long ~1,500 bp 16S reads are truncated; short ~420 bp V3-V4 amplicons are padded)
- Token IDs fall within the DNABERT-2 BPE vocabulary range used to initialize the DeepTaxa transformer
- Checkpoint loaded for inference must match the same vocabulary; deeptaxa predict on the 69,335-sequence Greengenes2 test set should reproduce ~92.95-92.96% species top-1 accuracy (seed-42 single model ~92.96% per-seed, cross-seed species std ≤0.0008 F1)
- V3-V4 checkpoint path: tokenized ~420 bp amplicons through deeptaxa-v3v4-v2.pt should yield ~87.5% species accuracy (87.55% seed 42, F1 0.8592, ECE 0.0278)

## Limitations

- Inputs are limited to a fixed 512-token window; full-length sequences longer than the window are truncated, so any signal beyond the retained region is lost
- DeepTaxa operates on FASTA-formatted sequences and does not incorporate base-level quality scores, so tokenization cannot convey read-error information
- The transformer is trained from scratch on the DNABERT-2 BPE vocabulary for 16S classification; tokenizations from other vocabularies are incompatible with the released checkpoints

## Evidence

- [methods] BPE tokenization of 16S sequences with DNABERT-2 tokenizer, truncated/padded to 512 tokens: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [abstract] Hybrid CNN-BERT with DNABERT-2 BPE vocabulary trained from scratch: "a hybrid CNN-BERT framework that pairs a multiscale CNN with a transformer trained from scratch on the DNABERT-2 BPE vocabulary"
- [readme] Checkpoint compatibility caveat: [evidence span withheld — anchor status: not_found]
