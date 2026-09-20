---
name: ribosomal-protein-inventory-annotation
description: Use when inventorying ribosomal protein complements across many draft/complete
  genome bins to test whether a supposedly universal protein (e.g. rpL30, rpL9, rpL1,
  Der) is truly absent from specific lineages.
when_to_use_negative:
- When the question is general functional annotation of genomes (metabolic pathways,
  KEGG/UniRef90 search) rather than ribosomal protein composition and absence testing
- When genome bins are too incomplete to meet the near-complete threshold (≥75% SCGs,
  ≤1.125 copies), since gene absence calls would be unreliable without an explicit
  absence-probability calculation
- When testing absence of non-ribosomal protein families that lack well-curated Pfam
  HMM profiles for detecting divergent homologs
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: HMMER hmmscan
  role: Search translated ORFs against Pfam ribosomal protein HMM profiles to inventory
    ribosomal proteins that keyword-based annotation misses due to sequence divergence
- name: gg Kbase
  role: Keyword gene lists used to supplement the HMM-based ribosomal protein inventory
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:57d1c56bbbbc33db48a680af01086d6b66a41426049d6e385b57143658e62195
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HMMER hmmscan
  - gg Kbase
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ribosomal-protein-inventory-annotation

## Summary

Build a lineage-by-ribosomal-protein presence/absence matrix across CPR/TM6 genome bins by combining HMMER hmmscan searches against Pfam ribosomal protein HMM profiles with gg Kbase keyword gene lists, to test claims that ribosomal proteins (e.g. rpL30) are absent from entire phyla.

## When to use

Use when inventorying ribosomal protein complements across many draft/complete genome bins to test whether a supposedly universal protein (e.g. rpL30, rpL9, rpL1, Der) is truly absent from specific lineages.

## When NOT to use

- When the question is general functional annotation of genomes (metabolic pathways, KEGG/UniRef90 search) rather than ribosomal protein composition and absence testing
- When genome bins are too incomplete to meet the near-complete threshold (≥75% SCGs, ≤1.125 copies), since gene absence calls would be unreliable without an explicit absence-probability calculation
- When testing absence of non-ribosomal protein families that lack well-curated Pfam HMM profiles for detecting divergent homologs

## Inputs

- ORF annotations from bacterial genome bins (translation table 11)
- Six-frame translations of genome bins
- Pfam ribosomal protein HMM profiles
- 16S rRNA gene / rp16 (concatenated ribosomal protein) phylogeny for lineage assignment
- gg Kbase keyword gene lists for ribosomal proteins

## Outputs

- Lineage x ribosomal-protein binary presence/absence matrix (rpL30, rpL9, rpL1, Der, others)
- Per-genome ribosomal protein inventory (hmmscan vs. Pfam hits)
- Per-lineage genome quality summary (completeness, SCG copy number)

## How to apply

Load ORF annotations and six-frame translations using bacterial translation table 11 for each genome bin (the paper analyzed 8 complete and 789 draft CPR/TM6 bins). Search all translated sequences against Pfam ribosomal protein HMM profiles with HMMER hmmscan, since divergent ribosomal proteins evade keyword-based annotation; supplement with gg Kbase keyword gene lists for ribosomal protein composition. Assign each genome to a lineage using the 16S rRNA gene (intron-removed, structurally aligned) / rp16 phylogeny. Restrict the final matrix to complete and near-complete genomes — ≥75% single-copy genes with ≤1.125 copies and an assembled 16S rRNA gene — and encode binary presence/absence per lineage for rpL30, rpL9, rpL1, Der and remaining ribosomal proteins. For lineages lacking complete genomes (e.g. WS6, median completeness 91%), compute the absence probability (1 − 0.91)^16 ≈ 2×10⁻⁷ to support true gene absence rather than incomplete sampling.

## Related tools

- **HMMER hmmscan** (Search translated ORFs against Pfam ribosomal protein HMM profiles to inventory ribosomal proteins that keyword-based annotation misses due to sequence divergence)
- **gg Kbase** (Keyword gene lists used to supplement the HMM-based ribosomal protein inventory)

## Examples

```
hmmscan --tblout rp_inventory.tbl --cpu 8 Pfam-A_ribosomal_protein.hmm orfs_table11.faa
```

## Evaluation signals

- hmmscan search independently confirms the keyword-based finding of missing ribosomal proteins in CPR lineages
- Matrix restricted to complete/near-complete bins: ≥75% of single-copy genes present, ≤1.125 copies per SCG, assembled 16S rRNA gene
- For incomplete lineages, absence probability ≈ (1 − completeness)^16 is vanishingly small (e.g. (1 − 0.91)^16 ≈ 2×10⁻⁷ for WS6)
- The matrix reproduces the reported pattern: all CPR/TM6 bacteria lack rpL30; specific lineages additionally lack rpL9, rpL1, and the biogenesis factor Der

## Limitations

- Draft-quality bins lack complete genomes for some lineages (e.g. WS6 at 91% median completeness), so absence must be supported probabilistically rather than observed directly
- Sequence-divergent ribosomal proteins evade keyword-based annotation, so keyword lists alone under-detect homologs and HMM-based search is required
- CPR 16S rRNA genes can carry large insertions (31% carried ≥10 bp insertions), so intron removal is critical before lineage assignment

## Evidence

- [methods] Insertion-encoded ORFs were searched against Pfam to associate encoded proteins with known families: [evidence span withheld — anchor status: not_found]
