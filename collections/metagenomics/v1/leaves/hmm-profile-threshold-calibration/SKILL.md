---
name: hmm-profile-threshold-calibration
description: Use when running KEMET's HMM-based missing-KEGG-ortholog search (--hmm_mode
  onebm) and you need to tune or justify the nhmmer score-per-length cutoff against
  verified reference hits rather than accepting the 0.43 default.
when_to_use_negative:
- You only need KEGG Module completeness evaluation and no HMM-based missing-KO search
  (--skip_hmm workflow, where --threshold_value is irrelevant)
- No manually verified ground-truth ortholog calls (e.g. curated BLASTp vs NCBI nr
  results) exist to calibrate the filter against
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0194
tools:
- name: KEMET
  role: Runs nhmmer profile HMM searches for missing KOs; its --threshold_value parameter
    (default 0.43) is the quality filter being calibrated
  repo: https://github.com/Matteopaluh/KEMET
- name: Python
  role: KEMET is an open-source Python library; Python is used to sweep thresholds
    and compute hit-retention fractions into threshold_tuning_results.csv
  repo: https://github.com/Matteopaluh/KEMET
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:9f59517fdae041b4d1cbef79798bdf316555c2f79c4e63ac9fa5898fd5ef4387
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Matteopaluh/KEMET
  repo_url: https://github.com/Matteopaluh/KEMET
  tools:
  - KEMET
  - Python
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hmm-profile-threshold-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates the KEMET nhmmer score-per-length quality filter (--threshold_value, default 0.43) by sweeping candidate thresholds and measuring agreement with manually verified translated BLASTp hits against NCBI nr, so that HMM-based missing-KO recovery is tuned to the ground truth on each input dataset.

## When to use

Use when running KEMET's HMM-based missing-KEGG-ortholog search (--hmm_mode onebm) and you need to tune or justify the nhmmer score-per-length cutoff against verified reference hits rather than accepting the 0.43 default.

## When NOT to use

- You only need KEGG Module completeness evaluation and no HMM-based missing-KO search (--skip_hmm workflow, where --threshold_value is irrelevant)
- No manually verified ground-truth ortholog calls (e.g. curated BLASTp vs NCBI nr results) exist to calibrate the filter against

## Inputs

- MAG protein FASTA files from two MAG datasets
- KEMET nhmmer profile HMM candidate hits for missing KOs (--hmm_mode onebm)
- Translated BLASTp search results against the NCBI nr database with manual curation of correct KEGG ortholog descriptors

## Outputs

- Chosen --threshold_value maximizing agreement with BLASTp-verified ground truth
- Per-dataset fraction of verified hits retained by the HMM filter (expected 4.6–7.5%)
- threshold_tuning_results.csv with per-dataset retention values and the selected threshold

## How to apply

Run KEMET with --hmm_mode onebm on the proteins of both MAG datasets, starting from the default score-per-length threshold of 0.43, and collect candidate missing-KO gene hits. Build a ground truth by validating these candidates with translated BLASTp searches against the NCBI nr database and manually curating which hits carry correct KEGG ortholog descriptors. Sweep alternative --threshold_value settings, computing at each threshold the fraction of manually verified BLASTp-ground-truth hits retained by the HMM filter and the number of false positives accepted. Select the threshold that maximizes agreement with the verified ground truth; the calibrated value identifies the most hits with correct KEGG ortholog descriptors and fewest false positives, retaining 4.6–7.5% of hits depending on the input dataset.

## Related tools

- **KEMET** (Runs nhmmer profile HMM searches for missing KOs; its --threshold_value parameter (default 0.43) is the quality filter being calibrated) — https://github.com/Matteopaluh/KEMET
- **Python** (KEMET is an open-source Python library; Python is used to sweep thresholds and compute hit-retention fractions into threshold_tuning_results.csv) — https://github.com/Matteopaluh/KEMET

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --threshold_value 0.43 --log
```

## Evaluation signals

- The tuned threshold retains between 4.6% and 7.5% of hits on each MAG dataset, matching the values reported in the paper's results section
- At the selected threshold, candidate hits show the most hits with correct KEGG ortholog descriptors and the fewest false positives relative to the BLASTp/NCBI nr ground truth
- Per-dataset retention values and the chosen threshold are written to threshold_tuning_results.csv
- Sweep results are computed on both MAG datasets, not just one, and the default 0.43 threshold is included as the baseline

## Limitations

- Retention fraction depends on the input dataset (4.6–7.5% range), so the calibrated threshold may not transfer to other genome sets
- Calibration depends on manually curated translated BLASTp hits against NCBI nr, which is a labor-intensive ground truth that may itself contain curation errors
- The article's discussion section contains no content ('_No changelog found._'), so limitations of the thresholding approach cannot be extracted beyond the results reported

## Evidence
