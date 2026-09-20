---
name: biological-versus-technical-zero-classification
description: Use when longitudinal sequencing count data (OTU table, taxa by samples
  over time) contain many observed zeros and you need posterior probabilities distinguishing
  biological zeros from technical/sampling zeros.
when_to_use_negative:
- 'Taxa are ultra-sparse (>90% missing/nonzero entries): the README recommends filtering
  such taxa out as a pre-processing step because there is little information in their
  trajectories over time.'
- Data are not raw longitudinal sequencing counts (e.g. already composited, imputed,
  or pseudo-count-transformed relative abundances), since zero classification relies
  on observing sampling zeros in count data.
- Single cross-sectional samples without longitudinal structure, since the state-space
  model pools information across longitudinal samples to classify zeros.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: LUMINATE
  role: Provides the `estimate` program whose variational state-space inference produces
    posterior probabilities of sampling zeros and estimated relative abundances from
    longitudinal count data
  repo: https://github.com/tyjo/luminate
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:f42def595ab1c7635ee390ce4169f4437dfc96301314e2edb025fdf20faf222b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyjo/luminate
  tools:
  - LUMINATE
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# biological-versus-technical-zero-classification

## Summary

Classify observed zeros in longitudinal 16S count data as biological zeros (taxon absent from the community) versus technical/sampling zeros (taxon below the detection threshold) using LUMINATE's variational state-space inference, which also yields denoised relative-abundance trajectories.

## When to use

Use when longitudinal sequencing count data (OTU table, taxa by samples over time) contain many observed zeros and you need posterior probabilities distinguishing biological zeros from technical/sampling zeros.

## When NOT to use

- Taxa are ultra-sparse (>90% missing/nonzero entries): the README recommends filtering such taxa out as a pre-processing step because there is little information in their trajectories over time.
- Data are not raw longitudinal sequencing counts (e.g. already composited, imputed, or pseudo-count-transformed relative abundances), since zero classification relies on observing sampling zeros in count data.
- Single cross-sectional samples without longitudinal structure, since the state-space model pools information across longitudinal samples to classify zeros.

## Inputs

- OTU table CSV (taxa by samples; row 1 = longitudinal sequence IDs, row 2 = sample times, column 1 = taxon names), e.g. datasets/bucci2016mdsine/cdiff-counts.csv from tyjo/luminate
- Optional external-perturbation events CSV with columns sequenceID,eventID,startDay,endDay,magnitude(optional), e.g. datasets/bucci2016mdsine/cdiff-events.csv

## Outputs

- cdiff-counts-nonzero-posterior-prob.csv: posterior probabilities that each observed zero is a sampling zero (taxon below the detection threshold) as opposed to absent from the community
- cdiff-counts-est.csv: estimated (denoised) relative abundances
- P.pkl: temporary result used for downstream analysis (train/predict)

## How to apply

Run LUMINATE's `estimate` program (`python main.py estimate`) on an OTU table whose first row gives sequence IDs and second row gives sample times, optionally passing `-e` with an events CSV (sequenceID,eventID,startDay,endDay,magnitude) for external perturbations. The program runs variational state-space inference via block coordinate ascent over q(x), forward-backward HMM updates for q(w), and conjugate gradient updates for h. Among its three outputs, `cdiff-counts-nonzero-posterior-prob.csv` gives, for each observed zero, the posterior probability that it is a sampling zero (taxon below detection threshold) rather than an absence. As pre-processing, filter out ultra-sparse taxa with >90% missing (nonzero) entries because their trajectories carry little information. In the paper, the zero-classification model was trained across 10 datasets of 10 longitudinal samples each, computing for each observed zero the posterior probability that it was a biological zero.

## Related tools

- **LUMINATE** (Provides the `estimate` program whose variational state-space inference produces posterior probabilities of sampling zeros and estimated relative abundances from longitudinal count data) — https://github.com/tyjo/luminate

## Examples

```
python main.py estimate datasets/bucci2016mdsine/cdiff-counts.csv -e datasets/bucci2016mdsine/cdiff-events.csv -o datasets/bucci2016mdsine
```

## Evaluation signals

- The `estimate` run produces all three expected files: cdiff-counts-est.csv, cdiff-counts-nonzero-posterior-prob.csv, and P.pkl in the specified output directory.
- Every observed zero in the input table has a corresponding posterior probability in [0,1] in cdiff-counts-nonzero-posterior-prob.csv, quantifying the chance it is a sampling zero rather than a biological zero.
- Estimated trajectories (cdiff-counts-est.csv) are visibly smoother than the raw noisy counts across all sequences, reproducing the paper's real-data smoothing figure.
- Ultra-sparse taxa (>90% missing entries) were removed before inference, as the README recommends as a pre-processing step.
- For the C. diff application, inference runs over the 5 gnotobiotic mice (average ~26 observed time points over 56 days) with the C. diff introduction treated as an external event via the -e flag.

## Limitations

- For ultra-sparse taxa (>90% missing entries) there is little information in the trajectory over time; the README recommends filtering them out as pre-processing.
- The biological zero detection framework does not yet include external perturbations such as antibiotics; the discussion names this as future work.
- Training cLV model parameters with LUMINATE is considered experimental per the README.

## Evidence

- [abstract] We further show that LUMINATE can accurately distinguish biological zeros, when a taxon is absent from the community, from technical zeros, when a taxon is below the detection threshold.: "We further show that LUMINATE can accurately distinguish biological zeros, when a taxon is absent from the community, from technical zeros, when a taxon is below the detection threshold."
- [methods] For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step.: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
- [methods] We trained our model across 10 datasets of 10 longitudinal samples each, and for each observed zero computed the posterior probability that it was a biological zero: [evidence span withheld — anchor status: not_found]
- [readme] cdiff-counts-nonzero-posterior-prob.csv: posterior probabilities of sampling zeros (i.e. a taxon is below the detection threshold as opposed to absent from the community): "cdiff-counts-nonzero-posterior-prob.csv: posterior probabilities of sampling zeros (i.e. a taxon is below the detection threshold as opposed to absent from the community)"
- [discussion] The biological zero detection framework can be extended to include external perturbations, such as antibiotics, to assess how external factors affect risk of colonization by pathogenic bacteria.: [evidence span withheld — anchor status: not_found]
