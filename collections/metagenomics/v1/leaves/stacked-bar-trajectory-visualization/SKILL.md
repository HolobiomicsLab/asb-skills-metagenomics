---
name: stacked-bar-trajectory-visualization
description: Use when you have a longitudinal OTU/count table (first row sequence
  IDs, second row sampling times) and need per-mouse/per-subject stacked bar trajectory
  plots, optionally with external-event indicators.
when_to_use_negative:
- You need quantitative model output (parameter matrices A, g, B or predicted trajectories)
  rather than exploratory visualization — use the train/predict programs instead
- You have already-composed figure assets or a non-LUMINATE-format table lacking the
  required row-1/row-2 sequence-ID and time structure without conversion
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: LUMINATE
  role: Provides the plot program that renders stacked bar trajectory plots from OTU
    tables with optional external-event overlays
  repo: https://github.com/tyjo/luminate
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:494d21a99496bacd40bdb4ff06b97ad06f30220ea968b5691cdba16ed5fb7e95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyjo/luminate
  tools:
  - LUMINATE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# stacked-bar-trajectory-visualization

## Summary

Generates per-sequence stacked bar plots of relative-abundance trajectories over time from an OTU table (taxa by samples with sequence IDs and sampling times), optionally overlaying external perturbation events, to visually inspect raw or LUMINATE-denoised longitudinal microbiome data.

## When to use

Use when you have a longitudinal OTU/count table (first row sequence IDs, second row sampling times) and need per-mouse/per-subject stacked bar trajectory plots, optionally with external-event indicators.

## When NOT to use

- You need quantitative model output (parameter matrices A, g, B or predicted trajectories) rather than exploratory visualization — use the train/predict programs instead
- You have already-composed figure assets or a non-LUMINATE-format table lacking the required row-1/row-2 sequence-ID and time structure without conversion

## Inputs

- OTU table CSV (taxa by samples; first row = longitudinal sequence IDs, second row = sample times, first column = taxon names), e.g. datasets/bucci2016mdsine/cdiff-counts.csv or cdiff-counts-est.csv
- Optional external-perturbation events CSV with columns sequenceID,eventID,startDay,endDay,magnitude (optional)

## Outputs

- One stacked bar plot PNG per sequence showing relative abundances over time (top 19 taxa uniquely colored, remainder aggregated), with external-event indicators when an events file is supplied

## How to apply

Run LUMINATE's plot program on an OTU table CSV (taxa by samples, sequence IDs in row 1, sampling times in row 2), supplying the events CSV via -e so external perturbations (e.g. introduction of C. diff in the bucci2016mdsine dataset) are drawn as indicators alongside the trajectories. Supply -i (input dir) and -o (output dir) flags as needed. The plot assigns the top 19 taxa a unique color each and aggregates the remaining taxa into a single component, producing one stacked bar plot per longitudinal sequence. To validate denoising, plot the estimate output (e.g. cdiff-counts-est.csv) and confirm the estimated trajectories are visibly smoother than the raw noisy counts, mirroring the paper's smoothed-trajectory figure for the 5 gnotobiotic mice sampled over ~56 days.

## Related tools

- **LUMINATE** (Provides the plot program that renders stacked bar trajectory plots from OTU tables with optional external-event overlays) — https://github.com/tyjo/luminate

## Examples

```
python main.py plot datasets/bucci2016mdsine/cdiff-counts-est.csv -e datasets/bucci2016mdsine/cdiff-events.csv -i datasets/bucci2016mdsine -o datasets/bucci2016mdsine
```

## Evaluation signals

- One stacked bar plot is produced per longitudinal sequence in the OTU table
- Exactly the top 19 taxa receive unique colors and all remaining taxa are aggregated into a single component
- External events from the events CSV appear as indicators on the plots (e.g. C. diff introduction marked per sequence)
- When applied to cdiff-counts-est.csv, plotted trajectories are visibly smoother than plots of the raw cdiff-counts.csv, reproducing the paper's real-data smoothing figure
- Plot output files are written to the directory given by -o (e.g. datasets/bucci2016mdsine)

## Limitations

- Ultra-sparse taxa (>90% missing/nonzero entries) carry little trajectory information; filtering them out as pre-processing makes plots more informative
- Plotting alone does not denoise data — smoothing seen in estimated-trajectory plots requires first running estimate to produce cdiff-counts-est.csv

## Evidence

- [readme] README: plot command and styling: "This will produce one stacked bar plot per sequence of relative abundances over time, along with indicators for external events. The top 19 taxa each receive a unique color"
- [readme] README: data sparsity caveat: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
- [abstract] Article: real-data smoothing finding: "We conclude by demonstrating the utility of LUMINATE on a real dataset, showing that LUMINATE smooths trajectories observed from noisy data."
- [methods] Article: input dataset description: "a dataset of 5 gnotobiotic mice colonized with a bacterial mixture of 16 species (the C. diff dataset)"
