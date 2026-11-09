---
name: sensitivity-specificity-threshold-optimization
description: Use when a heuristic HMM score filter (e.g. KEMET --threshold_value)
  must be calibrated so candidate missing-KO calls agree with an independently verified
  reference set such as translated BLASTp hits against NCBI nr.
when_to_use_negative:
- Input hits are already experimentally or manually verified — no threshold calibration
  against ground truth is needed
- The annotation tool used does not expose a tunable score filter (e.g. eggNOG-mapper
  or KAAS KO annotation used only for KEGG MODULES recap tables)
- The task is KEGG Module completeness evaluation only, run with --skip_hmm, since
  no HMM hit filtering occurs
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0622
tools:
- name: KEMET
  role: Performs nhmmer profile HMM searches for missing KOs and exposes --threshold_value
    (default 0.43) as the score-per-length quality filter being tuned
  repo: https://github.com/Matteopaluh/KEMET
- name: Python
  role: Implementation language of the KEMET pipeline used to sweep threshold values
    and compute per-dataset hit-retention fractions
  repo: https://github.com/Matteopaluh/KEMET
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:fe8fc509eb02166e79c750c8c3ed6d8a560226d82867afa18b06e37297cfe116
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
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sensitivity-specificity-threshold-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Empirical tuning of a profile-HMM hit quality threshold (KEMET's nhmmer score-per-length --threshold_value, default 0.43) against a manually curated BLASTp/NCBI nr ground truth, to maximize recovery of genuine missing KEGG orthologs while minimizing false positives.

## When to use

Use when a heuristic HMM score filter (e.g. KEMET --threshold_value) must be calibrated so candidate missing-KO calls agree with an independently verified reference set such as translated BLASTp hits against NCBI nr.

## When NOT to use

- Input hits are already experimentally or manually verified — no threshold calibration against ground truth is needed
- The annotation tool used does not expose a tunable score filter (e.g. eggNOG-mapper or KAAS KO annotation used only for KEGG MODULES recap tables)
- The task is KEGG Module completeness evaluation only, run with --skip_hmm, since no HMM hit filtering occurs

## Inputs

- Protein FASTA files of the MAG/genome datasets to be searched
- Candidate missing-KO gene hits from KEMET nhmmer (--hmm_mode onebm) runs at threshold 0.43
- Translated BLASTp results of candidate hits against the NCBI nr database
- Manually curated labels of which BLASTp hits are genuine KEGG orthologs

## Outputs

- Tuned --threshold_value for nhmmer hits
- Per-dataset fraction of verified hits retained at the tuned threshold (expected 4.6–7.5%)
- threshold_tuning_results.csv with retention values and chosen threshold per dataset

## How to apply

Run KEMET with nhmmer profile HMM searches (--hmm_mode onebm) on the proteins of each input dataset starting from the default score-per-length threshold of 0.43, then validate every candidate missing-KO hit by translated BLASTp against NCBI nr with manual curation of which hits represent genuine orthologs. Sweep alternative --threshold_value settings and, at each setting, compute the fraction of manually verified ground-truth hits retained by the HMM filter. Select the threshold that identifies the most hits carrying correct KEGG ortholog descriptors while keeping false positives fewest, and confirm the retained fraction falls in the reported 4.6–7.5% range across the two MAG datasets. Record the per-dataset retention values and chosen threshold in a results table (e.g. threshold_tuning_results.csv).

## Related tools

- **KEMET** (Performs nhmmer profile HMM searches for missing KOs and exposes --threshold_value (default 0.43) as the score-per-length quality filter being tuned) — https://github.com/Matteopaluh/KEMET
- **Python** (Implementation language of the KEMET pipeline used to sweep threshold values and compute per-dataset hit-retention fractions) — https://github.com/Matteopaluh/KEMET

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --threshold_value 0.43 --log
```

## Evaluation signals

- The chosen threshold retains between 4.6% and 7.5% of verified hits on each MAG dataset, matching the values reported in the paper's results section
- At the tuned threshold, hits with correct KEGG ortholog descriptors are maximized relative to other threshold settings
- False-positive count at the tuned threshold is the fewest across the swept threshold sweep on both datasets
- threshold_tuning_results.csv contains a retention value and threshold for each of the two MAG datasets
- Baseline KEMET runs reproduce candidate hits at the documented default threshold of 0.43 before sweeping

## Limitations

- The tuned threshold and 4.6–7.5% retention range are dataset-dependent, so transfer to new MAG sets requires repeating the BLASTp/NCBI nr curation
- Ground-truth construction relies on manual curation of BLASTp hits, introducing curator subjectivity
- The paper provides no discussion-section content (no changelog found), so no additional limitations or reproducibility caveats were documented

## Evidence

- [results] Retention range and false-positive rationale from source finding: [evidence span withheld — anchor status: not_found]
