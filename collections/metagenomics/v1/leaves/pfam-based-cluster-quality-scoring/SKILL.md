---
name: pfam-based-cluster-quality-scoring
description: Use when protein sequence clustering output must be quality-assessed
  against independent functional annotations (Pfam), e.g. to compare MMseqs2 easy-cluster
  runs across a sensitivity parameter sweep or to choose stable clustering settings.
when_to_use_negative:
- No Pfam (or equivalent domain-family) annotations exist for the input proteins —
  the V score requires an external functional ground truth
- Evaluating a supervised classification/model step (e.g. Snekmer apply predictions
  scored by cosine similarity) rather than de novo cluster assignments
- Input is nucleotide sequence or the clusters come from a non-protein workflow; V-score
  benchmarking here is defined for protein clustering
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MMseqs2
  role: easy-cluster is the clustering engine whose output clusters are scored against
    Pfam annotations across the sensitivity sweep
- name: Python
  role: pandas is used to assemble per-sensitivity coverage and Pfam V values into
    the summary CSV table
- name: Snekmer
  role: Alternative AAR-kmer-based protein clustering tool surfaced by the same article;
    provides unsupervised clustering of protein sequences that can be benchmarked
    the same way
  repo: https://github.com/PNNL-CompBio/Snekmer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:dea98f630cc5c1c99021055f6f23c8dc13e7ed3025f28148cdb578652cc91e10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - MMseqs2
  - Python
  - Snekmer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# pfam-based-cluster-quality-scoring

## Summary

Evaluates protein clustering results (e.g. MMseqs2 easy-cluster output) against Pfam domain annotations using the V-score, providing an external ground-truth-based measure of cluster quality that can be tracked across parameter sweeps.

## When to use

Use when protein sequence clustering output must be quality-assessed against independent functional annotations (Pfam), e.g. to compare MMseqs2 easy-cluster runs across a sensitivity parameter sweep or to choose stable clustering settings.

## When NOT to use

- No Pfam (or equivalent domain-family) annotations exist for the input proteins — the V score requires an external functional ground truth
- Evaluating a supervised classification/model step (e.g. Snekmer apply predictions scored by cosine similarity) rather than de novo cluster assignments
- Input is nucleotide sequence or the clusters come from a non-protein workflow; V-score benchmarking here is defined for protein clustering

## Inputs

- Multi-species FASTA file of protein sequences (e.g. a 10-genome protein set)
- Pfam annotations for the same protein set
- Per-run clustering outputs (e.g. MMseqs2 easy-cluster cluster assignments at sensitivity values 1–7.5)

## Outputs

- Per-run protein coverage counts (number of clustered proteins) per sensitivity setting
- Per-run Pfam V score values per sensitivity setting
- Tabular CSV summary (e.g. sensitivity_sweep_table.csv) with sensitivity, coverage, and Pfam V columns

## How to apply

For each clustering run (e.g. MMseqs2 easy-cluster at a given sensitivity value from 1 to 7.5), score the resulting clusters against Pfam annotations of the same 10-genome protein set using the Pfam V score, a combined homogeneity/completeness metric for cluster-to-family agreement. Record protein coverage (number of clustered proteins) and the V score per parameter setting. Assemble the per-setting values into a tabular summary with Python (pandas) and write it to a CSV such as sensitivity_sweep_table.csv. Rationale: in the source study, the V score varied only from 0.90 to 0.92 across sensitivities 1–7.5 even while coverage ranged from 20,240 to 28,977 proteins, demonstrating that this external functional benchmark is a stable criterion for judging whether parameter changes meaningfully affect cluster quality.

## Related tools

- **MMseqs2** (easy-cluster is the clustering engine whose output clusters are scored against Pfam annotations across the sensitivity sweep)
- **Python** (pandas is used to assemble per-sensitivity coverage and Pfam V values into the summary CSV table)
- **Snekmer** (Alternative AAR-kmer-based protein clustering tool surfaced by the same article; provides unsupervised clustering of protein sequences that can be benchmarked the same way) — https://github.com/PNNL-CompBio/Snekmer

## Examples

```
mmseqs easy-cluster proteins.fasta cluster_out tmp -s 7.5
```

## Evaluation signals

- Protein coverage column in the summary table falls within the reported range of 20,240 to 28,977 proteins across the swept sensitivities
- Pfam V score column falls within 0.90 to 0.92 across all swept sensitivity settings
- The output CSV (sensitivity_sweep_table.csv) contains one row per swept sensitivity value with paired coverage and V-score entries
- Each clustering run used identical protein input so only the sensitivity parameter varied between runs

## Limitations

- Coverage (number of clustered proteins) is highly sensitivity-dependent — from 20,240 to 28,977 proteins across the 1–7.5 sweep — so it is a weak standalone quality signal compared with the V score
- V-score evaluation depends entirely on the completeness and accuracy of the Pfam annotation set; proteins without Pfam families cannot be externally validated
- Representing proteins as exact-match kmer sets may be limited in sensitivity and coverage; amino acid recoding links sequences with distant similarity, which affects which clustering algorithm is appropriate before scoring

## Evidence

- [other] Across MMseqs2 sensitivity settings of 1 to 7.5, protein coverage ranged from a minimum of 20,240 to a maximum of 28,977 proteins, and the Pfam V score ranged only from 0.90 to 0.92.: [evidence span withheld — anchor status: not_found]
- [other] For each run, evaluate cluster quality against Pfam annotations using the Pfam V score and record the value per sensitivity setting.: [evidence span withheld — anchor status: not_found]
- [other] Assemble the per-sensitivity coverage and Pfam V values into a tabular summary with Python (pandas) and write it to sensitivity_sweep_table.csv.: [evidence span withheld — anchor status: not_found]
- [intro] Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms,: "Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms,"
- [abstract] Snekmer is written in Python using Snakemake.: "Snekmer is written in Python using Snakemake."
