---
name: longitudinal-microbiome-data-simulation
description: Use when evaluating longitudinal differential-abundance methods on data
  with known true positives — simulate count datasets across sample sizes, effect
  sizes and covariate counts to compute accuracy, TPR and FDR.
when_to_use_negative:
- You need to analyze a real longitudinal dataset for biological discovery rather
  than benchmark tools on data with known ground truth
- Your data are non-longitudinal (cross-sectional) — microbiomeDASim-simulated temporal
  structures and within-individual time shuffling do not apply
- Only a single method needs a quick sanity check without FDR/TPR ground truth, where
  simulation overhead outweighs benefit
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: LongDat
  role: Reference longitudinal-analysis method run on every simulated replicate via
    longdat_cont() with continuous time as test_var; validated for near-zero FDR
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparison method run on identical simulations in negative binomial (NEGBIN)
    mode; expected FDR escalation with >4 covariates
- name: R
  role: Environment for running simulation, LongDat pipeline and metric computation
- name: ANCOM
  role: Competitor method benchmarked for runtime and memory (observed ~350 GB memory)
    on multi-covariate simulated data
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: lgpr
  role: Competitor method; runtime bottleneck (~65 h with 150 samples and 1 covariate,
    96 h cluster timeouts)
- name: ZIBR
  role: Competitor method benchmarked for runtime (~55 h) on multi-covariate simulated
    data
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:54c5acc4828aad72a859e249a58991320235f0b5906bed6cbcc58ab8d3f2b2a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - ANCOM
  - LongDat
  - MaAsLin2
  - R
  - ZIBR
  - lgpr
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# longitudinal-microbiome-data-simulation

## Summary

Generate simulated longitudinal microbiome count datasets (e.g. with microbiomeDASim, plus SparseDOSSA2 for covariate-correlated settings) with known ground-truth features, to benchmark longitudinal differential-abundance tools such as LongDat and MaAsLin2 on accuracy, TPR, FDR, runtime and memory.

## When to use

Use when evaluating longitudinal differential-abundance methods on data with known true positives — simulate count datasets across sample sizes, effect sizes and covariate counts to compute accuracy, TPR and FDR.

## When NOT to use

- You need to analyze a real longitudinal dataset for biological discovery rather than benchmark tools on data with known ground truth
- Your data are non-longitudinal (cross-sectional) — microbiomeDASim-simulated temporal structures and within-individual time shuffling do not apply
- Only a single method needs a quick sanity check without FDR/TPR ground truth, where simulation overhead outweighs benefit

## Inputs

- Simulation parameters: sample sizes (10-300), effect sizes, number of covariates (1, 2, 4, 8, 16), time-covariate correlation rho (0.25-0.99), replicates per setting (50-100)
- microbiomeDASim simulation settings (e.g. 100 features, count data, no covariates or time-correlated covariates)
- SparseDOSSA2 simulation settings for covariate-correlated longitudinal count data
- Genus-level fasting stool microbiome longitudinal master table (for semi-synthetic shuffled-time nulls)

## Outputs

- Per-replicate metrics table: accuracy, true positive rate, false discovery rate per method per setting
- simulation_benchmark_results.csv / benchmark_results.csv aggregating metrics across replicates
- Runtime and peak-memory summary per tool, sample size and covariate setting (cf. Supplementary Table S1, Figs S10-S11)
- False-positive counts per tool per normalization method on shuffled-time nulls (cf. Supplementary Fig. S16)
- Figures reproducing Fig. 4 and Supplementary Figs S3-S8 (FDR/TPR trajectories vs covariate count)

## How to apply

Simulate longitudinal microbiome count data with microbiomeDASim, sweeping sample sizes (10–300), effect sizes (e.g. two levels) and numbers of dummy covariates correlated with time (1, 2, 4, 8, 16; rho 0.25–0.99), generating 100 (or 50 for heavy competitor tools) replicate simulations per parameter setting with no covariates or covariate structure as required. For semi-synthetic nulls, permute the time variable within each individual to erase true time–feature associations while preserving individual-level structure. Run each method (longdat_cont() with data_type='count' and a seed set for reproducible randomized control tests, MaAsLin2 in NEGBIN mode, ANCOM, lgpr, ZIBR) on identical simulated replicates, computing accuracy, FDR, TPR, wallclock runtime and peak memory per replicate. Aggregate metrics across replicates per setting into a results CSV, and flag competitor runs exceeding the 96 h cluster timeout. Accept a benchmark when LongDat's FDR median stays at zero (accuracy 0.9–1) and covariate-effect success rate median reaches ~0.95, while expected competitor bottlenecks appear (ANCOM ~350 GB memory, ZIBR ~55 h, lgpr ~65 h).

## Related tools

- **LongDat** (Reference longitudinal-analysis method run on every simulated replicate via longdat_cont() with continuous time as test_var; validated for near-zero FDR) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparison method run on identical simulations in negative binomial (NEGBIN) mode; expected FDR escalation with >4 covariates)
- **R** (Environment for running simulation, LongDat pipeline and metric computation)
- **ANCOM** (Competitor method benchmarked for runtime and memory (observed ~350 GB memory) on multi-covariate simulated data) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **lgpr** (Competitor method; runtime bottleneck (~65 h with 150 samples and 1 covariate, 96 h cluster timeouts))
- **ZIBR** (Competitor method benchmarked for runtime (~55 h) on multi-covariate simulated data)

## Examples

```
set.seed(42); res <- longdat_cont(input = simulated_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- LongDat accuracy on covariate-free simulations falls in the 0.9-1 range across all tested effect sizes and sample sizes
- LongDat's FDR median remains at zero across all settings, and TPR increases with sample size
- Median covariate-effect success rate for LongDat reaches ~0.95; its FDR stays stable and TPR does not diminish above four covariates
- Competitor bottlenecks reproduce within order of magnitude: ANCOM ~350 GB memory, ZIBR ~55 h runtime, lgpr ~65 h runtime with 96 h timeouts
- On shuffled-time semi-synthetic nulls, LongDat shows near-zero false positives while MaAsLin2, lgpr and ZIBR show measurably higher rates

## Limitations

- lgpr is a severe runtime bottleneck (~65 h with 150 samples and 1 covariate) and ANCOM consumes ~350 GB memory, requiring cluster resources and 96 h timeout flagging
- With data_type='count', longdat_cont() includes a randomized control test, so a seed must be set or results are not reproducible
- Simulations with time-correlated dummy covariates require careful rho specification (0.25-0.99); conclusions about stability beyond the tested 1-16 covariate range are not established

## Evidence

- [abstract] covariate-free simulations comparable accuracy: "We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data."
- [methods] longdat_cont pipeline: "Longitudinal analysis pipeline with longdat_cont()"
- [intro] seed for count reproducibility: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [intro] longdat_cont invocation: "test_cont <- longdat_cont(input = LongDat_cont_master_table,
data_type = "count",
test_var = "Day","
- [results] simulation finding: [evidence span withheld — anchor status: not_found]
- [readme] LongDat introduction: "LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at"
