---
name: in-silico-pcr-primer-matching
description: Use when a set of primer pairs (designed or published) must be scored
  for detection of a curated reference gene set — e.g., computing what fraction of
  720 amoA-AOB target genes are amplified in silico, or comparing coverage between
  two primer sets.
when_to_use_negative:
- When experimental wet-lab qPCR validation is required — the article notes computational
  in silico results still require further experimental validation
- When the input is not a matched primer/reference-gene pair problem (e.g., you already
  have an amplification/feature table rather than sequences)
- When reference gene diversity is so high that MetaFunPrimer would design numerous
  primer pairs exceeding qPCR platform run capacity
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0091
tools:
- name: MetaFunPrimer
  role: Performs the in silico PCR evaluation, degenerate-to-nondegenerate primer
    expansion, and computes coverage of designed vs published primers against the
    reference gene set
  repo: https://github.com/rdpstaff/EcoFunPrimer
- name: EcoFunPrimer
  role: Supplies the thermodynamically stable primer pairs (78 amoA-AOB pairs) that
    are the input to the in silico PCR scoring step
  repo: https://github.com/rdpstaff/EcoFunPrimer
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:a1c44ced501873a1756709773308f58086410eebe0aa5dfd8feecd8528682d50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rdpstaff/EcoFunPrimer
  tools:
  - EcoFunPrimer
  - MetaFunPrimer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# in-silico-pcr-primer-matching

## Summary

Evaluates designed or published primer sets computationally by expanding degenerate primers to all nondegenerate forms and scoring amplification against reference gene sequences, using MetaFunPrimer's criterion that both forward and reverse primers must 100% match a sequence. This is how the paper quantified 94% (676/720) coverage for 78 designed amoA-AOB primer pairs versus 49% (356/720) for previously published primers.

## When to use

Use when a set of primer pairs (designed or published) must be scored for detection of a curated reference gene set — e.g., computing what fraction of 720 amoA-AOB target genes are amplified in silico, or comparing coverage between two primer sets.

## When NOT to use

- When experimental wet-lab qPCR validation is required — the article notes computational in silico results still require further experimental validation
- When the input is not a matched primer/reference-gene pair problem (e.g., you already have an amplification/feature table rather than sequences)
- When reference gene diversity is so high that MetaFunPrimer would design numerous primer pairs exceeding qPCR platform run capacity

## Inputs

- Designed primer pairs from MetaFunPrimer/EcoFunPrimer (78 amoA-AOB pairs, including degenerate forms)
- Previously published primer pairs for comparison (from Table S5-style prior efforts)
- Curated reference gene nucleotide sequences in FASTA (720 soil-abundant amoA-AOB genes from Fungene v9.6, HMM score >400, HMM coverage >70.2%)
- Sequence cluster assignments (clusters defined at 96% amino acid similarity threshold)

## Outputs

- Per-primer-set coverage table: number and fraction of reference gene products amplified (e.g., 676/720 = 94% MetaFunPrimer vs 356/720 = 49% published)
- Per-cluster coverage breakdown (e.g., 96% vs 19% for cluster 3)
- coverage_curve.csv mapping subset size k to cumulative gene coverage (full set of 78 pairs ~94%)
- Per-gene amplification verdicts satisfying the 100% both-primer match criterion

## How to apply

Load the primer pairs (e.g., the 78 amoA-AOB pairs output by EcoFunPrimer/MetaFunPrimer) and the reference gene nucleotide sequences (the 720 soil-abundant amoA-AOB genes from Fungene 9.6). Expand every degenerate primer into all of its nondegenerate forward/reverse forms using MetaFunPrimer's degeneracy conversion. Run in silico PCR against the reference set, scoring amplification as successful only when both the forward and reverse primers achieve a 100% match against a sequence. Compute the number and fraction of reference gene products amplified, overall and per sequence cluster (e.g., cluster 3 defined at 96% amino acid similarity), and compare against reported values. For primer-set-size analysis, evaluate cumulative coverage for subsets of size k and plot the coverage curve, annotating where it approaches the full-set ~94% coverage.

## Related tools

- **MetaFunPrimer** (Performs the in silico PCR evaluation, degenerate-to-nondegenerate primer expansion, and computes coverage of designed vs published primers against the reference gene set) — https://github.com/rdpstaff/EcoFunPrimer
- **EcoFunPrimer** (Supplies the thermodynamically stable primer pairs (78 amoA-AOB pairs) that are the input to the in silico PCR scoring step) — https://github.com/rdpstaff/EcoFunPrimer

## Evaluation signals

- Recomputed overall coverage matches reported values within rounding: 676/720 (94%) for MetaFunPrimer primers and 356/720 (49%) for published primers
- Cluster-3 recomputed coverage matches reported 96% (MetaFunPrimer) vs 19% (published)
- Every gene counted as amplified satisfies the 100% forward-and-reverse match criterion — spot-check individual gene/primer alignments
- Full-set (78-pair) coverage in the coverage curve equals ~94%, matching the reported full-set in silico detection
- coverage_curve.csv exists with exactly one coverage value per evaluated subset size k

## Limitations

- Only computational/in silico validation is provided; further experimental validation is required before trusting detection estimates
- Highly diverse functional genes may require numerous primer pairs and numerous qPCR runs, exceeding even HT-qPCR platform capacity
- Coverage estimates depend on the curated reference set (Fungene v9.6 with HMM score >400 and >70.2% coverage) and would differ for other gene sets or environments
- EcoFunPrimer itself is marked as still in development in its README

## Evidence

- [methods] In silico PCR both-primer 100% match criterion: [evidence span withheld — anchor status: not_found]
- [abstract] Reported coverage comparison for designed vs published primers: "these amoA-AOB primers can cover 94% of the amoA-AOB genes observed in the 1,550 soil metagenomes compared with a 49% estimated coverage by previously published primers"
- [intro] MetaFunPrimer evaluates primers against abundant reference genes: "evaluates these primers against hundreds of environmentally abundant functional genes"
- [methods] Minimal primer set selection goal: "MetaFunPrimer outputs the minimal number of primer sets to capture the maximum number of reference gene products"
