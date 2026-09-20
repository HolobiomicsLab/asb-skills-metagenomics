---
name: r-statistical-programming
description: Use when implementing or reproducing the AMAT microbiome multivariate
  association pipeline (DC-based feature screening plus adaptive GLM-based testing)
  from simulated or real OTU count matrices with an outcome vector, in R.
when_to_use_negative:
- The analysis goal is single-taxon univariate testing rather than multivariate (clade/set-based)
  association analysis
- Only raw 16S sequences are available and OTU construction/QIIME preprocessing has
  not yet been performed
- The taxa-set of interest contains fewer than five taxa, below the article's threshold
  for association analysis
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0610
tools:
- name: R
  role: Computing environment in which the AMAT feature-selection and association
    test is implemented and run
  repo: https://github.com/kzb193/AMAT
- name: AMAT
  role: R package performing distance-correlation-based feature selection followed
    by the data-adaptive GLM association test
  repo: https://github.com/kzb193/AMAT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:d55ea85e315129f6571a8787620fbe467f68f14ba2d0bafce69086477d00b11c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/kzb193/AMAT
  repo_url: https://github.com/kzb193/AMAT
  tools:
  - AMAT
  - LDM
  - MiHC
  - OMiAT
  - OMiRKAT
  - R
  - aMiSPU
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# r-statistical-programming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Use the R computing environment and AMAT R package to implement distance-correlation-based feature selection and data-adaptive microbiome association testing on OTU count tables, comparing against competing methods (aMiSPU, OMiRKAT, OMiAT, LDM, MiHC) under their default software settings.

## When to use

Use when implementing or reproducing the AMAT microbiome multivariate association pipeline (DC-based feature screening plus adaptive GLM-based testing) from simulated or real OTU count matrices with an outcome vector, in R.

## When NOT to use

- The analysis goal is single-taxon univariate testing rather than multivariate (clade/set-based) association analysis
- Only raw 16S sequences are available and OTU construction/QIIME preprocessing has not yet been performed
- The taxa-set of interest contains fewer than five taxa, below the article's threshold for association analysis

## Inputs

- OTU count matrix (e.g. 856 OTUs x 60 samples throat data, or 557 OTUs x 98 samples gut data)
- Outcome vector (continuous or binary) per sample
- Known simulated signal set of truly associated taxa (for precision/recall benchmarking)

## Outputs

- Per-method P-values from 10 000 permutations (AMAT, aMiSPU, OMiRKAT, OMiAT, LDM, MiHC)
- Ranked taxa list from DC-based feature screening with selected testing subset
- CSV table of method, scenario, precision, and recall (one row per method-scenario combination)
- Power and empirical type I error rate estimates across simulation scenarios

## How to apply

Load simulated or real OTU count matrices (e.g. throat microbiome: 856 OTUs, 60 samples; gut microbiome: 557 OTUs, 98 samples) together with the outcome vector into R, then apply the AMAT distance-correlation (DC) screening step to rank taxa and shrink noise features to zero weight before the data-adaptive generalized-linear-model association test. Compute P-values via permutation, using 10 000 permutations for AMAT and the competing methods aMiSPU, OMiRKAT, OMiAT, LDM and MiHC, each run under the default settings of their corresponding software packages. For feature-selection benchmarking, compute precision (proportion of selected taxa truly associated) and recall (proportion of truly associated taxa selected) against the known simulated signal set for every method-scenario combination. Control the family-wise error rate at 0.05 within each taxonomic rank via Bonferroni correction, and restrict association analysis to taxa-sets with at least five taxa. The R implementation of AMAT is freely available at https://github.com/kzb193/AMAT.

## Related tools

- **R** (Computing environment in which the AMAT feature-selection and association test is implemented and run) — https://github.com/kzb193/AMAT
- **AMAT** (R package performing distance-correlation-based feature selection followed by the data-adaptive GLM association test) — https://github.com/kzb193/AMAT

## Examples

```
library(AMAT)
```

## Evaluation signals

- Output CSV exists with exactly one row per competing method per feature-selection scenario, with precision and recall values all in [0,1]
- P-values computed from 10 000 permutations for all six tests
- Empirical type I error rates are well controlled (near nominal 0.05) across simulation configurations, matching the reported result that 'All tests seem to have well controlled type I error rates'
- Bonferroni correction applied so family-wise error rate is controlled at 0.05 within each taxonomic rank
- AMAT power under Scenario I (n=100, 10-30% signal densities) reaches approximately 44.7%, 49.8% and 52.5%, exceeding runner-up methods (~33.6%-38.3%)

## Limitations

- MiHC tends to be conservative, especially with binary outcomes, so its conservative P-values should not be interpreted as a failure of the R implementation
- Score-statistic-based weighting alternatives still leave small non-zero weights on noise taxa, whose accumulation can deteriorate multivariate test performance for moderate or large numbers of taxa
- Naive aggregation of low-rank taxa loses power when effects have opposite directions and cancel out

## Evidence
