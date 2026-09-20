---
name: euclidean-distance-error-computation
description: Use when relative abundance estimates per reference genome (values in
  [0,1]) have been produced from mapped-read counts and must be scored against known
  mock-community mixing proportions by Euclidean distance.
when_to_use_negative:
- Input lacks ground-truth mixing proportions for the mock community, so no distance
  can be computed.
- Abundance estimates are not normalized to the [0,1] range per genome, which would
  make Euclidean distances incomparable across tools.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3174
tools:
- name: RawHash
  role: Maps pooled raw nanopore signals to the combined reference whose mapping counts
    feed the abundance estimates being scored
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline raw-signal mapper run under default settings to produce comparison
    abundance estimates
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Baseline raw-signal mapper run under default settings to produce comparison
    abundance estimates
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:ddbe1f7cadc5ba953ae72dd5c99d219f1306b681176ea3ea84614c41bdfcc6dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# euclidean-distance-error-computation

## Summary

Computes Euclidean distance between a tool's estimated per-genome relative abundance vector and the ground-truth mixing proportions of a mock community, providing a single accuracy metric to rank raw-signal mappers such as RawHash, UNCALLED, and Sigmap.

## When to use

Use when relative abundance estimates per reference genome (values in [0,1]) have been produced from mapped-read counts and must be scored against known mock-community mixing proportions by Euclidean distance.

## When NOT to use

- Input lacks ground-truth mixing proportions for the mock community, so no distance can be computed.
- Abundance estimates are not normalized to the [0,1] range per genome, which would make Euclidean distances incomparable across tools.

## Inputs

- mapping.paf alignments (UNCALLED-style PAF) from RawHash/rawhash2, UNCALLED, and Sigmap against the combined reference
- ground-truth mock-community mixing proportions per genome
- list of genomes/species taxonomy IDs in the combined reference FASTA

## Outputs

- table3_relative_abundance.csv with one row per genome in the combined reference plus one Euclidean-distance row per tool (RawHash, UNCALLED, Sigmap)
- per-tool Euclidean distance to ground truth

## How to apply

After mapping pooled FAST5 raw signals (from d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94) against the combined, taxonomy-annotated reference FASTA with RawHash (rawhash2, R9.4 450bps 6-mer pore model index, 32 threads, -x sensitive), UNCALLED, and Sigmap, tally mapped reads per reference genome and normalize counts to per-genome relative abundances in [0,1]. Then compute, for each tool, the Euclidean distance sqrt(sum((estimate_i - truth_i)^2)) between the per-genome abundance vector and the ground-truth mixing proportions. The tool with the smallest distance (RawHash in the paper) is judged most accurate. Assemble per-genome abundances and per-tool distances into table3_relative_abundance.csv.

## Related tools

- **RawHash** (Maps pooled raw nanopore signals to the combined reference whose mapping counts feed the abundance estimates being scored) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline raw-signal mapper run under default settings to produce comparison abundance estimates) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline raw-signal mapper run under default settings to produce comparison abundance estimates) — https://github.com/haowenz/sigmap

## Examples

```
python compute_abundance_accuracy.py --paf mapping.paf --ref combined_reference.fasta --truth mock_community_proportions.csv --out table3_relative_abundance.csv
```

## Evaluation signals

- table3_relative_abundance.csv exists with one row per genome in the combined reference plus one distance row per tool
- all abundance values fall in the [0,1] range
- RawHash Euclidean distance is not worse than UNCALLED's or Sigmap's, consistent with the paper's reported accuracy comparison
- each tool's per-genome abundance vector sums to approximately 1 across the mock-community genomes
- distances are non-negative reals computed with the same ground-truth vector for all tools

## Limitations

- The distance depends on the read-pool composition and the reference set; mixing different test data directories (d1, d3, d4, d5) changes the ground truth.
- UNCALLED can only be applied to legacy r9.4.1 data, so comparisons on newer pore chemistries are out of scope.

## Evidence

- [results] RawHash provides the most accurate relative abundance estimations, with the smallest Euclidean distance to the ground truth, compared to UNCALLED and Sigmap on the mock community.: [evidence span withheld — anchor status: not_found]
- [abstract] We evaluate RawHash on three applications: (i) read mapping, (ii) relative abundance estimation, and (iii) contamination analysis.: "We evaluate RawHash on three applications: (i) read mapping, (ii) relative abundance estimation, and (iii) contamination analysis."
- [readme] Sigmap indexes with an ONT pore model and outputs PAF-style mapping information usable for abundance tallying.: [evidence span withheld — anchor status: not_found]
- [methods] RawHash index is built with the legacy R9.4 450bps 6-mer pore model under extern/kmer_models.: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
