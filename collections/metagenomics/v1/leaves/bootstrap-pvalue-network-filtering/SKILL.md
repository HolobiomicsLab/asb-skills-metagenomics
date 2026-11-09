---
name: bootstrap-pvalue-network-filtering
description: Use when converting a bootstrapped interaction/correlation matrix from
  a network inference method (sparcc, propr, spieceasi, flashweave, cozine, harmonies,
  spring, mldm) into a significant co-occurrence network via Resample bootstraps,
  per-edge p-values, and threshold filtering.
when_to_use_negative:
- Input is still raw 16S sequence data or an unfiltered count table — the Resample/Pvalue
  steps operate on the output of an inference tool, not on reads (run the full or
  ta_op_ni workflow instead).
- The input is already a binary or p-value-filtered network/edge list — re-running
  Resample and Make network would double-filter and discard edge weights.
- You only need an interaction (correlation) matrix without significance assessment
  — use the correlation/direct inference step alone rather than bootstrapping 1000
  times.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3301
tools:
- name: MiCoNE
  role: Provides the Resample (bootstraps=1000), Pvalue, and pvalue-based Make network
    steps that turn each inferred interaction matrix into a threshold-filtered co-occurrence
    network
  repo: https://github.com/segrelab/MiCoNE
- name: Nextflow
  role: Executes the ni pipeline template containing the Bootstrap (Resample, Pvalue)
    and Make network processes
- name: Python
  role: MiCoNE Python library used to load interaction matrices/edge lists into Network
    objects and write filtered network files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:d0c7e9e2f8e66ffbdbede4fb39bbf10359b1c1d12a8603b3dd1e3c2f62a792bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - COZINE
  - FlashWeave
  - HARMONIES
  - MiCoNE
  - Nextflow
  - Python
  - SPRING
  - SparCC
  - SpiecEasi
  - mLDM
  - propr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# bootstrap-pvalue-network-filtering

## Summary

MiCoNE's significance-based network construction step: each network inference result is preceded by 1000 Resample bootstraps and Pvalue computation, then converted into a co-occurrence network keeping only interactions passing pvalue_threshold 0.05 and interaction_threshold 0.3.

## When to use

Use when converting a bootstrapped interaction/correlation matrix from a network inference method (sparcc, propr, spieceasi, flashweave, cozine, harmonies, spring, mldm) into a significant co-occurrence network via Resample bootstraps, per-edge p-values, and threshold filtering.

## When NOT to use

- Input is still raw 16S sequence data or an unfiltered count table — the Resample/Pvalue steps operate on the output of an inference tool, not on reads (run the full or ta_op_ni workflow instead).
- The input is already a binary or p-value-filtered network/edge list — re-running Resample and Make network would double-filter and discard edge weights.
- You only need an interaction (correlation) matrix without significance assessment — use the correlation/direct inference step alone rather than bootstrapping 1000 times.

## Inputs

- genus-level OTU/abundance table (otu_table.tsv) from the FMT dataset
- observation metadata with Kingdom-to-Species taxonomy columns (obs_metadata.tsv)
- bootstrapped interaction/correlation matrix from an NI tool (e.g. sparcc, propr, spieceasi 'mb', flashweave, cozine, harmonies, spring, mldm)
- per-interaction bootstrap p-value matrix (Resample bootstraps=1000, Pvalue slim=false)

## Outputs

- per-method filtered network JSON files with 'source', 'target', 'weight' edge fields, compatible with the MIND database
- grouped edge lists (nodes at genus level) used for cross-method UpSet intersection analysis of nodes and edges

## How to apply

Within the MiCoNE Nextflow pipeline (initialized with 'micone init -w ni'), precede each inference tool run with 1000 Resample bootstraps and a Pvalue computation step (slim=false, keeping full p-value information). Convert each bootstrapped result into a network using the pvalue-based Make network step with pvalue_threshold 0.05 (interaction p-value must pass significance) and interaction_threshold 0.3 (association strength must exceed 0.3). Group nodes at the genus taxonomic level using tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species'] and write each filtered network with network_group.write('network.', pvalue_filter=True, interaction_filter=False). The rationale is that bootstrapping distinguishes associations robust to compos-data resampling from noise, since different tools and parameters can lead to vastly different co-occurrence inferences from the same abundance table.

## Related tools

- **MiCoNE** (Provides the Resample (bootstraps=1000), Pvalue, and pvalue-based Make network steps that turn each inferred interaction matrix into a threshold-filtered co-occurrence network) — https://github.com/segrelab/MiCoNE
- **Nextflow** (Executes the ni pipeline template containing the Bootstrap (Resample, Pvalue) and Make network processes)
- **Python** (MiCoNE Python library used to load interaction matrices/edge lists into Network objects and write filtered network files)

## Examples

```
micone init -w ni -o ni_pipeline; bash run.sh
```

## Evaluation signals

- All eight per-method network JSON files exist and each contains 'source', 'target', 'weight' edge fields.
- Every retained edge satisfies p-value <= 0.05 (pvalue_threshold) and interaction strength >= 0.3 (interaction_threshold).
- The Bootstrap config records bootstraps=1000 and the Pvalue step runs with slim=false for each inference method.
- UpSet intersection counts over the eight filtered networks qualitatively match the Fig. 4B/C pattern (e.g. only 8 of 202 unique edges shared across all methods, 33 of 68 nodes shared, no method producing a unique node).
- 'micone validate-results' passes on the pipeline run directory.

## Limitations

- Threshold choices (pvalue_threshold 0.05, interaction_threshold 0.3) drive substantial rewiring across methods: different tools inferred largely non-overlapping edge sets, so a passing threshold does not guarantee cross-method agreement.
- Bootstrap resampling (1000 iterations) is computationally expensive and is applied per method and per sample subset, increasing run time of the Nextflow pipeline.
- The published comparison used specific defaults (e.g. sparcc iterations 50, spieceasi nlambda 20, harmonies iterations 10000); changing tool parameters upstream changes which edges survive filtering.

## Evidence

- [methods] Bootstrap Resample bootstraps 1000: "| Bootstrap   | Resample                     | bootstraps            | 1000         |"
- [methods] Bootstrap Pvalue slim false: "| Bootstrap   | Pvalue                       | slim                  | false        |"
- [methods] Loading interaction matrix into Network object: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"
- [readme] MiCoNE purpose from README: "It incorporates various popular, publicly available tools as well as custom Python modules and scripts to facilitate inference of co-occurrence networks from 16S data."
