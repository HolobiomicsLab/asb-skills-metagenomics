---
name: gene-cluster-coverage-quantification
description: Use when you need to benchmark primer-set sensitivity against a curated
  reference gene set, e.g., recomputing the 94% (MetaFunPrimer) vs 49% (published)
  amoA-AOB coverage or per-cluster coverage like cluster 3 at 96% vs 19%.
when_to_use_negative:
- Input is not a fixed reference gene set with known size (the denominator, e.g.,
  720 targets, is required to compute coverage fractions)
- The comparator primers should not be benchmarked by thermodynamic PCR simulation
  rather than the strict 100% both-primer match criterion used by MetaFunPrimer
- Sequences have not been filtered/curated (unfiltered Fungene-style inputs lack the
  HMM score >400 and coverage over 70.2% quality gate, distorting coverage estimates)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0080
tools:
- name: MetaFunPrimer
  role: Performs in silico PCR of degeneracy-expanded primer pairs against curated
    reference gene sequences and applies the 100% both-primer match amplification
    criterion
  repo: github.com/jialiu232/MetaFunPrimer
- name: EcoFunPrimer
  role: Embedded in MetaFunPrimer; generates degenerate or nondegenerate primer pairs
    that are the input for coverage evaluation
  repo: https://github.com/rdpstaff/EcoFunPrimer
- name: CD-HIT
  role: Clusters reference sequences (80% to 100% similarity in 1% increments) to
    define per-cluster coverage strata such as the 96% threshold
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:2d0254f787cf8a1bc31c4415a8a7df7bb65bebe7733442b3ae577784d12b9b3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - EcoFunPrimer
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gene-cluster-coverage-quantification

## Summary

Quantifies how many reference gene products (e.g., 720 curated amoA-AOB genes from Fungene v9.6) are amplified by a set of designed vs. published primers via in silico PCR, reporting coverage per set and per sequence cluster (e.g., 96% amino acid similarity clusters).

## When to use

Use when you need to benchmark primer-set sensitivity against a curated reference gene set, e.g., recomputing the 94% (MetaFunPrimer) vs 49% (published) amoA-AOB coverage or per-cluster coverage like cluster 3 at 96% vs 19%.

## When NOT to use

- Input is not a fixed reference gene set with known size (the denominator, e.g., 720 targets, is required to compute coverage fractions)
- The comparator primers should not be benchmarked by thermodynamic PCR simulation rather than the strict 100% both-primer match criterion used by MetaFunPrimer
- Sequences have not been filtered/curated (unfiltered Fungene-style inputs lack the HMM score >400 and coverage over 70.2% quality gate, distorting coverage estimates)

## Inputs

- Designed primer pairs from MetaFunPrimer output (e.g., 78 degenerate amoA-AOB primer pairs)
- Previously published comparator primer set (e.g., published amoA-AOB primers)
- Curated reference gene nucleotide sequences (720 amoA-AOB targets from Fungene version 9.6, HMM score >400 and coverage over 70.2%)
- Cluster assignments for reference genes (e.g., CD-HIT clusters at 96% amino acid similarity)

## Outputs

- Coverage table: number and fraction of the 720 reference gene products amplified per primer set (e.g., 676/720 = 94% for MetaFunPrimer, 356/720 = 49% for published)
- Per-cluster coverage fractions (e.g., cluster 3: 96% vs 19%)
- Per-gene amplification calls satisfying the 100% both-primer match criterion

## How to apply

Load the designed primer pairs (e.g., 78 amoA-AOB pairs from MetaFunPrimer) plus the published comparator primer set, and expand every degenerate primer into all nondegenerate forward/reverse forms using MetaFunPrimer's degeneracy conversion. Run in silico PCR against the curated reference nucleotide sequences (720 amoA-AOB targets from the Fungene-derived set), scoring amplification as successful only when both the forward and reverse primers achieve a 100% match against a sequence. Aggregate hits into counts and fractions of the 720 reference gene products amplified, overall and per cluster (e.g., clusters defined at the 96% amino acid similarity threshold used for CD-HIT clustering). Accept the coverage table only if recomputed fractions match reported values (94% vs 49% overall; 96% vs 19% for cluster 3) within rounding and every counted gene satisfies the both-primer 100% match criterion.

## Related tools

- **MetaFunPrimer** (Performs in silico PCR of degeneracy-expanded primer pairs against curated reference gene sequences and applies the 100% both-primer match amplification criterion) — github.com/jialiu232/MetaFunPrimer
- **EcoFunPrimer** (Embedded in MetaFunPrimer; generates degenerate or nondegenerate primer pairs that are the input for coverage evaluation) — https://github.com/rdpstaff/EcoFunPrimer
- **CD-HIT** (Clusters reference sequences (80% to 100% similarity in 1% increments) to define per-cluster coverage strata such as the 96% threshold)

## Evaluation signals

- Recomputed overall coverage matches reported values within rounding: 676/720 (94%) for MetaFunPrimer-designed primers and 356/720 (49%) for published primers
- Per-cluster recomputed coverage matches reported values (e.g., cluster 3 at the 96% amino acid similarity threshold: 96% vs 19%)
- Every gene counted as amplified satisfies the 100% both-primer match criterion (forward and reverse)
- Denominator equals the full curated reference set size (720 amoA-AOB gene products) in all fraction computations
- All degenerate primers were fully expanded into nondegenerate forms before the in silico PCR, with no primer left unexpanded

## Limitations

- Coverage is computationally estimated only; the article notes that 'further experimental validation is required' and results are not a substitute for wet-lab qPCR testing
- The strict 100% both-primer match criterion means primers with mismatches are counted as failures, so coverage is a lower bound relative to looser in silico PCR criteria
- Highly diverse functional genes may require numerous primer pairs and numerous qPCR runs, potentially exceeding platform capacity

## Evidence

- [methods] both-primer 100% match criterion: [evidence span withheld — anchor status: not_found]
- [abstract] reported coverage contrast: "these amoA-AOB primers can cover 94% of the amoA-AOB genes observed in the 1,550 soil metagenomes compared with a 49% estimated coverage by previously published primers"
- [methods] reference set curation thresholds: "filtering for a Hidden Markov Model (HMM) search score of >400 and HMM coverage over 70.2% amino acid similarity"
- [results] cluster threshold used for per-cluster coverage: "clustering sequences at greater than 96% similarity resulted in the largest increase in the resulting total unique clusters"
