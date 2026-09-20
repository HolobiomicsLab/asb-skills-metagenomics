---
name: strain-classification-threshold-application
description: Use when pairwise genome or metagenome comparisons must be split into
  same-strain vs. different-strain calls, especially when SNP-based tools alone miss
  structurally divergent strains (e.g., clonal popANI but wide APSS range).
when_to_use_negative:
- When the species under study lacks a training dataset or known strain pairs to calibrate
  thresholds — cutoffs are subjective and species-dependent, so a generic cutoff may
  misclassify
- When only SNP-level differences matter and structural variation is irrelevant —
  inStrain popANI alone suffices and SynTracker's SNP insensitivity adds little
- When comparing phages, plasmids, or very low-data contexts where strain delineation
  thresholds calibrated on cellular genomes do not apply
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: SynTracker
  role: Computes APSS (average pairwise synteny scores) from genome synteny of homologous
    regions, used as the structural-variation axis of the classification
  repo: https://github.com/leylabmpi/SynTracker
- name: inStrain
  role: Computes popANI values for each isolate pair, used as the SNP-based axis of
    the same-strain classification (popANI > 0.99999 cutoff)
- name: BLAST
  role: SynTracker builds a BLAST database of target genomes and searches 1-kbp central
    regions with minimal identity 97% and minimal query coverage 70% before APSS calculation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:cb450569e1cd2109600384f4a8ea58544d7c9ee3b328c6a6ac791dd446adc5a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - SynTracker
  - inStrain
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# strain-classification-threshold-application

## Summary

Applies combined APSS (SynTracker synteny) and popANI (inStrain SNP-based) thresholds to classify isolate or metagenome pairs as same vs. different strains, enabling detection of strains enriched in either structural changes or point mutations.

## When to use

Use when pairwise genome or metagenome comparisons must be split into same-strain vs. different-strain calls, especially when SNP-based tools alone miss structurally divergent strains (e.g., clonal popANI but wide APSS range).

## When NOT to use

- When the species under study lacks a training dataset or known strain pairs to calibrate thresholds — cutoffs are subjective and species-dependent, so a generic cutoff may misclassify
- When only SNP-level differences matter and structural variation is irrelevant — inStrain popANI alone suffices and SynTracker's SNP insensitivity adds little
- When comparing phages, plasmids, or very low-data contexts where strain delineation thresholds calibrated on cellular genomes do not apply

## Inputs

- SynTracker APSS tables (avg_synteny_scores_all_regions.csv or avg_synteny_scores_200_regions.csv) per species/reference
- inStrain popANI values for every isolate pair from SRA collections (Supplementary Tables 2-5)
- Per-species references: GCF_900087635.2 (N. gonorrhoeae), NC_000913.3 (E. coli), CP032479.1 (H. pylori), GCF_000331185.2 (S. rimosus)

## Outputs

- Four same/different strain classification tables corresponding to Fig. 3a-d (N. gonorrhoeae, hypermutator E. coli, H. pylori within-host, S. rimosus)
- Per-pair APSS and popANI continuous values with applied same-strain cutoff labels

## How to apply

Compute APSS from SynTracker's 'avg_synteny_scores_all_regions' (or subsampled) tables and popANI from inStrain pair runs, then apply the same-strain cutoffs APSS > 0.955 and popANI > 0.99999 to each pairwise comparison, producing classification tables (e.g., 66 N. gonorrhoeae, 185 hypermutator E. coli pairs, H. pylori within-host pairs, S. rimosus library pairs). Concordant calls (same by both) indicate point-mutation-dominated divergence or clonality; discordant calls (high popANI, low APSS) flag structural-variation-enriched pairs, as seen for S. rimosus across 185 comparisons. Use up to 200 regions per pairwise comparison, which yields maximal sensitivity (98.6%) and specificity (97%). Recognize that thresholds are subjective and species-dependent: the human-gut APSS cutoff of 0.95 was an average of per-species cutoffs, so tailored cutoffs should be derived from a training dataset relevant to the research question. Where possible, prefer reporting continuous APSS/popANI values over binary strain labels, per the authors' recommendation.

## Related tools

- **SynTracker** (Computes APSS (average pairwise synteny scores) from genome synteny of homologous regions, used as the structural-variation axis of the classification) — https://github.com/leylabmpi/SynTracker
- **inStrain** (Computes popANI values for each isolate pair, used as the SNP-based axis of the same-strain classification (popANI > 0.99999 cutoff))
- **BLAST** (SynTracker builds a BLAST database of target genomes and searches 1-kbp central regions with minimal identity 97% and minimal query coverage 70% before APSS calculation)

## Examples

```
python syntracker.py -target isolate_genomes/ -ref reference_dir/ -out SynTracker_output/
```

## Evaluation signals

- Each classification table must contain all pairwise comparisons for its species (e.g., 66 rows for N. gonorrhoeae, 185 for hypermutator E. coli)
- Spearman rho = 0.985 between APSS and popANI over the 66 N. gonorrhoeae comparisons
- All 185 hypermutator E. coli pairs are called same strain by both tools (APSS > 0.955 and popANI > 0.99999)
- S. rimosus pairs show a wide APSS range despite clonal (same-strain) popANI, reproducing the structural-variation signature
- H. pylori within-host subpopulations split between the two tools as reported in Fig. 3c

## Limitations

- Strain delineation thresholds (inStrain popANI 99.999%, APSS 0.95/0.955) are subjective and species-dependent; the 0.95 cutoff was an average of per-species cutoffs and may not be optimal for all settings
- Classification of genomes into strains is inherently subjective; the authors recommend avoiding same/different classifications in favor of continuous genomic difference measures
- Small numbers of regions per pairwise comparison mostly result in lower accuracy; a trade-off exists between accuracy and sample size
- Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis

## Evidence

- [discussion] Combined cutoffs drive classification: "we set an APSS score cutoff of 0.95 to designate the same strain in the human gut"
- [discussion] inStrain popANI cutoff: "The widely used software inStrain uses a popANI cutoff of 99.999%, which is based on comparisons of strains in (1) mock communities"
- [discussion] 200 regions maximize accuracy: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [discussion] Thresholds are subjective: "Such thresholds are subjective and depend on the species being considered, as different taxa accumulate genomic differences at different rates"
- [abstract] Combined SNP + synteny detection: "SynTracker, combined with an SNP-based tool, detects strains enriched in either point mutations or structural changes"
