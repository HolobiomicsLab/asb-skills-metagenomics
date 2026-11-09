---
name: gene-absence-confidence-statistics
description: Use when a presence/absence analysis calls a gene absent in lineages
  represented mainly by draft bins — especially when completeness estimates (e.g.
  ≥75% single-copy genes, ≤1.125 copies) are available and you must decide if the
  absence call is trustworthy.
when_to_use_negative:
- The genome set consists only of complete genomes, so incomplete-sampling artifacts
  cannot explain absences
- Gene detection relies on keyword-based annotation alone rather than HMM profiles
  — divergent homologs would inflate apparent absences before any probability calculation
  is meaningful
- The genes of interest are not single-copy or cannot be enumerated (n undefined),
  making the (1 − c)^n computation inapplicable
tools:
- name: HMMER hmmscan
  role: Inventories ribosomal proteins against Pfam HMM profiles so absences are not
    artifacts of divergent sequences evading keyword annotation
- name: gg Kbase
  role: Supplies keyword-based ribosomal protein gene lists to cross-check the HMM-based
    inventory feeding the presence/absence matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:2b7808b7fb47692b3404e41b2d76442e3c5791725f0ed34e85129681cdc1ccfa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HMMER hmmscan
  - gg Kbase
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-absence-confidence-statistics

## Summary

This skill quantifies whether a gene's absence from a lineage matrix (e.g. ribosomal protein L30 across CPR/TM6 bacteria) is a true biological absence rather than an artifact of incomplete draft genomes, by computing the probability that a complete genome would have missed n genes given its completeness estimate.

## When to use

Use when a presence/absence analysis calls a gene absent in lineages represented mainly by draft bins — especially when completeness estimates (e.g. ≥75% single-copy genes, ≤1.125 copies) are available and you must decide if the absence call is trustworthy.

## When NOT to use

- The genome set consists only of complete genomes, so incomplete-sampling artifacts cannot explain absences
- Gene detection relies on keyword-based annotation alone rather than HMM profiles — divergent homologs would inflate apparent absences before any probability calculation is meaningful
- The genes of interest are not single-copy or cannot be enumerated (n undefined), making the (1 − c)^n computation inapplicable

## Inputs

- Binary lineage × ribosomal-protein presence/absence matrix (e.g. rpL30, rpL9, rpL1, Der) from HMMER hmmscan/Pfam and gg Kbase keyword inventories
- Per-genome completeness estimates from single-copy gene (SCG) analysis (e.g. median completeness 91% for WS6)
- Number of genes (n) claimed absent per lineage, from the near-complete genome filter (≥75% SCGs, ≤1.125 copies, assembled 16S rRNA gene)

## Outputs

- Per-lineage absence probability statistics, (1 − completeness)^n, supporting or refuting true gene absence
- Validated absence annotations in the lineage × protein matrix (e.g. all CPR and TM6 lack rpL30; specific lineages lack rpL9, rpL1, Der)

## How to apply

First build the binary presence/absence matrix only from genomes passing completeness filters (complete and near-complete bins with ≥75% single-copy genes, fewer than 1.125 copies of each, and an assembled 16S rRNA gene), inventorying ribosomal proteins via HMMER hmmscan against Pfam HMM profiles rather than keyword annotation. When a lineage (e.g. WS6) lacks complete genomes but has a median completeness estimate c (91% for WS6), compute the probability that a genome of that completeness would have missed all n genes of interest as (1 − c)^n; for the CPR case with 16 ribosomal protein genes, (1 − 0.91)^16 ≈ 2×10⁻⁷, which is small enough to conclude the absence is real rather than due to incomplete sampling. Use this probability as the decision threshold: if (1 − c)^n is not vanishingly small, flag the absence as uncertain instead of recording a true loss. Apply this per lineage, since completeness differs across CPR phyla; report the statistic alongside each absence call (e.g. result_all_cpr_lack_rpl30 in the lineage × protein table).

## Related tools

- **HMMER hmmscan** (Inventories ribosomal proteins against Pfam HMM profiles so absences are not artifacts of divergent sequences evading keyword annotation)
- **gg Kbase** (Supplies keyword-based ribosomal protein gene lists to cross-check the HMM-based inventory feeding the presence/absence matrix)

## Examples

```
python -c "c=0.91; n=16; p=(1-c)**n; print(f'absence probability: {p:.2e}')"
```

## Evaluation signals

- For each lineage flagged as lacking a gene, the reported (1 − completeness)^n value is small (e.g. ≈2×10⁻⁷ for WS6 with 16 genes at 91% completeness), supporting true absence
- Only bins passing the near-complete filter (≥75% single-copy genes, ≤1.125 copies, assembled 16S rRNA gene) contribute absence calls
- The absence call for rpL30 is corroborated by both the HMMER/Pfam hmmscan inventory and the gg Kbase keyword gene lists
- Absence calls are stratified per lineage (e.g. rpL9, rpL1, Der lost only in specific CPR lineages, rpL30 in all CPR/TM6), matching the published result_all_cpr_lack_rpl30 finding

## Limitations

- The statistic depends on accurate completeness estimates; draft bins (e.g. WS6, median completeness 91%) introduce residual uncertainty even when (1 − c)^n is tiny
- Absence calls assume the hmmscan/Pfam search would detect even highly divergent homologs; if HMM sensitivity were insufficient, absences would be over-called
- The threshold for what counts as a 'vanishingly small' absence probability is a judgment call and is not formally defined in the source

## Evidence

- [supplementary] check that the absence probability (1 − 0.91)^16 ≈ 2×10⁻⁷ supports a true gene absence rather than incomplete sampling: [evidence span withheld — anchor status: not_found]
