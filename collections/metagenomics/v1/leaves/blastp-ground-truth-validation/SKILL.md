---
name: blastp-ground-truth-validation
description: Use when a profile-HMM screen (e.g., KEMET --hmm_mode onebm) yields candidate
  missing-KO hits that need independent verification against a reference database,
  or when the HMM score threshold must be tuned to balance sensitivity against false
  positives.
when_to_use_negative:
- Input genomes already have complete, trusted KO annotations from eggNOG-mapper/KAAS/KofamKoala
  and no HMM-based recovery of missing KOs is being attempted
- You need KEGG Module completeness tables only, i.e., KEMET run with --skip_hmm,
  where no HMM hits are produced to validate
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0080
tools:
- name: KEMET
  role: Generates candidate missing-KO hits via nhmmer profile HMM searches in --hmm_mode
    onebm; its --threshold_value quality filter (default 0.43) is the parameter being
    tuned against BLASTp ground truth
  repo: https://github.com/Matteopaluh/KEMET
- name: Python
  role: Implements the threshold sweep, hit-retention computation, and writing of
    threshold_tuning_results.csv
  repo: https://github.com/Matteopaluh/KEMET
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:cd8d299d6f42fc827eca96db3988827cc0e4abd9c4b68bb1b662aaaaf143ad12
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# blastp-ground-truth-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates HMM-based missing-KEGG-ortholog candidate hits (e.g., from KEMET's nhmmer searches) by translated BLASTp against NCBI nr with manual curation, then uses the curated set as ground truth to fine-tune the KEMET --threshold_value score-per-length filter (default 0.43).

## When to use

Use when a profile-HMM screen (e.g., KEMET --hmm_mode onebm) yields candidate missing-KO hits that need independent verification against a reference database, or when the HMM score threshold must be tuned to balance sensitivity against false positives.

## When NOT to use

- Input genomes already have complete, trusted KO annotations from eggNOG-mapper/KAAS/KofamKoala and no HMM-based recovery of missing KOs is being attempted
- You need KEGG Module completeness tables only, i.e., KEMET run with --skip_hmm, where no HMM hits are produced to validate

## Inputs

- MAG protein FASTA files (two MAG datasets in the reference workflow)
- KEMET nhmmer candidate missing-KO hits (default --threshold_value 0.43)
- NCBI nr database for translated BLASTp searches
- Manually curated ground-truth labels (correct KEGG ortholog descriptor per hit)

## Outputs

- Manually verified BLASTp ground-truth hit sets per MAG dataset
- Per-dataset fraction of verified hits retained at each swept threshold
- threshold_tuning_results.csv with per-dataset retention values and the chosen threshold

## How to apply

Run KEMET (--hmm_mode onebm) with nhmmer profile HMM searches on the proteins of each input MAG dataset, starting from the default score-per-length threshold of 0.43, to collect candidate missing-KO gene hits. Validate each candidate by translated BLASTp searches against the NCBI nr database and manually curate which hits carry the correct KEGG ortholog descriptor, yielding a verified ground-truth set per dataset. Sweep alternative --threshold_value settings with KEMET on both datasets, computing at each threshold the fraction of manually verified BLASTp-ground-truth hits retained by the HMM filter. Select the threshold that maximizes agreement with the verified ground truth while reproducing the paper's reported 4.6–7.5% hit-retention range across the two MAG datasets, and write per-dataset retention values and the chosen threshold to threshold_tuning_results.csv.

## Related tools

- **KEMET** (Generates candidate missing-KO hits via nhmmer profile HMM searches in --hmm_mode onebm; its --threshold_value quality filter (default 0.43) is the parameter being tuned against BLASTp ground truth) — https://github.com/Matteopaluh/KEMET
- **Python** (Implements the threshold sweep, hit-retention computation, and writing of threshold_tuning_results.csv) — https://github.com/Matteopaluh/KEMET

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --threshold_value 0.43 --log
```

## Evaluation signals

- The chosen threshold retains between 4.6% and 7.5% of hits on each MAG dataset, matching the paper's reported retention range
- The tuned threshold identifies the most hits with correct KEGG ortholog descriptors and the fewest false positives relative to the BLASTp-verified set
- threshold_tuning_results.csv contains one retention value per input dataset plus the selected threshold
- Retention fraction is monotonically related to the swept --threshold_value (higher thresholds retain fewer hits)
- Every retained candidate hit is cross-checkable against its translated BLASTp hit in NCBI nr with the expected KO descriptor

## Limitations

- Ground truth depends on manual curation of BLASTp hits against NCBI nr, so the validated hit set is curator-dependent and bounded by nr coverage
- The 4.6–7.5% retention range and the optimal threshold are dataset-dependent; the threshold is defined as a score-per-length filter that may transfer poorly to very different genome binning qualities

## Evidence

- [other] Fine-tuning against manually checked translated BLASTp hits from NCBI nr on two MAG datasets showed the threshold identifies the most hits with correct KEGG ortholog descriptors and fewest false positives, retaining 4.6–7.5% of hits: [evidence span withheld — anchor status: not_found]
- [readme] threshold_value parameter: "`--threshold_value [VALUE]`: use another quality filter to differentiate between legit HMM hits (default: 0.43)."
- [results] HMM search for missing KOs: "MAGs were also used to search for missing KOs by using kemet.py --hmm_mode onebm"
- [abstract] HMM recovery purpose: "KEMET focuses on the in-depth analysis of metabolic reaction networks to identify missing orthologs through hidden Markov model profiles"
