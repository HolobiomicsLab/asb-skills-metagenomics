---
name: presence-absence-matrix-construction
description: Use when you need to establish which ribosomal proteins (or other universal
  single-copy proteins) are present or absent across bacterial lineages from genome
  bins of mixed completeness, especially where sequence divergence defeats keyword-based
  gene calling.
when_to_use_negative:
- Genome bins that fail the quality cutoffs (≥75% single-copy genes with ≤1.125 copies
  and an assembled 16S rRNA gene) — apparent absences may reflect incomplete sampling
- Input is 16S amplicon data or assemblies without predicted ORFs — no protein inventory
  can be built
- Organisms outside the survey scope where the target protein is known to be variably
  present (e.g. ribosomal protein L30, often absent in symbionts) — absence is not
  lineage-diagnostic without the phylogenetic assignment
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: HMMER hmmscan
  role: Scans translated ORFs and six-frame translations against Pfam ribosomal protein
    HMM profiles to inventory ribosomal proteins, including divergent homologs missed
    by keyword annotation
- name: gg Kbase
  role: Supplies keyword-based ribosomal protein gene lists used to supplement the
    HMM-based inventory
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:9c57d14c827ce3d19641f50cc389815f7b84eadd97fa239b0d2e8d5bbb188175
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HMMER hmmscan
  - gg Kbase
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# presence-absence-matrix-construction

## Summary

Builds a lineage-by-protein binary presence/absence matrix (e.g. rpL30, rpL9, rpL1, Der across CPR and TM6 lineages) by inventorying ribosomal proteins with HMMER hmmscan against Pfam HMM profiles and ggKbase keyword gene lists, so that divergent homologs missed by keyword annotation are still detected.

## When to use

Use when you need to establish which ribosomal proteins (or other universal single-copy proteins) are present or absent across bacterial lineages from genome bins of mixed completeness, especially where sequence divergence defeats keyword-based gene calling.

## When NOT to use

- Genome bins that fail the quality cutoffs (≥75% single-copy genes with ≤1.125 copies and an assembled 16S rRNA gene) — apparent absences may reflect incomplete sampling
- Input is 16S amplicon data or assemblies without predicted ORFs — no protein inventory can be built
- Organisms outside the survey scope where the target protein is known to be variably present (e.g. ribosomal protein L30, often absent in symbionts) — absence is not lineage-diagnostic without the phylogenetic assignment

## Inputs

- ORF predictions and six-frame translations (translation table 11) for complete and draft genome bins
- Pfam ribosomal protein HMM profiles for hmmscan
- ggKbase keyword gene lists for ribosomal proteins
- 16S rRNA gene / rp16 phylogeny for lineage assignment
- Genome quality metrics (single-copy gene completeness, copy numbers, assembled 16S rRNA gene)

## Outputs

- Binary lineage-by-protein presence/absence matrix (rpL30, rpL9, rpL1, Der and remaining ribosomal proteins)
- HMMER hmmscan hit tables for ribosomal protein families per genome
- Per-lineage absence-probability calculations for draft-only lineages

## How to apply

Predict ORFs and six-frame translations using bacterial translation table 11 for the genome bins (here 8 complete and 789 draft CPR/TM6 genomes), then scan all translated sequences against Pfam ribosomal protein HMM profiles with HMMER hmmscan to inventory ribosomal proteins, capturing diverged homologs that evade keyword annotation. Supplement the HMM-based inventory with ggKbase keyword gene lists for ribosomal protein composition. Assign each genome to a lineage using the 16S rRNA gene / rp16 phylogeny (CPR rooted with Chloroflexi, TM6 included). Restrict the final matrix to complete and near-complete genomes — ≥75% single-copy genes with ≤1.125 copies and an assembled 16S rRNA gene (draft bins needed at least 50% SCGs with fewer than 1.125 copies) — and encode binary presence/absence per lineage. For lineages lacking complete genomes (e.g. WS6, median completeness 91%), compute the absence probability (1 − 0.91)^16 ≈ 2×10⁻⁷ to argue a true gene absence rather than incomplete sampling.

## Related tools

- **HMMER hmmscan** (Scans translated ORFs and six-frame translations against Pfam ribosomal protein HMM profiles to inventory ribosomal proteins, including divergent homologs missed by keyword annotation)
- **gg Kbase** (Supplies keyword-based ribosomal protein gene lists used to supplement the HMM-based inventory)

## Examples

```
hmmscan --cpu 8 --tblout rps_tblout.txt Pfam-A.hmm orfs_table11.faa
```

## Evaluation signals

- hmmscan search independently confirms the initial finding of missing ribosomal proteins in CPR lineages (e.g. result_all_cpr_lack_rpl30)
- Every genome in the final matrix passes ≥75% single-copy genes with ≤1.125 copies and has an assembled 16S rRNA gene
- Matrix cells are strictly binary (0/1) and every genome is assigned to a lineage via the 16S/rp16 phylogeny
- For draft-only lineages, computed absence probability should be vanishingly small — e.g. (1 − 0.91)^16 ≈ 2×10⁻⁷ for WS6
- Known absences match the published pattern: all CPR and TM6 bacteria lack rpL30, and specific lineages lack rpL9, rpL1, and Der

## Limitations

- Absence calls in draft genomes depend on completeness; the paper argues absence via probability (1 − completeness)^number_of_genes, which is only meaningful for lineages with high median completeness (e.g. WS6 at 91%)
- Keyword-based annotation alone misses ribosomal proteins due to sequence divergence, so relying on ggKbase gene lists without HMM validation risks false absences
- The matrix is restricted to complete and near-complete genomes, so lineages without such genomes are excluded or flagged rather than asserted absent

## Evidence

- [methods] Insertion-encoded ORFs were searched against Pfam to associate encoded proteins with known families: [evidence span withheld — anchor status: not_found]
