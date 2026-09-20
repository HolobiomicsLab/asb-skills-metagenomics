---
name: supplementary-figure-reproduction-from-simulation
description: Use when a paper reports a simulation-based supplementary figure (negative-control
  FPR comparison) and the goal is to regenerate the simulated count tables, rerun
  both analysis modes, and confirm the reported qualitative pattern.
when_to_use_negative:
- The data contain real treatment effects rather than negative controls with zero
  effect size — FPR is undefined without known-null features.
- The target figure is based on real cohort data rather than simulated longitudinal
  count tables from SparseDOSSA2.
- Only one normalization/model setting is of interest — the skill's validation depends
  on the FPR contrast between rarefied negative-binomial and linear/TSS settings.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: SparseDOSSA2
  role: Simulates negative-control longitudinal microbiome count tables with zero
    effect size and systematically varying sequencing depth
- name: LongDat
  role: Runs the count-data longitudinal pipeline (longdat_cont()/longdat_disc(),
    negative-binomial mode) on rarefied negative-control tables
  repo: https://github.com/CCY-dev/LongDat
- name: longdat_cont()
  role: Longitudinal analysis pipeline function applied to rarefied count tables with
    seed set for reproducible randomized control test
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: Implementation environment for rarefaction, TSS normalization, linear-model
    comparator fits, FPR computation, and figure rendering
  repo: https://github.com/CCY-dev/LongDat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:cc3b8016001a39449b8ee0a16d8f3187085d5d670f116909b773297689c9c16b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - R
  - SparseDOSSA2
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# supplementary-figure-reproduction-from-simulation

## Summary

Reproduce a supplementary figure (e.g. Supplementary Fig. S9-style negative-control sequencing-depth bias comparison) by regenerating simulated negative-control data with SparseDOSSA2, running the competing pipelines (rarefied negative-binomial via LongDat vs. TSS-normalized linear models), and tabulating the false positive rate contrast.

## When to use

Use when a paper reports a simulation-based supplementary figure (negative-control FPR comparison) and the goal is to regenerate the simulated count tables, rerun both analysis modes, and confirm the reported qualitative pattern.

## When NOT to use

- The data contain real treatment effects rather than negative controls with zero effect size — FPR is undefined without known-null features.
- The target figure is based on real cohort data rather than simulated longitudinal count tables from SparseDOSSA2.
- Only one normalization/model setting is of interest — the skill's validation depends on the FPR contrast between rarefied negative-binomial and linear/TSS settings.

## Inputs

- SparseDOSSA2-simulated negative-control longitudinal count tables (zero effect size, two time points, six sequencing-depth versions)
- Rarefied count tables (common library size per version)
- Non-rarefied count tables for TSS normalization
- Nominal alpha significance threshold

## Outputs

- Per-setting false positive rates (fraction of truly-null features called significant) for rarefied negative-binomial and non-rarefied linear/TSS settings
- Tabulated FPR contrast between the two normalization/model settings
- Supplementary Fig. S9-style comparison figure

## How to apply

Generate negative-control longitudinal count tables with SparseDOSSA2 using zero effect size across two time points, producing six dataset versions that differ systematically in sequencing depth. Rarefy each simulated count table to its designated depth so all samples within a version share a common library size, then run LongDat's count-data longitudinal pipeline (longdat_cont() or longdat_disc() in negative-binomial mode, with the seed set for reproducibility as required when data_type equals 'count'). Fit linear models on TSS-transformed versions of the non-rarefied count tables as the compositional-normalization comparator. Compute the false positive rate per setting as the fraction of truly-null features called significant at the nominal alpha threshold across all six depth versions, then tabulate the FPR contrast and render the Supplementary Fig. S9-style comparison figure.

## Related tools

- **SparseDOSSA2** (Simulates negative-control longitudinal microbiome count tables with zero effect size and systematically varying sequencing depth)
- **LongDat** (Runs the count-data longitudinal pipeline (longdat_cont()/longdat_disc(), negative-binomial mode) on rarefied negative-control tables) — https://github.com/CCY-dev/LongDat
- **longdat_cont()** (Longitudinal analysis pipeline function applied to rarefied count tables with seed set for reproducible randomized control test) — https://github.com/CCY-dev/LongDat
- **R** (Implementation environment for rarefaction, TSS normalization, linear-model comparator fits, FPR computation, and figure rendering) — https://github.com/CCY-dev/LongDat

## Examples

```
set.seed(1); res <- longdat_cont(input = master_table, data_type = "count", test_var = "Day", subject_ind = "Subject", time_ind = "Day")
```

## Evaluation signals

- Rarefied negative-binomial FPR is near zero (at or below the nominal alpha level) across all six sequencing-depth versions.
- Linear/TSS FPR on non-rarefied tables is markedly elevated under large sequencing-depth variation.
- LongDat count-data runs are reproducible: seed was set before running longdat_cont()/longdat_disc() so the randomized control test repeats identically.
- Each rarefied dataset version has a common library size across all samples within that version.
- Rendered figure qualitatively matches the Supplementary Fig. S9 pattern (near-zero vs. elevated FPR contrast).

## Limitations

- The validation is qualitative: acceptance depends on the rarefied negative-binomial FPR being at or below nominal alpha and the linear/TSS FPR being markedly elevated, which requires sufficiently large sequencing-depth variation to manifest.
- LongDat's count-data mode requires setting a seed because it performs a randomized control test; without it results are not reproducible.
- The negative-control conclusion applies to count data with zero true effect; it does not directly inform power or FDR on data with real longitudinal effects.

## Evidence

- [supplementary] Negative-control rarefaction vs TSS benchmark: [evidence span withheld — anchor status: not_found]
- [supplementary] TSS comparator failure: [evidence span withheld — anchor status: not_found]
- [intro] Seed requirement: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [intro] Pipeline invocation: "test_cont <- longdat_cont(input = LongDat_cont_master_table,
data_type = "count",
test_var = "Day","
