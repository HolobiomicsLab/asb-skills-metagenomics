---
name: computation-benchmark-timing
description: Use when you need to empirically quantify how an amino acid recoding
  alphabet changes the compute cost of kmer vectorization, e.g. verifying the near-4x
  speedup of MIQS-recoded 4-mers over unencoded 4-mers on nitrogen cycling protein
  FASTA inputs.
when_to_use_negative:
- The goal is comparing model accuracy rather than runtime — the benchmark only measures
  the vectorization stage and does not itself evaluate classification performance.
- The input is already a feature/kmer vector table, so no vectorization step exists
  to time.
- The analysis requires full pipeline results (clustering, model, apply outputs like
  results/apply/snekmer_results.csv), since `--until vectorize` deliberately halts
  the DAG before downstream stages.
tools:
- name: Snekmer
  role: Executes the vectorization (feature-generation) stage being benchmarked, with
    `--until vectorize` limiting execution to that stage under each k/alphabet parameterization.
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Workflow engine underlying Snekmer; Snakemake arguments such as `--until`
    are passed through directly to Snakemake (they are not Snekmer-specific) to restrict
    the run to vectorization.
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Runtime (3.11+) and scripting environment used to compute the speedup ratio
    and assemble the snekmer_feature_timing.csv table.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:c939201faebd1502dd6e1fce2304e9491b647d1f1acb9e5a211a6caba313af96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Python
  - Snakemake
  - Snekmer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# computation-benchmark-timing

## Summary

Benchmark the wall-clock runtime of a Snekmer feature-generation (vectorization) stage under two parameterizations — MIQS amino-acid-recoded 4-mers versus unencoded (native 20-letter) 4-mers — to quantify the computational benefit of amino acid recoding (AAR) reported in the article's Supplementary Table S2.

## When to use

Use when you need to empirically quantify how an amino acid recoding alphabet changes the compute cost of kmer vectorization, e.g. verifying the near-4x speedup of MIQS-recoded 4-mers over unencoded 4-mers on nitrogen cycling protein FASTA inputs.

## When NOT to use

- The goal is comparing model accuracy rather than runtime — the benchmark only measures the vectorization stage and does not itself evaluate classification performance.
- The input is already a feature/kmer vector table, so no vectorization step exists to time.
- The analysis requires full pipeline results (clustering, model, apply outputs like results/apply/snekmer_results.csv), since `--until vectorize` deliberately halts the DAG before downstream stages.

## Inputs

- Nitrogen cycling protein family FASTA sequences (placed in the Snekmer input/ directory)
- Snekmer config.yaml parameterization: k=4; alphabet=miqs (encoding 5, reduced alphabet size 10) and alphabet=None (size 20)

## Outputs

- snekmer_feature_timing.csv with runtime entries for both parameterizations (optionally per-family) and the computed speedup ratio
- Recorded wall-clock runtime of the vectorization stage for the MIQS-recoded and unencoded 4-mer runs

## How to apply

Place the nitrogen cycling protein family FASTA sequences into the Snekmer input/ directory, then run only the vectorization stage with `snekmer {mode} --until vectorize` twice: once with k=4 and alphabet=miqs (encoding 5, reduced alphabet size 10), and once with k=4 and alphabet=None (no reduced alphabet, size 20), recording the wall-clock runtime of each feature-generation run. Because Snekmer orchestrates its steps via Snakemake, `--until` restricts the DAG so downstream clustering/modeling stages do not inflate the measurement. Compute the speedup as unencoded runtime divided by MIQS-recoded runtime and compare against the near-4x figure from Supplementary Table S2. Assemble the per-parameterization (optionally per-family) timings plus the ratio into a CSV (snekmer_feature_timing.csv) so model-quality context — MIQS-recoded 4-mers perform comparably to unencoded 4-mers — can be weighed against the compute savings.

## Related tools

- **Snekmer** (Executes the vectorization (feature-generation) stage being benchmarked, with `--until vectorize` limiting execution to that stage under each k/alphabet parameterization.) — https://github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Workflow engine underlying Snekmer; Snakemake arguments such as `--until` are passed through directly to Snakemake (they are not Snekmer-specific) to restrict the run to vectorization.) — https://github.com/PNNL-CompBio/Snekmer
- **Python** (Runtime (3.11+) and scripting environment used to compute the speedup ratio and assemble the snekmer_feature_timing.csv table.)

## Examples

```
snekmer model --until vectorize -k 4 --alphabet miqs  # then repeat with --alphabet None and time each run
```

## Evaluation signals

- The MIQS-recoded 4-mer vectorization completes approximately 4x faster than the unencoded 4-mer run, matching the near-4x speedup in Supplementary Table S2.
- snekmer_feature_timing.csv contains a runtime entry for both parameterizations (alphabet=miqs and alphabet=None, both k=4) and the computed speedup ratio.
- Both runs used identical inputs and the `--until vectorize` restriction, so the only differing factor is the alphabet (10-letter MIQS recode vs 20-letter native).
- Reported runtimes are wall-clock measurements of the feature-generation step only, with no downstream Snakemake jobs (clustering/modeling) included.

## Limitations

- Runtime ratios are machine- and load-dependent, so the observed speedup may deviate from the paper's near-4x figure on different hardware.
- The speedup applies to the vectorization step only; the article reports that model performance is comparable between MIQS-recoded and unencoded 4-mers, so timing does not justify accuracy differences.
- A working C/C++ toolchain may be required for some dependencies depending on platform, which can complicate reproducing the benchmark environment.

## Evidence

- [supplementary] near-4x speedup finding: [evidence span withheld — anchor status: not_found]
- [methods] Snakemake orchestration: "Snakemake arguments are passed through directly to Snakemake (they are not Snekmer-specific)."
- [readme] README orchestration note: [evidence span withheld — anchor status: external_doc]
- [abstract] Python implementation: "Snekmer is written in Python using Snakemake."
