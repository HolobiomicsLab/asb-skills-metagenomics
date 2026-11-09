---
name: detection-curve-visualization
description: Use when you need to determine how many raw MinION reads are required
  before a baseLess abundance-mode classifier's MSRD plateau is reached, e.g. plotting
  MSRD-vs-reads curves for multiple target species on a single figure.
when_to_use_negative:
- You need per-read target detection output — in abundance mode baseLess retains no
  reads, so read-level filtering plots are not applicable.
- You are using read_detection mode on 16S data where the relevant metric is the >66%
  k-mer fraction threshold per read, not an MSRD stability curve.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0092
tools:
- name: baseLess
  role: runs the abundance-mode inference over fast5 reads whose per-read-count MSRD
    values feed the stabilization curves
  repo: https://github.com/cvdelannoy/baseLess
- name: mamba
  role: installs baseLess and its dependencies from bioconda/conda-forge
- name: MinION
  role: produces the raw electrical-signal fast5 readsets on which MSRD stabilization
    is evaluated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:e6310a8831e4fe0eecbac9c4a46693d983f594aa627ad16577f3addbe8818dd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - MinION
  - baseLess
  - conda
  - mamba
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# detection-curve-visualization

## Summary

Visualizes MSRD (model stability metric) as a function of the number of MinION reads processed by baseLess abundance-mode models, to identify the read count at which each species classification stabilizes.

## When to use

Use when you need to determine how many raw MinION reads are required before a baseLess abundance-mode classifier's MSRD plateau is reached, e.g. plotting MSRD-vs-reads curves for multiple target species on a single figure.

## When NOT to use

- You need per-read target detection output — in abundance mode baseLess retains no reads, so read-level filtering plots are not applicable.
- You are using read_detection mode on 16S data where the relevant metric is the >66% k-mer fraction threshold per read, not an MSRD stability curve.

## Inputs

- per-read MSRD values logged during 'baseLess run_inference' for each species' abundance model
- fast5 read sets (e.g. 2000 MinION reads per species)
- compiled abundance-mode .h5 models (one per target species)

## Outputs

- figure plotting MSRD-vs-reads curves for all species with annotated stabilization read counts
- stabilization points (read counts at MSRD plateau, e.g. 52, 352, 84 reads)
- per-species classification correctness confirmation

## How to apply

Run each compiled abundance model (.h5) with 'baseLess run_inference --inference-mode once' on the species' fast5 read set (e.g. 2000 reads), recording the MSRD metric as reads accumulate. Track MSRD per processed read count for each species-specific model, then plot the MSRD-vs-reads curves for all species (e.g. P. gracilis, P. januarius, P. turneri) on a single figure. Annotate the stabilization points — the read counts at which MSRD plateaus, e.g. 52, 352 and 84 reads in the guppy study. Validation requires that each curve becomes visibly stable at the annotated read count and that inference calls each species correctly.

## Related tools

- **baseLess** (runs the abundance-mode inference over fast5 reads whose per-read-count MSRD values feed the stabilization curves) — https://github.com/cvdelannoy/baseLess
- **mamba** (installs baseLess and its dependencies from bioconda/conda-forge)
- **MinION** (produces the raw electrical-signal fast5 readsets on which MSRD stabilization is evaluated)

## Examples

```
baseLess run_inference --fast5-in directory/of/fast5s --out-dir your/output/directory --model your_model_name.h5 --inference-mode once
```

## Evaluation signals

- MSRD curves reach a plateau rather than continuing to drift, indicating the classification has stabilized
- Annotated stabilization read counts reproduce reported values (52, 352 and 84 reads for P. gracilis, P. januarius and P. turneri)
- Each abundance model assigns reads to the correct target species against the other two Poeciliidae genomes
- Stabilization occurs well before the full 2000-read set is consumed, confirming the models do not need all reads

## Limitations

- Stabilization read counts are species- and model-specific; they do not transfer to other genome pairs or k-mer network sets.
- MSRD stabilization requires inference over an ordered read stream; curves from shuffled or partial read sets may misrepresent the plateau point.
- Reads used for on-the-fly k-mer network training must first be basecalled with guppy, split to single fast5, and corrected with tombo resquiggle before model compilation.

## Evidence

- [results] BaseLess consistently called the correct species for each analyzed 2000-read set (Fig. 2A-C), and did not need the full 2000 reads for any classification: [evidence span withheld — anchor status: not_found]
- [results] stable MSRD values attained after 52, 352 and 84 reads: [evidence span withheld — anchor status: not_found]
- [methods] Run 'baseLess run_inference --inference-mode once' with each compiled model on the corresponding 2000-read fast5 set: [evidence span withheld — anchor status: not_found]
- [readme] To run on a directory of reads and stop when all reads are analysed, provide `--inference-mode once`: "To run on a directory of reads and stop when all reads are analysed, provide `--inference-mode once`"
