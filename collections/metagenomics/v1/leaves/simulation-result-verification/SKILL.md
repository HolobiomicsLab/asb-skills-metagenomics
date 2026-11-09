---
name: simulation-result-verification
description: Use when a paper claims a quantitative simulation result (e.g., 'all
  ΔRA are positive and ΔRA decreases as coverage increases') and raw or aggregated
  reproduction outputs exist from which the metric can be recomputed and checked.
when_to_use_negative:
- The claim to verify is qualitative or based on real (non-simulated) data — this
  skill is specific to recomputing ΔRA from simulated reproduction outputs.
- Upstream reproduction output files for LEMON and LUMPY RA at both coverage levels
  are unavailable, since ΔRA cannot be recomputed without paired RA values per tool
  per coverage.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: LEMON
  role: Graph-based method under test; its Reconstruction Accuracy values are compared
    against LUMPY's via ΔRA
  repo: https://github.com/lichen2018/LEMON
- name: LUMPY
  role: Third-party baseline tool whose RA values provide the comparison term subtracted
    from LEMON's in ΔRA
- name: WGSIM
  role: Simulator that generated the paired-end reads underlying the simulated reproduction
    outputs being verified
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:83bf1eb271c3decec8a0343a78ae6fbef846f9751248949c1b9791649323034e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - LEMON
  - LUMPY
  - WGSIM
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# simulation-result-verification

## Summary

Quantitatively verifies a published simulation finding by recomputing derived metrics — here, delta Reconstruction Accuracy (ΔRA = RA_LEMON − RA_LUMPY) from simulated gut metagenomics reproduction outputs at 5X and 30X coverage — and checking explicit positivity and trend criteria against the paper's Fig. 5a claim.

## When to use

Use when a paper claims a quantitative simulation result (e.g., 'all ΔRA are positive and ΔRA decreases as coverage increases') and raw or aggregated reproduction outputs exist from which the metric can be recomputed and checked.

## When NOT to use

- The claim to verify is qualitative or based on real (non-simulated) data — this skill is specific to recomputing ΔRA from simulated reproduction outputs.
- Upstream reproduction output files for LEMON and LUMPY RA at both coverage levels are unavailable, since ΔRA cannot be recomputed without paired RA values per tool per coverage.

## Inputs

- Simulated Reconstruction Accuracy (RA) values at 5X and 30X coverage for LEMON and LUMPY from upstream reproduction outputs (WGSIM-simulated reads, 4 repetitions)

## Outputs

- delta_ra_summary.csv containing ΔRA per coverage level (5X, 30X) and pass/fail verdicts for the positivity and trend criteria

## How to apply

Load the simulated Reconstruction Accuracy (RA) values for LEMON and LUMPY from the upstream reproduction output files, which were produced with WGSIM-simulated paired-end reads over 4 repetitions. For each coverage level (5X and 30X), compute ΔRA = RA_LEMON − RA_LUMPY as defined for Fig. 5a of the LEMON paper. Apply two explicit criteria: (1) positivity — every ΔRA must be > 0, i.e. LEMON outperforms LUMPY at both 5X and 30X; (2) trend — ΔRA at 5X must exceed ΔRA at 30X, confirming ΔRA decreases as sequencing coverage increases. Write the per-coverage ΔRA values and pass/fail verdicts for both criteria to delta_ra_summary.csv. The verification succeeds only if all ΔRA values are positive and ΔRA decreases with increasing coverage, matching the reported finding.

## Related tools

- **LEMON** (Graph-based method under test; its Reconstruction Accuracy values are compared against LUMPY's via ΔRA) — https://github.com/lichen2018/LEMON
- **LUMPY** (Third-party baseline tool whose RA values provide the comparison term subtracted from LEMON's in ΔRA)
- **WGSIM** (Simulator that generated the paired-end reads underlying the simulated reproduction outputs being verified)

## Examples

```
python verify_delta_ra.py --ra lemon_ra_5x.csv,lumpy_ra_5x.csv,lemon_ra_30x.csv,lumpy_ra_30x.csv --coverage 5,30 --out delta_ra_summary.csv
```

## Evaluation signals

- Every computed ΔRA value (at 5X and 30X, across the 4 repetitions) is strictly greater than 0
- ΔRA at 5X > ΔRA at 30X, reproducing the reported decreasing trend with increasing coverage
- delta_ra_summary.csv is produced and contains both the ΔRA values per coverage level and the pass/fail verdicts for both criteria
- Recomputed RA values match those loaded from the upstream reproduction output files produced with WGSIM-simulated reads

## Limitations

- The verification covers only the two simulated coverage levels (5X and 30X) and the LEMON-vs-LUMPY comparison; it does not validate LEMON on real metagenomic data such as the HMP IBD dataset.
- RA is evaluated on reference sequences of the microbiome in simulation, so accuracy conclusions may not transfer to real, fragmented references.

## Evidence

- [abstract] The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome.: "The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome."
- [methods] WGSIM simulation with default third-party tools: "All true local strains were used to generated 20X paired-end reads with WGSIM [28] as an input of LEMON."
- [methods] LUMPY as comparison tool with default parameters: "Most third-party tools used in this article are set default parameters, including BWA, LUMPY [25], iRep [26], and STAR-Fusion [27]."
- [results] ΔRA positivity and decreasing trend from Fig. 5a finding: "all ΔRA are positive and ΔRA decreases as coverage increases"
