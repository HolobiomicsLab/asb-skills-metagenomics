---
name: blast-all-against-all-identity-computation
description: 'Use when auditing a sequence clustering''s sensitivity at a fixed identity
  threshold (e.g., 50%): you need a BLAST-derived reference of clusterable sequence
  pairs (≥90% coverage of the shorter sequence) to compute per-tool false-negative
  fractions.'
when_to_use_negative:
- The goal is to produce a production clustering of a large sequence set rather than
  to validate/benchmark an existing clustering's false-negative rate
- The sequence set is small enough for exhaustive exact all-against-all comparison,
  making BLAST-based sampling estimation unnecessary
- The required coverage convention differs from cov-mode 1 (coverage relative to the
  shorter sequence), e.g., when coverage must be computed over the full representative
  length
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: BLAST
  role: Runs all-against-all alignments among the 1000 sampled cluster representatives
    to establish the reference set of threshold-satisfying (≥90% coverage) pairs
- name: MMseqs2
  role: Provides Linclust ('mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5')
    and the cascaded cluster workflow whose clustering outputs are evaluated for false
    negatives
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Benchmarked clustering tool run with '-T 16 -M 0 -n 3 -c 0.5' whose missed
    clusterable pairs are counted
- name: UCLUST 7.0.1090
  role: Benchmarked clustering tool run with '--id 0.5' whose missed clusterable pairs
    are counted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:b48b4595ef1af786ca126120f24f131c616ba011f735aac9f46d5dd0b6e73b8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - BLAST
  - CD-HIT
  - CD-HIT 4.6
  - MMseqs2
  - UCLUST
  - UCLUST 7.0.1090
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# blast-all-against-all-identity-computation

## Summary

Benchmark the sensitivity of sequence clustering tools by computing BLAST all-against-all identities among 1000 randomly sampled cluster representatives, filtering to pairs that satisfy the clustering threshold with ≥90% coverage (cov-mode 1), and measuring the fraction of these clusterable pairs each tool missed (false negatives).

## When to use

Use when auditing a sequence clustering's sensitivity at a fixed identity threshold (e.g., 50%): you need a BLAST-derived reference of clusterable sequence pairs (≥90% coverage of the shorter sequence) to compute per-tool false-negative fractions.

## When NOT to use

- The goal is to produce a production clustering of a large sequence set rather than to validate/benchmark an existing clustering's false-negative rate
- The sequence set is small enough for exhaustive exact all-against-all comparison, making BLAST-based sampling estimation unnecessary
- The required coverage convention differs from cov-mode 1 (coverage relative to the shorter sequence), e.g., when coverage must be computed over the full representative length

## Inputs

- FASTA-formatted protein sequence test set (123-million-sequence UniProt 2016_03 set)
- Per-tool clustering outputs (cluster membership / representative sequences from Linclust, MMseqs2, CD-HIT 4.6, UCLUST 7.0.1090)
- Random sample of 1000 representative sequences per tool for BLAST all-against-all comparison

## Outputs

- benchmark_fnr_results.csv with per-tool false-negative fractions (2%, 10%, 16%, 28% expected at 50% identity)
- Cumulative sequence-identity distributions over sampled representative pairs per tool
- Reference set of true homologous pairs passing the 90% minimum-coverage criterion

## How to apply

Cluster the 123-million-sequence UniProt 2016_03 test set at 50% identity with each tool — Linclust via 'mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5', the MMseqs2 cascaded 'mmseqs cluster' workflow, CD-HIT 4.6 with '-T 16 -M 0 -n 3 -c 0.5', and UCLUST 7.0.1090 with '--id 0.5'. Randomly sample 1000 representative sequences from each tool's clustering and run BLAST all-against-all among all sampled representatives. Filter BLAST hits to those satisfying the 90% minimum-coverage criterion (aligned residues divided by the length of the shorter sequence, matching MMseqs2 cov-mode 1) to define the reference set of true clusterable pairs. For each tool, compute the cumulative sequence-identity distribution over sampled representative pairs and the false-negative fraction (reference pairs missed by the clustering), writing both per-tool metrics to benchmark_fnr_results.csv. Compare against published values: 2% (Linclust/MMseqs2), 10% (UCLUST), 16% (Linclust-m80), 28% (Linclust) at 50% identity.

## Related tools

- **BLAST** (Runs all-against-all alignments among the 1000 sampled cluster representatives to establish the reference set of threshold-satisfying (≥90% coverage) pairs)
- **MMseqs2** (Provides Linclust ('mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5') and the cascaded cluster workflow whose clustering outputs are evaluated for false negatives) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Benchmarked clustering tool run with '-T 16 -M 0 -n 3 -c 0.5' whose missed clusterable pairs are counted)
- **UCLUST 7.0.1090** (Benchmarked clustering tool run with '--id 0.5' whose missed clusterable pairs are counted)

## Examples

```
mmseqs linclust uniprot2016_03.fa clu_50 tmp --cov-mode 1 -c 0.9 --min-seq-id 0.5
```

## Evaluation signals

- benchmark_fnr_results.csv exists and contains an entry for every benchmarked tool
- Every false-negative fraction lies between 0 and 1
- False-negative fractions are consistent with published values at 50% identity: 2% Linclust/MMseqs2, 10% UCLUST, 16% Linclust-m80, 28% Linclust
- BLAST hits are retained only when aligned residues divided by the length of the shorter sequence ≥ 0.9 (cov-mode 1 semantics)
- Results reproduce against the scripts and benchmark data deposited at https://bitbucket.org/martin_steinegger/linclust-analysis

## Limitations

- False-negative rates are estimated on a random sample of 1000 representatives, not an exhaustive all-against-all comparison of the 123-million-sequence test set, so sampling noise affects the estimate
- The reference set only includes pairs passing the 90% minimum-coverage criterion (aligned residues / shorter sequence); pairs failing coverage are excluded from the false-negative accounting
- BLAST defines the 'true' clusterable pairs approximately; approximate k-mer prefilter algorithms can miss pairs below the sampling detection limit

## Evidence

- [supplementary] sampling 1000 representatives for BLAST all-against-all: [evidence span withheld — anchor status: not_found]
- [supplementary] 90% minimum-coverage filtering of BLAST hits (cov-mode 1): [evidence span withheld — anchor status: not_found]
- [methods] UniProt 2016_03 clustered at 90%, 70%, 50% identity: "The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated."
- [methods] Linclust command-line parameters: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] benchmark scripts and data availability: "All scripts and benchmark data including command-line parameters necessary to reproduce the benchmark and analysis results presented here are available at"
- [methods] CD-HIT 4.6 parameters including the 50% identity setting: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [readme] MMseqs2 as the fast search/clustering suite: "MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets."
