---
name: reconstruction-accuracy-delta-computation
description: Use when comparing HGT local-strain reconstruction methods (LEMON vs
  LUMPY) on simulated metagenomic reads at multiple coverage levels to verify that
  graph-based reconstruction outperforms split-read-only methods.
when_to_use_negative:
- Evaluation is on real (non-simulated) metagenomic data where ground-truth donor/receptor
  strains and true breakpoints are unknown, so RA cannot be computed against reference
  segments
- The goal is HGT breakpoint detection accuracy rather than transferred-segment reconstruction
  quality
- Input is a single coverage condition — the delta-RA trend criterion (ΔRA decreasing
  with coverage) requires at least 5X and 30X conditions to compare
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LEMON
  role: Graph-based method that detects HGT breakpoints and reconstructs local strains
    whose transferred segments are scored for RA; expected to yield higher RA than
    LUMPY
  repo: https://github.com/lichen2018/LEMON
- name: LUMPY
  role: Third-party structural-variant tool run with default parameters as the baseline
    whose RA is subtracted from LEMON's RA to form ΔRA
- name: Smith-Waterman algorithm
  role: Local sequence alignment (via ssw-py) with match_score=1, mismatch_score=-1
    used to measure similarity between reconstructed and reference segments, yielding
    SW(hi,Hi) scores for RA computation
  repo: https://github.com/lichen2018/LEMON
- name: WGSIM
  role: Generates simulated paired-end shotgun reads from true local strains at 5X
    and 30X coverage for the evaluation
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:007f3e9aff37478e1e43b89ee8377e9b33968d7a4ec17969612e5419b7cd0db2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - LEMON
  - LUMPY
  - Smith-Waterman algorithm
  - WGSIM
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# reconstruction-accuracy-delta-computation

## Summary

Quantifies how well LEMON reconstructs transferred genome segments versus LUMPY by computing Reconstruction Accuracy (RA) from Smith-Waterman alignment scores and evaluating ΔRA = RA_LEMON − RA_LUMPY across WGSIM-simulated 5X and 30X coverage, reproducing the Fig. 5a trend.

## When to use

Use when comparing HGT local-strain reconstruction methods (LEMON vs LUMPY) on simulated metagenomic reads at multiple coverage levels to verify that graph-based reconstruction outperforms split-read-only methods.

## When NOT to use

- Evaluation is on real (non-simulated) metagenomic data where ground-truth donor/receptor strains and true breakpoints are unknown, so RA cannot be computed against reference segments
- The goal is HGT breakpoint detection accuracy rather than transferred-segment reconstruction quality
- Input is a single coverage condition — the delta-RA trend criterion (ΔRA decreasing with coverage) requires at least 5X and 30X conditions to compare

## Inputs

- Simulated Reconstruction Accuracy (RA) values at 5X and 30X coverage for LEMON and LUMPY from upstream reproduction output files (WGSIM-simulated reads, 4 repetitions)
- Candidate transferred segment sequences (segments of Hi) reconstructed by LEMON
- True/reference segment sequences (hi) and donor/receptor sequences from the metagenomic reference FASTA
- Accurate HGT breakpoints file (e.g. sample.acc.txt) with breakpoint pair positions

## Outputs

- segment_similarity_scores.csv with per-segment Smith-Waterman scores, match flags, and RAi values
- delta_ra_summary.csv with ΔRA per coverage level and pass/fail verdicts for the positivity and trend criteria

## How to apply

Reconstruct local strains with LEMON and LUMPY from WGSIM-simulated paired-end reads (4 repetitions per coverage level, e.g. 5X and 30X). For each strain, align each reconstructed segment sequence Hi against the corresponding true reference segment hi with the Smith-Waterman local alignment algorithm (match_score = 1, mismatch_score = -1), then compute RAi = SW(hi,Hi)/(mi*match_score), where mi is the number of segments in the true strain (RAi = 0 for an empty reconstruction); flag a segment as matched when both reconstructed breakpoint positions lie within 20 bp of the true breakpoint pair. Next compute ΔRA = RA_LEMON − RA_LUMPY per coverage level and apply two criteria: every ΔRA must be positive, and ΔRA at 5X must exceed ΔRA at 30X (ΔRA decreases as coverage increases), matching the paper's reported Fig. 5a finding. Write per-segment alignment scores, match flags, and RA values to segment_similarity_scores.csv, and ΔRA values plus pass/fail verdicts to delta_ra_summary.csv.

## Related tools

- **LEMON** (Graph-based method that detects HGT breakpoints and reconstructs local strains whose transferred segments are scored for RA; expected to yield higher RA than LUMPY) — https://github.com/lichen2018/LEMON
- **LUMPY** (Third-party structural-variant tool run with default parameters as the baseline whose RA is subtracted from LEMON's RA to form ΔRA)
- **Smith-Waterman algorithm** (Local sequence alignment (via ssw-py) with match_score=1, mismatch_score=-1 used to measure similarity between reconstructed and reference segments, yielding SW(hi,Hi) scores for RA computation) — https://github.com/lichen2018/LEMON
- **WGSIM** (Generates simulated paired-end shotgun reads from true local strains at 5X and 30X coverage for the evaluation)

## Examples

```
python LEMON/Scripts/reconstruct_HGT_strain.py -r reference.fasta -c coverage.txt -s sample -a sample.acc.txt -o out/
```

## Evaluation signals

- Every computed ΔRA value is strictly greater than 0 at both 5X and 30X coverage
- ΔRA at 5X exceeds ΔRA at 30X (ΔRA decreases as coverage increases), matching the reported Fig. 5a finding
- Each RAi value falls within [0, 1], with RAi = 1 when all transferred segments are matched (SW equals mi*match_score) and RAi = 0 for empty reconstructions
- Every segment row in segment_similarity_scores.csv contains a finite Smith-Waterman score and a match flag determined by breakpoints within 20 bp of the true pair
- delta_ra_summary.csv records ΔRA per coverage level and explicit pass/fail verdicts for both criteria

## Limitations

- Results are validated only on simulated gut metagenomics data with WGSIM-generated reads; real-data accuracy depends on the availability of gene fusion points near HGT breakpoints
- RA measurement assumes the true strain's segment set (mi) is known and that match_score=1 / mismatch_score=-1 scoring is appropriate; RA is 0 for any empty reconstruction
- Third-party tools (LUMPY, BWA, iRep, STAR-Fusion) are used at default parameters, so ΔRA reflects LEMON's gain under default baseline configuration rather than a tuned baseline

## Evidence

- [methods] Smith-Waterman as RA basis: "We apply Smith-Waterman algorithm to measure the similarity between segment sequences of Hi and hi."
- [abstract] Simulation accuracy finding: "The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome."
- [intro] LEMON pipeline scope: "LEMON(https://github.com/lichen2018/LEMON) takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred segments, and reconstruct the local strain"
- [methods] Default-parameter baselines: "Most third-party tools used in this article are set default parameters, including BWA, LUMPY [25], iRep [26], and STAR-Fusion [27]."
- [methods] WGSIM read simulation: "All true local strains were used to generated 20X paired-end reads with WGSIM [28] as an input of LEMON."
- [readme] Repository install command: "git clone --recursive https://github.com/lichen2018/hgt-detection.git"
