---
name: hmm-hit-threshold-filtering
description: Use when KEMET HMM-search (--hmm_mode onebm/module/kos) output must be
  filtered so only KOs whose HMM profile score passes the quality cutoff (default
  0.43) count as recovered annotations, e.g. in truncation or KO-removal recovery
  tests.
when_to_use_negative:
- When the goal is only KEGG Module completeness evaluation without HMM searches (use
  --skip_hmm instead)
- When hits are already filtered by another downstream quality control, so applying
  the 0.43 cutoff again would double-filter and underestimate recovery
- When evaluating KEMET's Module partitioning against KEGG Mapper or METABOLIC, which
  does not involve HMM hit scoring
tools:
- name: KEMET
  role: Runs the HMM profile searches and exposes the --threshold_value quality filter
    (default 0.43) that defines legitimate HMM hits
  repo: https://github.com/Matteopaluh/KEMET
- name: eggNOG-Mapper
  role: Provides the initial KO annotations (-a eggnog format) against which HMM-search
    recovery is measured after threshold filtering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:3a48a283919c16fbee0b97271491764fe45448c200996333c5d9b13571f34aad
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
  - eggNOG-Mapper
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# hmm-hit-threshold-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A filtering step from the KEMET workflow that differentiates legitimate HMM hits from spurious profile-HMM matches by applying a quality threshold (default 0.43) to HMMER search scores when recovering missing KEGG ortholog annotations.

## When to use

Use when KEMET HMM-search (--hmm_mode onebm/module/kos) output must be filtered so only KOs whose HMM profile score passes the quality cutoff (default 0.43) count as recovered annotations, e.g. in truncation or KO-removal recovery tests.

## When NOT to use

- When the goal is only KEGG Module completeness evaluation without HMM searches (use --skip_hmm instead)
- When hits are already filtered by another downstream quality control, so applying the 0.43 cutoff again would double-filter and underestimate recovery
- When evaluating KEMET's Module partitioning against KEGG Mapper or METABOLIC, which does not involve HMM hit scoring

## Inputs

- KEMET HMM-search output (HMMER hits for candidate KOs from kemet.py with --hmm_mode onebm)
- eggNOG-mapper (.emapper.annotations) or kaas/kofamkoala KO annotation files
- MAG or complete-genome nucleotide FASTA files

## Outputs

- Set of HMM hits passing the quality threshold (0.43 default), i.e. recovered KO annotations
- Recovery/true-positive rate counts (recovered KOs / total candidate KOs)

## How to apply

Run KEMET in HMM-search mode (e.g. './kemet.py [FASTA_file] -a eggnog --hmm_mode onebm') so tested KOs are searched against the genome/MAG sequences with HMMER profile models. Classify a hit as a legitimate annotation recovery only if its HMM quality score passes the default threshold_value of 0.43; use the '--threshold_value [VALUE]' option only if a stricter or looser filter is deliberately being benchmarked. When counting recovery rates (recovered KOs / total KO candidates, e.g. after 30% or 70% sequence truncation or mock KO removal), tally only threshold-passing hits as true positives — the article's 20-33% recovery at 30% truncation and 55-67% true positive rates assume the default 0.43 cutoff. Document the threshold used, since changing it directly shifts the sensitivity/specificity balance of annotation recovery.

## Related tools

- **KEMET** (Runs the HMM profile searches and exposes the --threshold_value quality filter (default 0.43) that defines legitimate HMM hits) — https://github.com/Matteopaluh/KEMET
- **eggNOG-Mapper** (Provides the initial KO annotations (-a eggnog format) against which HMM-search recovery is measured after threshold filtering)

## Examples

```
./kemet.py [FASTA_file] -a eggnog --hmm_mode onebm --threshold_value 0.43 --skip_gsmm
```

## Evaluation signals

- All reported recovered KOs in the output have an HMM score at or above the stated threshold (default 0.43)
- KO mock-removal recovery falls within reported ranges: 55% (20/36) true positive rate for MAGs and 67% (8/12) for complete genomes
- Truncation recovery rates fall within the reported 20-33% range at 30% truncation and decline monotonically at 70% truncation
- Changing --threshold_value is documented and consistently applied across all datasets in the comparison
- Recovery counts are computed as recovered KOs divided by total truncated/removed KOs per dataset and condition

## Limitations

- The default 0.43 threshold trades sensitivity for specificity; stricter values will lower measured recovery rates while looser values risk spurious KO recoveries
- Recovery strongly depends on gene prediction: 30% truncation yielded 20-33% recovery accounting for truncation's impact on gene calling (42% assuming unbiased gene prediction)
- Recovery degrades with sequence fragmentation, most clearly in MAG datasets with gapped assemblies

## Evidence
