---
name: amino-acid-reduction-encoding
description: Use when protein FASTA sequences (e.g., nitrogen cycling families) must
  be recoded into AAR k-mer vectors, or when comparing feature-generation runtime/model
  performance of MIQS-recoded k-mers versus unencoded k-mers at a fixed k.
when_to_use_negative:
- Input is already a vectorized k-mer feature table or model output (e.g., results/apply/snekmer_results.csv)
  rather than raw FASTA sequences
- The goal is downstream prediction/annotation scoring (apply mode with Score/delta/Confidence
  columns), not feature generation or its runtime benchmarking
- Input sequences are nucleotide (e.g., FASTQ) rather than protein sequences — Snekmer
  recodes proteins
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: Snekmer
  role: Performs amino-acid reduction and k-mer vectorization (--until vectorize)
    under configurable k and alphabet parameters
  repo: github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Orchestrates the Snekmer pipeline stages so vectorization can be run in isolation
    with --until vectorize
- name: Python
  role: Runtime for Snekmer and for computing/summarizing the runtime speedup ratio
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:0cf76ff5b8140a1f05c311eca42e180438dba306c41bd6a564d099e1bf58c4f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Snakemake
  - Snekmer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# amino-acid-reduction-encoding

## Summary

Recodes protein sequences into amino-acid-reduced (AAR) k-mer vectors with Snekmer (e.g., MIQS reduced alphabet, k=4) and benchmarks the feature-generation cost against unencoded native-sequence k-mers. The MIQS-recoded vectorization is nearly 4x faster while yielding comparable models (Supplementary Table S2).

## When to use

Use when protein FASTA sequences (e.g., nitrogen cycling families) must be recoded into AAR k-mer vectors, or when comparing feature-generation runtime/model performance of MIQS-recoded k-mers versus unencoded k-mers at a fixed k.

## When NOT to use

- Input is already a vectorized k-mer feature table or model output (e.g., results/apply/snekmer_results.csv) rather than raw FASTA sequences
- The goal is downstream prediction/annotation scoring (apply mode with Score/delta/Confidence columns), not feature generation or its runtime benchmarking
- Input sequences are nucleotide (e.g., FASTQ) rather than protein sequences — Snekmer recodes proteins

## Inputs

- Nitrogen cycling protein family sequences in FASTA format (placed in Snekmer input/ directory)
- Snekmer config.yaml specifying k=4 and alphabet ('miqs' vs None)

## Outputs

- AAR-recoded k-mer feature vectors from the vectorize stage
- Timing table (snekmer_feature_timing.csv) with runtimes for MIQS-recoded and unencoded 4-mer runs plus the computed speedup ratio

## How to apply

Place the protein-family FASTA files in Snekmer's input/ directory and set config.yaml parameters: k=4 with alphabet 'miqs' (encoding 5, reduced alphabet size 10). Run only the vectorization stage with `snekmer <mode> --until vectorize` and record the wall-clock runtime of the feature-generation step. Repeat the identical run with k=4 and alphabet set to None (no reduced alphabet, 20-letter native encoding), again recording runtime. Compute the speedup ratio as unencoded runtime divided by MIQS-recoded runtime and compare against the near-4x speedup reported in Supplementary Table S2; the article also notes MIQS-recoded 4-mer models perform comparably to unencoded 4-mer models, so the recoding trades little accuracy for large speedup. Assemble per-parameterization (optionally per-family) runtimes plus the ratio into a timing table such as snekmer_feature_timing.csv.

## Related tools

- **Snekmer** (Performs amino-acid reduction and k-mer vectorization (--until vectorize) under configurable k and alphabet parameters) — github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Orchestrates the Snekmer pipeline stages so vectorization can be run in isolation with --until vectorize)
- **Python** (Runtime for Snekmer and for computing/summarizing the runtime speedup ratio)

## Examples

```
snekmer model --until vectorize  # with config.yaml: k: 4, alphabet: miqs (then rerun with alphabet: None to time both)
```

## Evaluation signals

- The MIQS-recoded k=4 vectorization completes approximately 4x faster than the unencoded k=4 run, matching Supplementary Table S2
- snekmer_feature_timing.csv contains runtime entries for both parameterizations (alphabet miqs and None) plus the computed speedup ratio
- Both runs use identical inputs and k=4; the only differing parameter is the alphabet (reduced size 10 vs 20)
- The vectorize stage produces valid k-mer vectors for all input families in each run (no failed/missing vector outputs)
- MIQS-recoded 4-mer downstream models perform comparably to unencoded 4-mer models, confirming accuracy is retained

## Limitations

- The kmer representation is an exact-match methodology that may be limited in sensitivity and coverage; AAR recoding links sequences with distant similarity but collapses the 20-letter alphabet
- Absolute wall-clock runtimes are hardware- and load-dependent, so only the relative speedup ratio (not raw seconds) generalizes
- The reported near-4x speedup was measured on the nitrogen cycling protein families; other datasets or k values may shift the ratio

## Evidence

- [abstract] Here, we describe Snekmer, a software tool for recoding proteins into AAR kmer vectors and performing either (i) construction of supervised classification models trained on input protein families: "Here, we describe Snekmer, a software tool for recoding proteins into AAR kmer vectors and performing either (i) construction of supervised classification models trained on input protein families"
- [abstract] this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage: "this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage"
- [intro] Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach.: "Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach."
- [supplementary] Models built from MIQS-recoded 4-mers perform comparably to unencoded 4-mers, but the MIQS-recoded feature generation step is nearly 4 times faster (Supplementary Table S2).: [evidence span withheld — anchor status: not_found]
- [readme] There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,"
