---
name: primer-degeneracy-expansion
description: Use when you have degenerate primer sequences (IUPAC ambiguity codes)
  from a primer-design tool like MetaFunPrimer or EcoFunPrimer and need nondegenerate
  variants to test 100% both-primer match coverage against reference gene sequences.
when_to_use_negative:
- Reference sequences contain ambiguous bases or sequencing errors, so a strict 100%
  both-primer match criterion would undercount real amplification
- The goal is thermodynamic primer quality assessment (Tm, hairpin, homodimer) rather
  than enumerating degenerate variants — use EcoFunPrimer's design parameters instead
- Primer pairs are already nondegenerate; expansion is a no-op and direct matching
  suffices
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: MetaFunPrimer
  role: Outputs degenerate amoA-AOB primer pairs and provides the degeneracy conversion
    used to expand them into nondegenerate forms for in silico PCR
  repo: https://github.com/jialiu232/MetaFunPrimer_paper_info.git
- name: EcoFunPrimer
  role: Embedded primer design component that generates degenerate or nondegenerate
    primers; defines degeneracy bounds (default degenMax 8, nondegenerate = 1)
  repo: https://github.com/rdpstaff/EcoFunPrimer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:5996f3df3bb5dede2e56d489bbfd6b99680db4a8d7b3c2fcbe59c12299d7f7f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jialiu232/MetaFunPrimer_paper_info.git
  tools:
  - EcoFunPrimer
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# primer-degeneracy-expansion

## Summary

Expand degenerate primer outputs (e.g., the 78 amoA-AOB primer pairs designed by MetaFunPrimer/EcoFunPrimer) into all constituent nondegenerate forward/reverse forms so that per-sequence primer matching can be scored exactly during in silico PCR coverage evaluation.

## When to use

Use when you have degenerate primer sequences (IUPAC ambiguity codes) from a primer-design tool like MetaFunPrimer or EcoFunPrimer and need nondegenerate variants to test 100% both-primer match coverage against reference gene sequences.

## When NOT to use

- Reference sequences contain ambiguous bases or sequencing errors, so a strict 100% both-primer match criterion would undercount real amplification
- The goal is thermodynamic primer quality assessment (Tm, hairpin, homodimer) rather than enumerating degenerate variants — use EcoFunPrimer's design parameters instead
- Primer pairs are already nondegenerate; expansion is a no-op and direct matching suffices

## Inputs

- Degenerate primer pair sequences output by MetaFunPrimer (78 amoA-AOB pairs in the case study)
- Previously published degenerate primer sets for comparison
- Curated reference gene nucleotide sequences (720 amoA-AOB targets from Fungene v9.6, HMM score >400 and coverage >70.2%)
- Cluster assignments of reference sequences (e.g., 96% amino acid similarity clusters)

## Outputs

- Expanded set of nondegenerate forward/reverse primer variants per degenerate primer pair
- Per-primer-set amplification table over the 720 reference genes under the 100% both-primer match criterion
- Coverage fractions overall and per cluster (e.g., 94% vs 49% overall; 96% vs 19% for cluster 3)

## How to apply

Take the degenerate primer pairs output by MetaFunPrimer (in the case study, 78 amoA-AOB primer pairs) and use the pipeline's degeneracy conversion to enumerate every nondegenerate forward and reverse primer form implied by each IUPAC ambiguity code. Treat a nondegenerate primer as degeneracy 1, consistent with EcoFunPrimer's definition that 'Nondegenerate primers = 1' and its default maximum degeneracy of 8 per primer pair. Run the expanded primers in an in silico PCR against the curated reference gene nucleotide set (here, 720 amoA-AOB genes from Fungene v9.6), scoring amplification as successful only when both the forward and reverse primers achieve a 100% match against a sequence. Summarize the amplified fraction per primer set and per sequence cluster (e.g., cluster 3 defined at 96% amino acid similarity) to compare against expected coverages such as 676/720 (94%) for MetaFunPrimer primers versus 356/720 (49%) for published primers.

## Related tools

- **MetaFunPrimer** (Outputs degenerate amoA-AOB primer pairs and provides the degeneracy conversion used to expand them into nondegenerate forms for in silico PCR) — https://github.com/jialiu232/MetaFunPrimer_paper_info.git
- **EcoFunPrimer** (Embedded primer design component that generates degenerate or nondegenerate primers; defines degeneracy bounds (default degenMax 8, nondegenerate = 1)) — https://github.com/rdpstaff/EcoFunPrimer

## Evaluation signals

- Every gene counted as amplified satisfies the 100% forward and reverse primer match criterion against the 720-reference nucleotide set
- Recomputed overall coverages match reported values within rounding: 676/720 (94%) for MetaFunPrimer primers and 356/720 (49%) for published primers
- Cluster-level coverage matches reported values (e.g., 96% vs 19% for cluster 3 at the 96% amino acid similarity threshold)
- Total enumerated nondegenerate variants per primer pair equal the product of the degeneracies implied by its IUPAC ambiguity codes, with no primer exceeding the configured maximum degeneracy

## Limitations

- In silico coverage relies on purely computational validation; the article notes 'further experimental validation is required, but computational efforts can help determine which candidates to test experimentally'
- Highly diverse targets can yield numerous primer pairs with high degeneracy; EcoFunPrimer recommends degenMax 'no higher than 10' per primer pair
- 100% match scoring is strict — single-base mismatches between an expanded nondegenerate primer and a reference gene count as non-amplification

## Evidence

- [methods] a pair of primers was considered to amplify a gene product successfully if both the forward and reverse primers achieve a 100% match against a sequence: [evidence span withheld — anchor status: not_found]
- [abstract] these amoA-AOB primers can cover 94% of the amoA-AOB genes observed in the 1,550 soil metagenomes compared with a 49% estimated coverage by previously published primers: "these amoA-AOB primers can cover 94% of the amoA-AOB genes observed in the 1,550 soil metagenomes compared with a 49% estimated coverage by previously published primers"
- [results] EcoFunPrimer is a primer design tool which is capable of generating degenerate or nondegenerate primers based on input genes and parameters defined by the user.: "EcoFunPrimer is a primer design tool which is capable of generating degenerate or nondegenerate primers based on input genes and parameters defined by the user."
- [results] clustering sequences at greater than 96% similarity resulted in the largest increase in the resulting total unique clusters: "clustering sequences at greater than 96% similarity resulted in the largest increase in the resulting total unique clusters"
