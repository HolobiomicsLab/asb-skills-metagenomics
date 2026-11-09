---
name: cross-tool-comparison-intersection
description: Use when per-sample metagenomic assemblies have been profiled by both
  SynTracker (APSS) and inStrain (popANI), and the goal is to detect species whose
  within-species diversity is preferentially point-mutation- or structurally-driven.
when_to_use_negative:
- Tracking phages or plasmids, where SynTracker alone achieved a significantly higher
  AUC than inStrain and an SNP-based comparison set is unnecessary
- Classifying pairs as same/different strain for delineation, which the authors recommend
  avoiding in favor of continuous genomic difference measures because thresholds like
  APSS 0.95 or popANI 99.999% are subjective and species-dependent
- Datasets where only one tool was run or where the two comparison sets share too
  few sample pairs to define a meaningful intersected background
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: SynTracker
  role: Produces synteny-based APSS values for pairwise metagenomic assembly comparisons;
    its top-similarity set flags structurally divergent (hyper-recombinant) species
  repo: https://github.com/leylabmpi/SynTracker
- name: inStrain
  role: Provides SNP-based popANI for the same pairwise comparisons; its top-similarity
    set flags hypermutator species
- name: BLAST
  role: Underlying search engine of SynTracker, run with 97% minimal identity and
    70% minimal query coverage to locate homologous regions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:21b98824927c3067c47cbf8caccd4268bd2d44a01d323b2730950c8f2b66cf2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - SynTracker
  - inStrain
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cross-tool-comparison-intersection

## Summary

Intersects strain-comparison sets produced by a synteny-based tool (SynTracker APSS) and an SNP-based tool (inStrain popANI) per species, then uses hypergeometric tests on each tool's most-similar 5% comparisons to classify species as hypermutators (point mutations) or hyper-recombinants (structural differences).

## When to use

Use when per-sample metagenomic assemblies have been profiled by both SynTracker (APSS) and inStrain (popANI), and the goal is to detect species whose within-species diversity is preferentially point-mutation- or structurally-driven.

## When NOT to use

- Tracking phages or plasmids, where SynTracker alone achieved a significantly higher AUC than inStrain and an SNP-based comparison set is unnecessary
- Classifying pairs as same/different strain for delineation, which the authors recommend avoiding in favor of continuous genomic difference measures because thresholds like APSS 0.95 or popANI 99.999% are subjective and species-dependent
- Datasets where only one tool was run or where the two comparison sets share too few sample pairs to define a meaningful intersected background

## Inputs

- SynTracker per-genome and summary APSS tables (avg_synteny_scores_all_regions.csv)
- inStrain popANI pairwise comparison tables per species
- Per-sample metagenomic assemblies (target directory) and per-species reference genome directories
- Shared sample/species mapping linking SynTracker and inStrain comparisons

## Outputs

- species_enrichment_results.csv with hypergeometric p-values and classification per species
- Set of hypermutator species (e.g., Phocaeicola vulgatus, Bacteroides fragilis, Alistipes putredinis)
- Set of hyper-recombinant species (e.g., Phocaeicola massiliensis, Streptococcus thermophilus, Streptococcus gallolyticus)

## How to apply

Run SynTracker in 'new' mode on per-sample assemblies with default parameters (region length 5000, BLAST identity 97, coverage 70), collecting APSS values from avg_synteny_scores_all_regions.csv, and run inStrain on the same assemblies to obtain popANI for every pairwise comparison of each species. Restrict both tools to the comparison sets they share per species (in the paper this retained >12,000 comparisons out of ~40,000 inStrain and ~19,000 SynTracker comparisons) and exclude sample pairs with fewer than n regions, as SynTracker already excludes pairs lacking sufficient regions. For each tool independently, rank the shared comparisons by similarity and take the most similar 5% as the success set. For each species, use scipy.stats.hypergeom to test whether SynTracker-similar or inStrain-similar comparisons are over-represented relative to the shared background. Species enriched in SynTracker's top 5% (low APSS sensitivity to SNPs, sensitivity to structural change) are hyper-recombinants, while species enriched in inStrain's top 5% (SNP-sensitive) are hypermutators; write species_enrichment_results.csv.

## Related tools

- **SynTracker** (Produces synteny-based APSS values for pairwise metagenomic assembly comparisons; its top-similarity set flags structurally divergent (hyper-recombinant) species) — https://github.com/leylabmpi/SynTracker
- **inStrain** (Provides SNP-based popANI for the same pairwise comparisons; its top-similarity set flags hypermutator species)
- **BLAST** (Underlying search engine of SynTracker, run with 97% minimal identity and 70% minimal query coverage to locate homologous regions)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Total comparison counts match expectations (~40,000 for inStrain vs ~19,000 for SynTracker) with >12,000 comparisons shared by both tools
- No comparison in either top-5% success set involves sample pairs excluded for having fewer than n regions
- Hypergeometric enrichment calls reproduce the Fig. 4a,b partition: hypermutators P. vulgatus, B. fragilis, A. putredinis; hyper-recombinants P. massiliensis, S. thermophilus, S. gallolyticus
- E. coli is recovered as a hypermutator in the combined single-species analysis, consistent with prior observations
- species_enrichment_results.csv assigns every tested species to exactly one class or non-enriched, with finite hypergeometric p-values

## Limitations

- Strain delineation thresholds are subjective and species-dependent, as different taxa accumulate genomic differences at different rates; the APSS 0.95 cutoff was based on averaging per-species cutoffs and may not be optimal for all settings
- A small number of regions per pairwise comparison mostly results in lower accuracy; there is a trade-off between accuracy and sample size
- SynTracker has low sensitivity to SNPs, so it cannot by itself resolve point-mutation-driven diversity; the hypermutator call depends entirely on the SNP-based tool's rankings

## Evidence

- [abstract] Combined SNP and synteny detection: "SynTracker, combined with an SNP-based tool, detects strains enriched in either point mutations or structural changes, providing insights into microbial evolution in situ."
- [discussion] E. coli hypermutator: "our combined analysis of single species revealed E. coli as a hypermutator, in line with previous observations"
- [discussion] SynTracker vs SNP tools complementarity: "SynTracker outperformed MIDAS35 and StrainPhlAn36 and achieved a slightly better performance score than InStrain28 in terms of accuracy"
- [readme] All-regions APSS output used for intersection: "The table containing the APSS using all regions (without subsampling) is called `avg_synteny_scores_all_regions.csv`."
- [methods] Exclusion of sparse region pairs: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
