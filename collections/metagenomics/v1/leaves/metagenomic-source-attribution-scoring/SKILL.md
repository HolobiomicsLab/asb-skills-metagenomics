---
name: metagenomic-source-attribution-scoring
description: Use when benchmarking metagenomic fecal source tracking predictions against
  samples with known source composition (e.
when_to_use_negative:
- No ground-truth source composition exists for the samples — specificity/sensitivity
  and apportionment accuracy cannot be computed, so only attribution/apportionment
  itself is possible, not scoring
- The goal is a mass-balance quantitation of contaminant mass — SourceApp portions
  estimate prokaryotic cell fractions via genome equivalents, not mass balances
- The question is the age of contamination or which signal comes from growth-capable
  (nonconservative) populations — SourceApp cannot infer either
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: SourceApp
  role: Produces the attribution, apportionment (Portion), and fractioning (Fraction)
    tables that are scored against ground-truth mesocosm cell fractions; parameters
    (-r 0.93, -q 0.7, LOD 0.01%, --use-geq) are tuned to maximize sensitivity/specificity
  repo: https://github.com/bglindner/SourceApp
- name: MicrobeCensus
  role: Estimates genome equivalents (GEQ) used to normalize summed mapped sequence
    depths into cell fractions (cell fraction_j = sum(X_i)/GEQ) for scoring against
    ground truth
- name: CoverM
  role: Read-mapping engine (v0.7.0) underlying the per-genome mapped read counts
    and sequence depths that feed the attribution and apportionment calculations
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:c37d1b5c9919de1264b8dc68f9bc6257b720051bc97ef2017c41b651eece09b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CoverM
  - MicrobeCensus
  - Python
  - SourceApp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metagenomic-source-attribution-scoring

## Summary

Scoring SourceApp's metagenomic fecal source attribution (presence/absence calls) and apportionment (per-source Portion/Fraction values) against ground-truth cell fractions from spike-in mesocosms of known composition, yielding specificity, sensitivity, and apportionment-accuracy estimates.

## When to use

Use when benchmarking metagenomic fecal source tracking predictions against samples with known source composition (e.g., spike-in mesocosms with recorded ground-truth cell fractions), or when tuning SourceApp parameters (percent identity, query coverage, LOD) to maximize attribution sensitivity/specificity.

## When NOT to use

- No ground-truth source composition exists for the samples — specificity/sensitivity and apportionment accuracy cannot be computed, so only attribution/apportionment itself is possible, not scoring
- The goal is a mass-balance quantitation of contaminant mass — SourceApp portions estimate prokaryotic cell fractions via genome equivalents, not mass balances
- The question is the age of contamination or which signal comes from growth-capable (nonconservative) populations — SourceApp cannot infer either

## Inputs

- Gzipped raw shotgun metagenomic reads in FASTQ format (e.g., 2 × 150-bp Illumina NovaSeq 6000 reads from mesocosms)
- Source-associated genome database with known source associations (SourceApp default database from Zenodo 10.5281/zenodo.12795076, or a de novo database built with sourceapp_build.py, including cross-reactive genome flags at ANI ≥ 95%)
- Ground-truth per-source cell fractions for each mesocosm (Supporting Information Table S1)
- Genome-equivalent estimates per metagenome (MicrobeCensus) for --use-geq normalization

## Outputs

- Attribution table (CSV): presence/absence call per source category per metagenome
- Apportionment table (CSV): Portion, the per-source fraction of total present-source cell fraction
- Fractioning table (CSV): Fraction in c/c units when --use-geq is used
- Performance metrics: attribution specificity (~0.898) and sensitivity (~0.902), and apportionment accuracy versus ground-truth cell fractions (linear correction r2 = 0.64 at best)

## How to apply

Run SourceApp on each shotgun metagenome (gzipped FASTQ reads) with tuned parameters — 93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection of 0.01% relative abundance — using --use-geq so Fraction is reported in c/c units normalized by MicrobeCensus genome equivalents. Apply the attribution rule: call a source present only when at least one source-specific genome is detected and the source's estimated cell fraction (sum of per-genome depths over genome equivalents) exceeds the 0.01% LOD. Compute apportionment Portion for each present source as its summed specific-plus-cross-reactive cell fraction divided by the sum of cell fractions of all present sources; cross-reactive genomes are those sharing ≥95% ANI with genomes in other source categories. Score the binary attribution calls against the ground-truth cell fractions recorded for the 32 mesocosms (Supporting Information Table S1) to compute specificity and sensitivity, and compare Portion/Fraction values to the constructed cell fractions, optionally assessing linear correction of estimates. Validation expectation: with the tuned parameters, attribution specificity should be near 0.898 and sensitivity near 0.902, which exceeded all alternative tools benchmarked.

## Related tools

- **SourceApp** (Produces the attribution, apportionment (Portion), and fractioning (Fraction) tables that are scored against ground-truth mesocosm cell fractions; parameters (-r 0.93, -q 0.7, LOD 0.01%, --use-geq) are tuned to maximize sensitivity/specificity) — https://github.com/bglindner/SourceApp
- **MicrobeCensus** (Estimates genome equivalents (GEQ) used to normalize summed mapped sequence depths into cell fractions (cell fraction_j = sum(X_i)/GEQ) for scoring against ground truth)
- **CoverM** (Read-mapping engine (v0.7.0) underlying the per-genome mapped read counts and sequence depths that feed the attribution and apportionment calculations) — https://github.com/wwood/CoverM

## Examples

```
python sourceapp.py --use-geq -r 0.93 -q 0.7 mesocosm_01_R1.fastq.gz mesocosm_01_R2.fastq.gz
```

## Evaluation signals

- With tuned parameters (93% identity, 70% overlap, 0% masking, LOD 0.01%), attribution specificity should be near 0.898 and sensitivity near 0.902
- A source is called present only if at least one source-specific genome is detected AND its cell fraction exceeds the 0.01% LOD — verify no calls violate this rule
- Portion values across all present sources in a metagenome must sum to 1, since each Portion is the source's cell fraction divided by the sum over present sources
- Fraction values should be in c/c (cell fraction) units only when --use-geq is passed; without it the genome-equivalent normalization is absent
- Apportionment accuracy is judged against the ground-truth cell fractions from mesocosm construction (SI Table S1); even best-case linear correction leaves residuals (r2 = 0.64), so large deviations for weakly sampled sources (e.g., cows) are expected

## Limitations

- SourceApp's performance varied substantially by source, with cows underestimated due to under sampling of cow fecal microbiomes (false negatives for cows point to poor genomic characterization of cow feces)
- The portions reported by SourceApp do not represent a mass balance but efforts to estimate proportions of cells via genome equivalents, and substantial residuals remain after linear correction (r2 = 0.64)
- SourceApp cannot infer the age of contamination or identify nonconservative (growth-capable) populations, and the environmental persistence of the species-level marker genomes is largely unknown
- SourceApp works best when users supply supplementary reference genomes to bolster the default database; performance degrades with under-sampled source categories

## Evidence

- [methods] Attribution rule: "the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default)"
- [results] Tuned parameters and performance: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [abstract] Benchmark outcome: "Parameter tuning revealed sensitivity and specificity near 0.90 overall"
- [results] Interpretation of Portion: "the portions reported by SourceApp do not represent the results of a mass balance but rather efforts to estimate the proportions of cells belonging to source categories based on genome equivalents"
- [readme] Output semantics and benchmarking: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called) and has been benchmarked in laboratory trials"
- [results] Apportionment accuracy ceiling: "substantial residuals remained after linear correction even with the best performing parameters (r 2 = 0.64"
