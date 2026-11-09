---
name: representative-sequence-sampling
description: Use when you need to estimate how many clusterable sequence pairs a clustering
  tool misses (false-negative rate) on a protein set too large (e.g. 123M UniProt
  sequences) for exhaustive pairwise verification.
when_to_use_negative:
- The sequence set is small enough that exhaustive all-against-all pairwise comparison
  is feasible — then compute true cluster quality directly instead of sampling representatives.
- You need sensitivity/recall of a search tool against a curated ground truth (e.g.
  Swiss-Prot) rather than an estimate of pairs missed by a clustering prefilter.
- The clustering threshold differs from the BLAST filtering criteria — mismatched
  identity/coverage settings (e.g. 90% identity with -n 5 for CD-HIT vs 50% with -n
  3) make the reference set invalid for the comparison.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust (MMseqs2)
  role: Clustering tool whose representatives are sampled and whose false-negative
    rate at 50% identity (2% for Linclust/MMseqs2, 28% for Linclust alone) is estimated;
    run with 'mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5'
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Provides the cascaded cluster workflow ('mmseqs cluster') benchmarked alongside
    Linclust, whose representative sequences are sampled for BLAST-based false-negative
    estimation
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Comparison clustering tool run with '-T 16 -M 0 -n 3 -c 0.5'; its representatives
    are sampled to estimate its false-negative fraction
- name: UCLUST 7.0.1090
  role: Comparison clustering tool run with '--id 0.5'; its representatives are sampled
    to estimate its false-negative fraction (10% at 50% identity)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:bd3681ba81d12c86cfd0240691d5d4d28773dfdfeb0ad89d1ac08fde9477f99f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT
  - CD-HIT 4.6
  - Linclust (MMseqs2)
  - MMseqs2
  - UCLUST
  - UCLUST 7.0.1090
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# representative-sequence-sampling

## Summary

Randomly sample ~1000 representative sequences from each clustering tool's output and run BLAST all-against-all among them to build a trusted reference set of homologous pairs, enabling estimation of false-negative rates (missed clusterable pairs) for huge-scale sequence clustering tools such as Linclust, MMseqs2, CD-HIT, and UCLUST.

## When to use

Use when you need to estimate how many clusterable sequence pairs a clustering tool misses (false-negative rate) on a protein set too large (e.g. 123M UniProt sequences) for exhaustive pairwise verification.

## When NOT to use

- The sequence set is small enough that exhaustive all-against-all pairwise comparison is feasible — then compute true cluster quality directly instead of sampling representatives.
- You need sensitivity/recall of a search tool against a curated ground truth (e.g. Swiss-Prot) rather than an estimate of pairs missed by a clustering prefilter.
- The clustering threshold differs from the BLAST filtering criteria — mismatched identity/coverage settings (e.g. 90% identity with -n 5 for CD-HIT vs 50% with -n 3) make the reference set invalid for the comparison.

## Inputs

- FASTA-formatted protein sequence set (e.g. UniProt 2016_03, 123M sequences)
- Clustering results from each tool (representative/centroid sequence lists), produced at 50% sequence identity with the 90% minimum-coverage criterion (cov-mode 1, -c 0.9)
- BLAST all-against-all hits among the ~1000 sampled representative sequences per tool

## Outputs

- Per-tool false-negative fraction of clusterable sequence pairs missed (e.g. 2% Linclust/MMseqs2, 10% UCLUST, 16% Linclust-m80, 28% Linclust at 50% identity)
- Per-tool cumulative sequence-identity distribution over sampled representative pairs
- benchmark_fnr_results.csv with one entry per benchmarked tool

## How to apply

After clustering the full sequence set (e.g. 123-million-sequence UniProt 2016_03 test set) with each tool — Linclust via 'mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5', the MMseqs2 cascaded 'mmseqs cluster' workflow, CD-HIT 4.6 with '-T 16 -M 0 -n 3 -c 0.5', or UCLUST 7.0.1090 with '--id 0.5' — randomly sample 1000 representative (centroid) sequences from each tool's clusters. Run BLAST all-against-all among the sampled representatives and filter hits to those meeting the 90% minimum-coverage criterion (aligned residues divided by the length of the shorter sequence, cov-mode 1) and the target identity threshold, defining the reference set of true homologous pairs. For each tool, compute the cumulative sequence-identity distribution over sampled representative pairs and the false-negative fraction (reference pairs the clustering missed), writing both per-tool metrics to benchmark_fnr_results.csv. The rationale is that representative sequences stand in for their clusters, so a BLAST-verified missed pair among representatives estimates the fraction of clusterable pairs the approximate k-mer prefilter-based clustering fails to recover.

## Related tools

- **Linclust (MMseqs2)** (Clustering tool whose representatives are sampled and whose false-negative rate at 50% identity (2% for Linclust/MMseqs2, 28% for Linclust alone) is estimated; run with 'mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5') — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Provides the cascaded cluster workflow ('mmseqs cluster') benchmarked alongside Linclust, whose representative sequences are sampled for BLAST-based false-negative estimation) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Comparison clustering tool run with '-T 16 -M 0 -n 3 -c 0.5'; its representatives are sampled to estimate its false-negative fraction)
- **UCLUST 7.0.1090** (Comparison clustering tool run with '--id 0.5'; its representatives are sampled to estimate its false-negative fraction (10% at 50% identity))

## Examples

```
mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5 db clu tmp
```

## Evaluation signals

- benchmark_fnr_results.csv exists and contains an entry for every benchmarked tool (Linclust/MMseqs2, MMseqs2 cascaded, CD-HIT, UCLUST).
- Every reported false-negative fraction lies between 0 and 1.
- At 50% identity, values reproduce the expected ordering and magnitudes: 2% (Linclust/MMseqs2) < 10% (UCLUST) < 16% (Linclust-m80) < 28% (Linclust).
- BLAST reference pairs satisfy the 90% minimum-coverage criterion (aligned residues / length of shorter sequence, cov-mode 1) before being counted as true clusterable pairs.
- Exactly 1000 representative sequences were sampled per tool, drawn randomly from each tool's clustering output.

## Limitations

- The false-negative estimate is based on BLAST of only ~1000 sampled representatives, not exhaustive pairwise comparison, so it is an approximation and depends on sampling randomness.
- Linclust's prefilter only finds k-mer matches approximately; false pairs are not corrected by exact verification, so missed pairs surface as false negatives rather than false positives.
- If falsely clustered pairs are costly (e.g. for redundancy filtering), pairwise links need verification with the exact similarity measure, which has quadratic time complexity — sampling sidesteps this but cannot certify individual edges.

## Evidence

- [methods] BLAST-based FN estimation: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] CD-HIT parameters: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [discussion] k-mer prefilter approximate: "like the preclustering step in canopy clustering or Linclust’s prefilter to find k-mer matches, these algorithms are only approximate."
- [discussion] prefilter comparison factor: [evidence span withheld — anchor status: not_found]
- [methods] benchmark data availability: "All scripts and benchmark data including command-line parameters necessary to reproduce the benchmark and analysis results presented here are available at"
- [discussion] exact verification quadratic: "pairwise links need to be verified with the exact similarity measure, which still requires quadratic time complexity."
