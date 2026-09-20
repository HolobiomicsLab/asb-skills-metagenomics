---
name: microbial-network-algorithm-configuration
description: Use when configuring FlashWeave's learn_network call for OTU abundance
  data and you must decide between sensitive/fine-grained vs fast modes, or whether
  multi-habitat/multi-protocol data requires the heterogeneous (HE) variant.
when_to_use_negative:
- The data set is single-habitat and small (< thousands of samples) — the heterogeneous
  (FlashWeaveHE) modes are not indicated.
- The goal is running the pipeline on an already-configured network inference call
  rather than deciding which algorithm mode to use.
- Missing entries remain in the OTU or meta data tables — FlashWeave does not support
  missing data, so samples must be removed before any mode configuration matters.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3174
tools:
- name: FlashWeave
  role: Target tool whose sensitive/heterogeneous/max_k parameters are dispatched
    into the four algorithm modes; configured via learn_network
  repo: https://github.com/meringlab/FlashWeave.jl
- name: Julia 1.6
  role: Runtime language in which the dispatch function and mode_dispatch_test.jl
    unit tests are implemented (FlashWeave supports Julia >= 1.6)
  repo: https://github.com/JuliaLang/julia
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cels.2019.08.002
    title: Rapid Inference of Direct Interactions in Large-Scale Ecological Networks
      from Heterogeneous Microbial Sequencing Data
  source_content_hash: sha256:8c48ae55c312ea11d7e5505688c339873c31c96164b5ea4de2fd6c2ad60eb3f5
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# microbial-network-algorithm-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps FlashWeave's (sensitive, heterogeneous) flag pair onto its four algorithm modes — FlashWeave-F, FlashWeave-S, FlashWeaveHE-F, FlashWeaveHE-S — including the max_k=0 univariate special case that forces the fast (F) variant. Correct mode selection matches the probabilistic graphical model to data scale and metadata structure.

## When to use

Use when configuring FlashWeave's learn_network call for OTU abundance data and you must decide between sensitive/fine-grained vs fast modes, or whether multi-habitat/multi-protocol data requires the heterogeneous (HE) variant.

## When NOT to use

- The data set is single-habitat and small (< thousands of samples) — the heterogeneous (FlashWeaveHE) modes are not indicated.
- The goal is running the pipeline on an already-configured network inference call rather than deciding which algorithm mode to use.
- Missing entries remain in the OTU or meta data tables — FlashWeave does not support missing data, so samples must be removed before any mode configuration matters.

## Inputs

- OTU table (delimited .tsv/.csv or BIOM 1.0) of compositional microbial abundances
- Optional meta data table (environmental/technical factors such as habitat or protocol)
- Boolean parameter pair (sensitive, heterogeneous) and integer max_k

## Outputs

- Selected FlashWeave algorithm mode (FlashWeave-F, FlashWeave-S, FlashWeaveHE-F, or FlashWeaveHE-S)
- Configured learn_network invocation with the chosen flags
- Unit-test assertions in mode_dispatch_test.jl covering each flag-pair combination and the max_k=0 override

## How to apply

Read the learn_network parameter definitions for sensitive, heterogeneous, and max_k from the FlashWeave documentation, then dispatch per the flag pair: sensitive=false, heterogeneous=false → FlashWeave-F; sensitive=true, heterogeneous=false → FlashWeave-S; sensitive=false, heterogeneous=true → FlashWeaveHE-F; sensitive=true, heterogeneous=true → FlashWeaveHE-S. Enable heterogeneous mode only for multi-habitat or multi-protocol data with at least thousands of samples, since FlashWeaveHE is designed for such scale. Apply the max_k=0 special case: it results in no conditioning (univariate mode), which forces the F (fast) variant regardless of the sensitive flag. The mapping is order-agnostic over the two boolean flags, and the dispatch reflects the two underlying dimensions of the method — a flexible Probabilistic Graphical Model framework that integrates metadata and predicts direct interactions from heterogeneous microbial abundance data.

## Related tools

- **FlashWeave** (Target tool whose sensitive/heterogeneous/max_k parameters are dispatched into the four algorithm modes; configured via learn_network) — https://github.com/meringlab/FlashWeave.jl
- **Julia 1.6** (Runtime language in which the dispatch function and mode_dispatch_test.jl unit tests are implemented (FlashWeave supports Julia >= 1.6)) — https://github.com/JuliaLang/julia

## Examples

```
netw_results = learn_network(data_path, meta_data_path, sensitive=true, heterogeneous=false); # max_k=0 would force the fast (F) variant in univariate mode
```

## Evaluation signals

- Each order-agnostic (sensitive, heterogeneous) pair maps to the expected mode: (false,false)→FlashWeave-F, (true,false)→FlashWeave-S, (false,true)→FlashWeaveHE-F, (true,true)→FlashWeaveHE-S.
- Setting max_k=0 forces an F (fast) variant even when sensitive=true (univariate, no-conditioning mode).
- All assertions in mode_dispatch_test.jl pass under Julia 1.6 or above.
- heterogeneous=true is only applied to multi-habitat or -protocol data with at least thousands of samples.
- The resulting learn_network call reproduces the documented parameter semantics (sensitive=false yields fast modes FlashWeave-F/FlashWeaveHE-F).

## Limitations

- From version 0.19 FlashWeave dropped support for Julia versions < 1.6; older Julia installations require FlashWeave 0.18.1 (or lower) via `add FlashWeave@0.18`.
- FlashWeave currently does not support missing data — all samples with missing entries in OTU or meta data tables must be removed prior to running.
- max_k values beyond 0 can strongly increase runtime since they enlarge conditioning-set sizes.

## Evidence

- [readme] sensitive - enable fine-grained associations (FlashWeave-S, FlashWeaveHE-S), sensitive=false results in the fast modes FlashWeave-F or FlashWeaveHE-F: "sensitive - enable fine-grained associations (FlashWeave-S, FlashWeaveHE-S), sensitive=false results in the fast modes FlashWeave-F or FlashWeaveHE-F"
- [readme] heterogeneous - enable heterogeneous mode for multi-habitat or -protocol data with at least thousands of samples (FlashWeaveHE): "heterogeneous - enable heterogeneous mode for multi-habitat or -protocol data with at least thousands of samples (FlashWeaveHE)"
- [readme] max_k - maximum size of conditioning sets, high values can strongly increase runtime. max_k=0 results in no conditioning (univariate mode): "max_k - maximum size of conditioning sets, high values can strongly increase runtime. max_k=0 results in no conditioning (univariate mode)"
- [abstract] FlashWeave, a computational approach based on a flexible Probabilistic Graphical Model framework: "FlashWeave, a computational approach based on a flexible Probabilistic Graphical Model framework"
- [methods] Make sure you install Julia 1.6 or above, the versions currently supported by FlashWeave.: "Make sure you install Julia 1.6 or above, the versions currently supported by FlashWeave."
