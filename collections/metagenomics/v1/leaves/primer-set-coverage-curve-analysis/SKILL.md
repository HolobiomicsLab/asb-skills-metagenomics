---
name: primer-set-coverage-curve-analysis
description: 'Use when a designed primer set (e.g., 78 nondegenerate amoA-AOB pairs)
  must be pared down or evaluated: you need the minimal subset of primer pairs that
  captures maximum reference gene diversity, quantified as in silico PCR coverage.'
when_to_use_negative:
- When the goal is to design new primers from raw gene/metagenome input rather than
  evaluate coverage of an existing designed set — use MetaFunPrimer's design pipeline
  instead
- When primer–template matching tolerates mismatches (e.g., in silico PCR with relaxed
  3'-end rules); this skill's coverage numbers are defined only by the 100% forward-and-reverse
  match criterion
- When genes are too diverse for a practical qPCR panel — the article notes MetaFunPrimer
  'will likely design numerous primer pairs, which may require numerous runs, even
  on HT-qPCR platforms', so coverage curves for such targets may not identify a practical
  subset
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaFunPrimer
  role: Designs the primer pairs, ranks genes by R-score, expands degenerate primers
    to nondegenerate forms, and evaluates in silico PCR coverage against the reference
    gene set
  repo: github.com/jialiu232/MetaFunPrimer
- name: EcoFunPrimer
  role: Embedded in MetaFunPrimer; generates degenerate or nondegenerate thermodynamically
    screened primer pairs from aligned nucleotide input that are then subset and scored
    for coverage
  repo: https://github.com/rdpstaff/EcoFunPrimer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:00e4de4f873eb524bc355bac2fdf4891af25fd4f0a55f586c11f4e11574bd6e1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoFunPrimer
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# primer-set-coverage-curve-analysis

## Summary

Computes the tradeoff between the number of designed primer pairs (k) and cumulative in silico gene coverage, following MetaFunPrimer's approach for the amoA-AOB case study where 78 nondegenerate primer pairs achieved 94% coverage (676/720 reference genes) versus 49% for previously published primers.

## When to use

Use when a designed primer set (e.g., 78 nondegenerate amoA-AOB pairs) must be pared down or evaluated: you need the minimal subset of primer pairs that captures maximum reference gene diversity, quantified as in silico PCR coverage.

## When NOT to use

- When the goal is to design new primers from raw gene/metagenome input rather than evaluate coverage of an existing designed set — use MetaFunPrimer's design pipeline instead
- When primer–template matching tolerates mismatches (e.g., in silico PCR with relaxed 3'-end rules); this skill's coverage numbers are defined only by the 100% forward-and-reverse match criterion
- When genes are too diverse for a practical qPCR panel — the article notes MetaFunPrimer 'will likely design numerous primer pairs, which may require numerous runs, even on HT-qPCR platforms', so coverage curves for such targets may not identify a practical subset

## Inputs

- Designed primer pairs with coordinates (e.g., 78 MetaFunPrimer amoA-AOB pairs; nondegenerate expanded forms for degenerate primers)
- Curated reference gene nucleotide set (e.g., amoA-AOB from RDP Fungene v9.6, HMM score >400 and HMM coverage >70.2%)
- Optional: per-pair gene-capture ranking (MetaFunPrimer output, ordered by incremental reference gene products captured)

## Outputs

- coverage_curve.csv with a coverage value for each subset size k
- Coverage-vs-primer-set-size plot annotated at the full-set coverage point (~94% for 78 amoA-AOB pairs)
- Comparison statistic versus previously published primers (94% vs 49% baseline for amoA-AOB)

## How to apply

Load the full designed primer set (e.g., 78 MetaFunPrimer primer pairs for soil-abundant amoA-AOB) and the curated reference gene set (e.g., amoA-AOB sequences from Ribosomal Database Project Fungene v9.6, filtered at HMM score >400 and >70.2% coverage). For each candidate subset size k, order pairs by incremental gene capture, expand any degenerate primers to all nondegenerate forms, and run in silico PCR against the reference genes using MetaFunPrimer's criterion that a pair amplifies a gene product successfully only if both forward and reverse primers achieve a 100% match. Compute cumulative coverage (fraction of reference gene products amplified) for each k and plot the coverage curve, annotating where coverage approaches the full-set value (94% for the 78-pair amoA-AOB set). Rationale: MetaFunPrimer explicitly characterizes 'the minimal number of primer sets to capture the maximum number of reference gene products,' and gene clusters are typically prioritized by cumulative R-score (80% in the case study) so subset ordering reflects environment-specific abundance and prevalence.

## Related tools

- **MetaFunPrimer** (Designs the primer pairs, ranks genes by R-score, expands degenerate primers to nondegenerate forms, and evaluates in silico PCR coverage against the reference gene set) — github.com/jialiu232/MetaFunPrimer
- **EcoFunPrimer** (Embedded in MetaFunPrimer; generates degenerate or nondegenerate thermodynamically screened primer pairs from aligned nucleotide input that are then subset and scored for coverage) — https://github.com/rdpstaff/EcoFunPrimer

## Evaluation signals

- coverage_curve.csv exists and contains one coverage value per subset size k from 1 to the full set (78 for the amoA-AOB case)
- Coverage at the full subset size reproduces the reported ~94% (676/720) for the 78 amoA-AOB primer pairs under the 100% forward-and-reverse match criterion
- Cumulative coverage is monotonically nondecreasing as pairs are added in incremental-capture order
- Previously published primer baseline evaluates to ~49% (356/720) on the same 720 reference genes, matching the paper's comparison
- Every counted amplification involves nondegenerate expanded primer forms, matching the full set's nondegenerate expansion

## Limitations

- Coverage is a computational estimate under a strict 100% forward-and-reverse match rule; further experimental validation is required before trusting coverage as lab performance
- Highly diverse gene targets may yield numerous primer pairs exceeding HT-qPCR platform capacity, so a coverage plateau may still imply an impractical assay count
- Coverage depends on the reference gene set definition (Fungene v9.6 amoA-AOB with HMM score >400 and >70.2% coverage); a different reference set yields different curve values

## Evidence

- [methods] 100% forward-and-reverse match criterion for successful amplification: [evidence span withheld — anchor status: not_found]
- [methods] Minimal primer set objective: "MetaFunPrimer outputs the minimal number of primer sets to capture the maximum number of reference gene products"
- [abstract] 78 primer pairs case study scale: "a total of 78 primer pairs were designed to target observed ammonia monooxygenase subunit A (amoA) genes of ammonia-oxidizing bacteria (AOB)"
- [intro] Skill's core question is a stated capability: "characterizes the number of primer pairs that is required to capture the diversity in a given reference gene set"
- [methods] Gene cluster inclusion threshold for prioritization: "gene clusters are included until the user-input threshold of the cumulative R-score (i.e., 80% in the case study) is reached"
- [discussion] Practical assay-count limitation: "MetaFunPrimer will likely design numerous primer pairs, which may require numerous runs, even on HT-qPCR platforms."
