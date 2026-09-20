---
name: consensus-network-construction
description: Use when you have association/interaction matrices (plus pvalue matrices)
  from multiple network inference methods on the same taxa and need to merge them
  into a single robust consensus network.
when_to_use_negative:
- You have only a single inference method's interaction matrix and no multi-method
  consensus is needed — use direct Network import instead
- The NI networks were not built on the same set of taxa (id_field), so edges cannot
  be aligned across methods
- You are still in the sequence processing (SP) or OTU assignment stages and have
  not yet produced interaction/pvalue matrices
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MiCoNE
  role: Provides the Network/NetworkGroup objects, the 'Create consensus' task (SV
    and scaled_sum algorithms with theta, pvalue_filter, interaction_filter parameters)
    and JSON export to MIND
  repo: https://github.com/segrelab/MiCoNE
- name: Python
  role: Library in which Network objects are loaded from tsv/csv interaction matrices
    and the consensus pipeline is implemented
  repo: https://github.com/segrelab/MiCoNE
- name: spieceasi
  role: One of the NI methods whose output association matrices feed the consensus;
    example data shipped with SPIEC-EASI (amgut1.filt) fit the NorTA synthetic benchmark
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:ecdd0352685526e23ae9fb094eb5bcb875497ae335b3bda6a7da1bd4be0daeb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - NetworkGroup
  - Python
  - spieceasi
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# consensus-network-construction

## Summary

Construct a consensus co-occurrence network from multiple network-inference (NI) association matrices using MiCoNE's simple voting (SV) or scaled-sum (SS) algorithms, retaining edges that pass method-count or thresholded-weight criteria. The consensus improves robustness over any single inference method, as benchmarked on NorTA and seqtime synthetic data.

## When to use

Use when you have association/interaction matrices (plus pvalue matrices) from multiple network inference methods on the same taxa and need to merge them into a single robust consensus network.

## When NOT to use

- You have only a single inference method's interaction matrix and no multi-method consensus is needed — use direct Network import instead
- The NI networks were not built on the same set of taxa (id_field), so edges cannot be aligned across methods
- You are still in the sequence processing (SP) or OTU assignment stages and have not yet produced interaction/pvalue matrices

## Inputs

- Eight filtered NI association matrices (interaction matrices plus pvalue matrices) in tsv/csv format
- Consensus configuration: method ('scaled_sum' or simple voting), theta (default 0.333), pvalue_filter=true, interaction_filter=true, id_field 'taxid'

## Outputs

- Consensus network multigraph in JSON format compatible with the MIND database
- Precision/sensitivity table per simulator and per theta (precision_sensitivity_table.csv) when benchmarked against synthetic data

## How to apply

Load the filtered NI association matrices (interaction matrices plus pvalue matrices) into MiCoNE Network objects with Network.load_data and aggregate them in a NetworkGroup. For simple voting (SV), compute a sign-based voting count M_j for each candidate edge across the w=8 networks and retain edges where M_j >= floor(theta x w). For scaled-sum (SS), max-normalize each network's interaction scores S_i, sum the scaled weights s_j across networks, and retain edges where |s_j| > (w-1) x theta; the default configuration uses method='scaled_sum', theta=0.333, pvalue_filter=true, interaction_filter=true. Assemble only qualifying edges into a multigraph annotated with per-method interaction weights, then export with network_group.write to a JSON file compatible with the MIND database. Sweep theta (e.g. 0.333 vs 1.000) and compute precision/sensitivity against ground-truth networks when benchmark data are available — SS reaches precision 1.000 at theta=1.000 and 0.956 (NorTA) / 0.688 (seqtime) at theta=0.333.

## Related tools

- **MiCoNE** (Provides the Network/NetworkGroup objects, the 'Create consensus' task (SV and scaled_sum algorithms with theta, pvalue_filter, interaction_filter parameters) and JSON export to MIND) — https://github.com/segrelab/MiCoNE
- **Python** (Library in which Network objects are loaded from tsv/csv interaction matrices and the consensus pipeline is implemented) — https://github.com/segrelab/MiCoNE
- **spieceasi** (One of the NI methods whose output association matrices feed the consensus; example data shipped with SPIEC-EASI (amgut1.filt) fit the NorTA synthetic benchmark)

## Examples

```
from micone import Network, NetworkGroup; nets = [Network.load_data(f'method{i}_interaction.csv', f'method{i}_pvalue.csv') for i in range(8)]; ng = NetworkGroup(nets); consensus = ng.create_consensus(method='scaled_sum', parameter=0.333, pvalue_filter=True, interaction_filter=True, id_field='taxid'); ng.write('consensus.')
```

## Evaluation signals

- Output JSON exists and parses as a MIND-compatible multigraph
- Every retained edge passes the applicable threshold: M_j >= floor(theta x w) for SV or |s_j| > (w-1) x theta for SS (with w=8, theta=0.333 gives |s_j| > 2.331)
- Each retained edge carries weights from the contributing inference methods
- On synthetic benchmarks (NorTA, seqtime) with 1000 bootstrap resamples, interaction_threshold=0.3, pvalue_threshold=0.05, scaled_sum precision matches Fig. 5: 1.000 at theta=1.000 for both simulators, 0.956 (NorTA) and 0.688 (seqtime) at theta=0.333, without significant sensitivity loss
- Micone's 'validate-results' subcommand reports the pipeline run as valid

## Limitations

- Consensus quality depends on theta: at default theta=0.333 precision drops to 0.688 on seqtime synthetic data (vs 0.956 on NorTA), so theta may need sweeping per dataset
- MiCoNE requires the mamba package manager (otherwise 'micone init' will not work), and the pipeline data directory must be manually downloaded from Zenodo (record 7051556)
- The Python-only library (pip install micone) does not provide functionality to execute the full pipelines

## Evidence

- [methods] SV threshold rule: [evidence span withheld — anchor status: not_found]
- [methods] SS algorithm details: [evidence span withheld — anchor status: not_found]
- [results] Benchmark precision: [evidence span withheld — anchor status: not_found]
- [methods] JSON export to MIND: "This ``JSON`` file is compatible with the ``MIND`` database and can be uploaded and visualized directly on the website"
- [abstract] Consensus rationale: "We also develop a consensus network algorithm that helps generate more robust co-occurrence networks based on benchmark synthetic data sets"
- [readme] README citation: "Kishore, D., Birzu, G., Hu, Z., DeLisi, C., Korolev, K., & Segrè, D. (2023). Inferring microbial co-occurrence networks from amplicon data: A systematic evaluation. mSystems."
