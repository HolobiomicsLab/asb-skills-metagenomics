---
name: hmm-profile-protein-family-detection
description: Use when building protein-family inventories or presence/absence matrices
  from predicted ORFs whose sequence divergence defeats keyword-based annotation —
  e.g. surveying ribosomal protein L30, L9, L1 absence across divergent lineages such
  as CPR and TM6.
when_to_use_negative:
- When the goal is annotating well-conserved, already-keyword-labeled proteins in
  non-divergent reference genomes — keyword gene lists are sufficient and faster
- 'When genomes are highly fragmentary: absence calls from missing HMM hits are unreliable
  unless the bin meets the near-complete quality gate (≥75% single-copy genes, ≤1.125
  copies, assembled 16S rRNA gene)'
- When the family of interest has no curated Pfam HMM profile — hmmscan cannot detect
  it and a custom profile or keyword search is required
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0621
tools:
- name: HMMER hmmscan
  role: Scans translated ORF sets against Pfam ribosomal protein HMM profiles to inventory
    ribosomal proteins despite sequence divergence
- name: gg Kbase
  role: Supplies keyword ribosomal protein gene lists used to supplement and cross-check
    the HMM-based inventory
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:e5a39bed723bf74c59b025c0a08b946676cfa6113ce58505e15ea049ce724c85
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HMMER hmmscan
  - gg Kbase
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# hmm-profile-protein-family-detection

## Summary

Inventory protein families (e.g. ribosomal proteins rpL30, rpL9, rpL1, Der) across genome bins by scanning all translated ORFs against Pfam HMM profiles with HMMER hmmscan, capturing divergent homologs that keyword-based annotation misses.

## When to use

Use when building protein-family inventories or presence/absence matrices from predicted ORFs whose sequence divergence defeats keyword-based annotation — e.g. surveying ribosomal protein L30, L9, L1 absence across divergent lineages such as CPR and TM6.

## When NOT to use

- When the goal is annotating well-conserved, already-keyword-labeled proteins in non-divergent reference genomes — keyword gene lists are sufficient and faster
- When genomes are highly fragmentary: absence calls from missing HMM hits are unreliable unless the bin meets the near-complete quality gate (≥75% single-copy genes, ≤1.125 copies, assembled 16S rRNA gene)
- When the family of interest has no curated Pfam HMM profile — hmmscan cannot detect it and a custom profile or keyword search is required

## Inputs

- Predicted ORF annotations and six-frame translations from complete and draft genome bins (bacterial translation table 11)
- Pfam ribosomal protein HMM profiles (HMMER format)
- Keyword-based ribosomal protein gene lists (gg Kbase) for cross-validation
- Lineage assignments per genome from the 16S rRNA gene / rp16 phylogeny

## Outputs

- Per-genome ribosomal protein inventory (hmmscan hits with family assignments)
- Lineage × protein binary presence/absence matrix for rpL30, rpL9, rpL1, Der and remaining ribosomal proteins (Supplementary Data 4 / Table 1)

## How to apply

Predict ORFs from each genome bin (complete and draft-quality CPR/TM6 assemblies) and translate them using bacterial translation table 11. Scan all translated sequences against Pfam ribosomal protein HMM profiles with HMMER hmmscan; HMM profiles are essential here because divergent CPR homologs evade ggKbase-style keyword gene lists. Supplement the HMM inventory with keyword gene lists (ggKbase) as a cross-check. Assign each genome to its lineage via the 16S rRNA gene / rp16 phylogeny (CPR rooted with Chloroflexi, TM6 included), then restrict the final presence/absence matrix to complete or near-complete genomes (≥75% single-copy genes, ≤1.125 copies, assembled 16S rRNA gene) and encode binary presence/absence per lineage. For draft lineages (e.g. WS6, median completeness 91%), support true absence with a sampling probability such as (1 − 0.91)^16 ≈ 2×10⁻⁷ rather than declaring absence from missing hits alone.

## Related tools

- **HMMER hmmscan** (Scans translated ORF sets against Pfam ribosomal protein HMM profiles to inventory ribosomal proteins despite sequence divergence)
- **gg Kbase** (Supplies keyword ribosomal protein gene lists used to supplement and cross-check the HMM-based inventory)

## Examples

```
hmmscan --tblout rp_hits.tbl Pfam-A.hmm translated_orfs.faa > rp_scan.txt
```

## Evaluation signals

- hmmscan search independently confirms the initially observed absence of ribosomal proteins (e.g. rpL30) in CPR lineages
- Keyword-based ggKbase gene lists and HMM hits agree on presence calls where both exist; discrepancies flag divergent homologs
- Every lineage in the matrix is backed by a bin passing the quality gate: ≥75% single-copy genes, ≤1.125 copies, assembled 16S rRNA gene
- For draft-only lineages, computed absence probability (1 − completeness)^n is small enough to support true gene absence (e.g. ≈2×10⁻⁷ at 91% completeness across 16 genes)
- Matrix entries are strictly binary (present/absent) and consistent with the ribosomal protein HMM hit table

## Limitations

- Draft-quality bins can cause false absences; absence claims must be backed by completeness-aware probability checks, not just missing hits
- Keyword-based annotation alone misses divergent homologs, so keyword lists are unreliable as a sole inventory method
- Absence calls depend on lineage assignment from the 16S rRNA gene / rp16 phylogeny; divergent 16S sequences complicate placement

## Evidence

- [other] HMM inventory over keyword annotation: [evidence span withheld — anchor status: not_found]
- [other] Absence probability check for draft lineages: [evidence span withheld — anchor status: not_found]
- [methods] Pfam search precedent in the article methods: [evidence span withheld — anchor status: not_found]
- [other] Quality gate for the presence/absence matrix: [evidence span withheld — anchor status: not_found]
