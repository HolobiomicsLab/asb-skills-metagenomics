---
name: otu-abundance-table-normalization
description: Use when an OTU abundance table (e.g. .tsv, .csv or BIOM, such as HMP_SRA_gut_tiny.tsv)
  must be normalized automatically before FlashWeave-style statistical co-occurrence
  network inference.
when_to_use_negative:
- The input table still contains missing entries — FlashWeave requires all samples
  with missing entries (in OTU and meta data tables) to be removed prior to running
- The data are not compositional OTU counts from sequencing data (FlashWeave's statistical
  co-occurrence framework is designed for large-scale compositional abundance data)
- Network inference or downstream graph analysis (save_network, LightGraphs.jl, Cytoscape)
  is the actual goal rather than reproducing the normalization step
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: FlashWeave
  role: 'Source of the normalization behavior being reconstructed: learn_network''s
    normalize parameter automatically chooses and performs data normalization based
    on sensitive and heterogeneous flags; installed via the Julia package manager
    (pkg> add FlashWeave)'
  repo: https://github.com/meringlab/FlashWeave.jl
- name: Julia 1.6
  role: Runtime environment in which FlashWeave runs and in which the normalization
    step can be re-implemented; Julia 1.6 or above is required by FlashWeave
  repo: https://github.com/JuliaLang/julia
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.cels.2019.08.002
    title: Rapid Inference of Direct Interactions in Large-Scale Ecological Networks
      from Heterogeneous Microbial Sequencing Data
  source_content_hash: sha256:a648225ca53bbd464ee876d1bf2c0797d4f89145d3f440c026b4540a7d70e399
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/meringlab/FlashWeave.jl
  repo_url: https://github.com/meringlab/FlashWeave.jl
  tools:
  - FlashWeave
  - Julia 1.6
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# otu-abundance-table-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs FlashWeave's automatic pre-inference normalization of compositional OTU abundance tables, in which the normalization scheme is chosen automatically from the sensitive and heterogeneous flags (mirroring normalize=true in learn_network) before statistical co-occurrence/co-abundance network inference.

## When to use

Use when an OTU abundance table (e.g. .tsv, .csv or BIOM, such as HMP_SRA_gut_tiny.tsv) must be normalized automatically before FlashWeave-style statistical co-occurrence network inference.

## When NOT to use

- The input table still contains missing entries — FlashWeave requires all samples with missing entries (in OTU and meta data tables) to be removed prior to running
- The data are not compositional OTU counts from sequencing data (FlashWeave's statistical co-occurrence framework is designed for large-scale compositional abundance data)
- Network inference or downstream graph analysis (save_network, LightGraphs.jl, Cytoscape) is the actual goal rather than reproducing the normalization step

## Inputs

- Compositional OTU abundance table (.tsv, .csv, or BIOM 1.0), e.g. HMP_SRA_gut_tiny.tsv, rows as samples and columns as variables
- sensitive and heterogeneous mode flags (e.g. sensitive=true, heterogeneous=false) controlling the automatic normalization scheme

## Outputs

- normalized_otu_table.tsv: delimited normalized abundance matrix with dimensions matching the input OTU table, non-negative entries, no missing values

## How to apply

Load the input OTU table (rows as samples, columns as variables by default; set transposed=true if rows are variables) in .tsv/.csv or BIOM 1.0 format, using unique string row identifiers in the first column if present. First remove all samples with missing entries, since FlashWeave does not support missing data in either OTU or meta data tables. Select the normalization scheme automatically as a function of the sensitive and heterogeneous flags, replicating the 'normalize=true' behavior of learn_network, and apply the transform to the count data retaining a sparse representation (make_sparse=true in almost all cases). Write the resulting matrix to normalized_otu_table.tsv as a delimited file whose dimensions match the input table. The rationale is that FlashWeave analyzes large-scale compositional abundance data (OTU tables from sequencing data) through statistical co-occurrence or co-abundance, so the pre-inference transform must be deterministic, non-negative, and complete (no missing values).

## Related tools

- **FlashWeave** (Source of the normalization behavior being reconstructed: learn_network's normalize parameter automatically chooses and performs data normalization based on sensitive and heterogeneous flags; installed via the Julia package manager (pkg> add FlashWeave)) — https://github.com/meringlab/FlashWeave.jl
- **Julia 1.6** (Runtime environment in which FlashWeave runs and in which the normalization step can be re-implemented; Julia 1.6 or above is required by FlashWeave) — https://github.com/JuliaLang/julia

## Examples

```
julia> netw_results = learn_network("/my/example/data.tsv", sensitive=true, heterogeneous=false, normalize=true, make_sparse=true)
```

## Evaluation signals

- normalized_otu_table.tsv exists and parses as a delimited table (consistent with FlashWeave's accepted .tsv input conventions)
- Dimensions of the normalized table match the input OTU table exactly
- All entries are non-negative (compositional abundance values remain valid counts/proportions after the transform)
- No missing values remain in the output, consistent with FlashWeave's requirement that samples with missing entries are removed beforehand
- The normalization scheme selected matches what learn_network would pick automatically for the given sensitive/heterogeneous flag combination (normalize=true behavior)

## Limitations

- FlashWeave does not support missing data: all samples with missing entries in OTU or meta data tables must be removed before any processing
- Sparse representation (make_sparse) should be left at true in almost all cases; changing it can degrade behavior
- Normalization is chosen automatically from sensitive/heterogeneous rather than user-specified, so results depend on the mode flags; heterogeneous mode additionally requires at least thousands of samples
- The exact normalization transform is not detailed in the provided README beyond its dependence on the sensitive and heterogeneous flags

## Evidence

- [readme] normalize flag behavior: "normalize - automatically choose and perform data normalization (based on sensitive and heterogeneous)"
- [intro] input data type and statistical mechanism: "FlashWeave predicts ecological interactions between microbes from large-scale compositional abundance data (i.e. OTU tables constructed from sequencing data) through statistical co-occurrence or"
- [readme] sparse representation requirement: "make_sparse - use a sparse data representation (should be left at true in almost all cases)"
- [methods] missing data removal requirement: "FlashWeave currently does not support missing data, please remove all samples with missing entries (both in OTU and meta data tables) prior to running FlashWeave."
- [readme] transposition convention: "transposed - if true, rows of data are variables and columns are samples"
