---
name: speedup-ratio-analysis
description: Use when comparing the computational cost of Snekmer's vectorization
  stage across amino-acid-recoding parameterizations (e.g., alphabet miqs vs None
  at k=4) and you must quantify and validate a speedup ratio against a published benchmark.
when_to_use_negative:
- The goal is to compare model accuracy/score distributions rather than feature-generation
  runtime — use the supervised modeling evaluation instead.
- Inputs are already-vectorized kmer feature tables, so no vectorization stage will
  be run and no timing comparison is possible.
- Comparing parameterizations other than alphabet-recoding settings (e.g., different
  k values or clustering algorithms), where the near-4x expectation from Supplementary
  Table S2 does not apply.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Snekmer
  role: Runs the vectorization stage being benchmarked via `snekmer {mode} --until
    vectorize` with k=4 under alphabet=miqs and alphabet=None; the --until flag limits
    execution to the feature-generation step whose wall-clock time is measured.
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Workflow engine underlying Snekmer; the `--until vectorize` argument is passed
    through directly to Snakemake to stop the DAG after feature generation.
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Runtime environment (Python 3.11+, venv) in which Snekmer is installed and
    used to compute the speedup ratio and write snekmer_feature_timing.csv.
  repo: https://github.com/PNNL-CompBio/Snekmer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:2148cd86e44a01376a047a81241c6da738d9eedfcea9fbf09e5e7de06792239d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Python
  - Snakemake
  - Snekmer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# speedup-ratio-analysis

## Summary

Benchmarks the wall-clock runtime of Snekmer's feature-generation (vectorization) step for MIQS amino-acid-recoded 4-mers versus unencoded native-sequence 4-mers on nitrogen cycling protein families, then computes a speedup ratio to reproduce the near-4x advantage reported in Supplementary Table S2.

## When to use

Use when comparing the computational cost of Snekmer's vectorization stage across amino-acid-recoding parameterizations (e.g., alphabet miqs vs None at k=4) and you must quantify and validate a speedup ratio against a published benchmark.

## When NOT to use

- The goal is to compare model accuracy/score distributions rather than feature-generation runtime — use the supervised modeling evaluation instead.
- Inputs are already-vectorized kmer feature tables, so no vectorization stage will be run and no timing comparison is possible.
- Comparing parameterizations other than alphabet-recoding settings (e.g., different k values or clustering algorithms), where the near-4x expectation from Supplementary Table S2 does not apply.

## Inputs

- Nitrogen cycling protein family FASTA sequences (placed in the Snekmer input/ directory)
- Snekmer config.yaml with `k` set to 4 and `alphabet` set to miqs (encoding 5) or None
- Snekmer 1.4.1 installed in a Python 3.11+ venv via pip

## Outputs

- snekmer_feature_timing.csv — timing table with runtime entries for both parameterizations (miqs k=4 and unencoded k=4, optionally per-family) and the computed speedup ratio
- Snekmer vectorized kmer feature files for each alphabet setting from the --until vectorize runs

## How to apply

Place the nitrogen cycling protein family FASTA sequences into the Snekmer input/ directory and run only the vectorization stage with `snekmer {mode} --until vectorize`, first with `k` = 4 and `alphabet` set to miqs (encoding 5, reduced alphabet size 10), recording the wall-clock runtime; Snekmer orchestrates this via Snakemake, and Snakemake arguments such as `--until` are passed through directly to Snakemake. Repeat the identical run with `k` = 4 and `alphabet` set to None (full 20-letter alphabet), recording the runtime under the same hardware. Compute the speedup ratio as unencoded runtime divided by MIQS-recoded runtime. The rationale is that amino acid recoding shrinks the k-mer alphabet from 20 to 10 letters, collapsing the 4-mer feature space (20^4 = 160,000 native 4-mers vs at most 10^4 = 10,000 recoded 4-mers), which is what yields the near-4x feature-generation speedup reported in Supplementary Table S2 while model performance remains comparable. Assemble a timing table with per-parameterization (optionally per-family) runtimes plus the computed ratio and save it as snekmer_feature_timing.csv.

## Related tools

- **Snekmer** (Runs the vectorization stage being benchmarked via `snekmer {mode} --until vectorize` with k=4 under alphabet=miqs and alphabet=None; the --until flag limits execution to the feature-generation step whose wall-clock time is measured.) — https://github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Workflow engine underlying Snekmer; the `--until vectorize` argument is passed through directly to Snakemake to stop the DAG after feature generation.) — https://github.com/PNNL-CompBio/Snekmer
- **Python** (Runtime environment (Python 3.11+, venv) in which Snekmer is installed and used to compute the speedup ratio and write snekmer_feature_timing.csv.) — https://github.com/PNNL-CompBio/Snekmer

## Examples

```
snekmer model --until vectorize  # with config.yaml: k: 4; alphabet: miqs (then repeat with alphabet: None) and record wall-clock runtime of each run
```

## Evaluation signals

- snekmer_feature_timing.csv contains a runtime entry for both parameterizations (alphabet=miqs, k=4 and alphabet=None, k=4) plus a computed speedup ratio column.
- The MIQS-recoded 4-mer vectorization run completes approximately 4x faster than the unencoded 4-mer run, matching the near-4x speedup in Supplementary Table S2.
- Both runs are executed with identical inputs (the same nitrogen cycling FASTA files), identical k=4 setting, and the --until vectorize stage boundary, so timings are directly comparable.
- The MIQS setting corresponds to encoding 5 with reduced alphabet size 10; verify the config used reports alphabet size 10 for miqs and 20 for None.
- Runtimes should be measured as wall-clock time of the feature-generation step only, excluding Snakemake DAG setup and any downstream model/clustering stages.

## Limitations

- The reported speedup reflects feature-generation time only; the near-4x figure applies to the vectorization step and does not guarantee similar savings in model training or search stages.
- Timings are hardware-dependent, so the ratio should be compared against the published ~4x expectation rather than absolute runtimes, and both runs must use the same machine.
- Snekmer's kmer approach is exact-match and may be limited in sensitivity and coverage; the speedup from recoding comes alongside reduced alphabet resolution, though model performance was reported as comparable.

## Evidence

- [supplementary] MIQS-recoded 4-mers outperform speed expectation: [evidence span withheld — anchor status: not_found]
- [methods] Snakemake passthrough for stage limiting: "Snakemake arguments are passed through directly to Snakemake (they are not Snekmer-specific)."
- [readme] Six operation modes of Snekmer: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,"
- [abstract] Implementation stack: "Snekmer is written in Python using Snakemake."
