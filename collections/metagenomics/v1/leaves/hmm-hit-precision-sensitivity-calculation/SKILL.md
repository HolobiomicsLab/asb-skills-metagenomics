---
name: hmm-hit-precision-sensitivity-calculation
description: Use when KEMET --hmm_mode onebm HMM hits on MAGs must be validated against
  the same HMM search on corresponding NCBI/PATRIC complete isolate genomes, and precision/sensitivity
  of recovered missing KOs is needed.
when_to_use_negative:
- Only KEGG Module completeness evaluation is needed without HMM-based annotation
  expansion (use --skip_hmm instead)
- No matched complete reference isolate genome exists for the MAG, so no pair-wise
  concordance can be computed
- Input KO annotations are not in a supported KEGG annotation format (eggnog / kaas
  / kofamkoala), which KEMET requires for the MODULES recap and HMM workflow
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: KEMET
  role: Performs the profile HMM search for missing KOs on both MAGs and complete
    genomes (--hmm_mode onebm) whose hit sets are compared
  repo: https://github.com/Matteopaluh/KEMET
- name: HMMER
  role: Underlying HMM search engine whose soft errors are silenced with -q during
    the onebm searches
- name: eggNOG-Mapper
  role: Produces the .emapper.annotations KO annotations supplied to KEMET via -a
    eggnog
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:eabc955383a30e74a1c816a1d3832bad8aeccb1924e7b1629a2f87fe7bad4799
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
  - HMMER
  - KEMET
  - eggNOG-Mapper
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hmm-hit-precision-sensitivity-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies concordance between KEGG ortholog (KO) hits recovered by KEMET profile HMM searches on MAGs versus the identical searches on matched complete reference isolate genomes, yielding per-pair common/MAG-only/complete-genome-only splits and derived precision and sensitivity (e.g. 84.76% shared, 7.62% each discordant class, 91.75% precision/sensitivity).

## When to use

Use when KEMET --hmm_mode onebm HMM hits on MAGs must be validated against the same HMM search on corresponding NCBI/PATRIC complete isolate genomes, and precision/sensitivity of recovered missing KOs is needed.

## When NOT to use

- Only KEGG Module completeness evaluation is needed without HMM-based annotation expansion (use --skip_hmm instead)
- No matched complete reference isolate genome exists for the MAG, so no pair-wise concordance can be computed
- Input KO annotations are not in a supported KEGG annotation format (eggnog / kaas / kofamkoala), which KEMET requires for the MODULES recap and HMM workflow

## Inputs

- MAG FASTA files (12 in the reference study)
- Corresponding complete reference isolate genomes (NCBI or PATRIC, e.g. via IGGdb pointers)
- eggNOG-format KO annotation files (.emapper.annotations) for each genome
- KEMET HMM search outputs for missing KOs (--hmm_mode onebm) from each genome pair

## Outputs

- Per-pair KO hit classification: common, MAG-only, complete-genome-only
- Aggregated concordance fractions (e.g. 84.76% shared, 7.62% complete-genome-only, 7.62% MAG-only)
- Derived precision and sensitivity metrics (~91.75%) written to hmm_concordance_metrics.csv

## How to apply

Run kemet.py on each MAG with `-a eggnog` and `--hmm_mode onebm --skip_gsmm`, keeping the default legit-hit quality threshold of 0.43 and using `-q` to silence MAFFT and HMMER soft errors. Repeat the identical onebm run on each matched complete reference genome (recovered from NCBI or PATRIC as pointed by IGGdb), whose genes are annotated with eggNOG-mapper v2 producing `.emapper.annotations` files. For each genome pair, set-compare the KOs recovered by the HMM search, classifying each hit as common, MAG-only, or complete-genome-only. Aggregate counts across all genome pairs (12 in the reference study), compute the three concordance fractions, and derive precision = sensitivity = shared/(shared + MAG-only) ≈ 84.76/(84.76+7.62) ≈ 91.75%. Write the summary metrics to a CSV (hmm_concordance_metrics.csv) so the fractions can be checked against expected values.

## Related tools

- **KEMET** (Performs the profile HMM search for missing KOs on both MAGs and complete genomes (--hmm_mode onebm) whose hit sets are compared) — https://github.com/Matteopaluh/KEMET
- **HMMER** (Underlying HMM search engine whose soft errors are silenced with -q during the onebm searches)
- **eggNOG-Mapper** (Produces the .emapper.annotations KO annotations supplied to KEMET via -a eggnog)

## Examples

```
python kemet.py MAG1.fasta -a eggnog --hmm_mode onebm --skip_gsmm -q
```

## Evaluation signals

- Aggregated hit-concordance fractions match the expected 84.76% common, 7.62% complete-genome-only, and 7.62% MAG-only
- Derived precision and sensitivity both equal 84.76/(84.76+7.62) ≈ 91.75%
- Every HMM-recovered KO in each genome pair is assigned to exactly one class (common, MAG-only, complete-genome-only) so classes sum to the total hit count
- The legit-hit quality threshold defaults to 0.43 unless --threshold_value was explicitly set, so filtering is consistent across all paired runs
- hmm_concordance_metrics.csv exists and contains one precision/sensitivity row per aggregation run

## Limitations

- HMM recovery is imperfect: KO mock-removal tests achieved only 55% (20/36) true positive rate for MAGs and 67% (8/12) for complete genomes
- Even partial gene sequence truncations (30% or 70% of original length) can affect recovery of unannotated genes
- MAFFT and HMMER soft errors may clutter output unless suppressed with -q
- Concordance depends on MAG quality; reference screening kept only MAGs with ≥90% completeness and ≤5% contamination

## Evidence

- [other] concordance fractions from the source workflow: [evidence span withheld — anchor status: not_found]
