---
name: hypergeometric-enrichment-testing
description: Use when two independent strain-similarity metrics (e.g. APSS from SynTracker
  and popANI from inStrain) have been computed on the same sample pairs and you must
  identify species whose within-species diversity arises preferentially from point
  mutations versus structural differences.
when_to_use_negative:
- When the two tools' comparison sets do not overlap substantially (fewer than the
  ~12,000 shared comparisons), since enrichment is defined only against the shared
  background
- When strain similarity is measured by a single metric only — the hypermutator vs
  hyper-recombinant contrast requires both a synteny-based (APSS) and an SNP-based
  (popANI) top-5% set
- When comparing isolates rather than metagenomic sample pairs, where the comparison
  background and 5% selection logic do not apply as designed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Provides synteny-based APSS similarity scores whose top-5% comparisons define
    the structural-difference (hyper-recombinant) success set
  repo: https://github.com/leylabmpi/SynTracker
- name: inStrain
  role: Provides popANI values whose top-5% comparisons define the point-mutation
    (hypermutator) success set
- name: scipy.stats.hypergeom
  role: Performs the per-species hypergeometric enrichment test of tool-similar comparisons
    against the shared background
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:cd09cd84fcc2e5078c60a7ac5b33830125d7fddacb45715a0c8073ec4ea46e96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - SynTracker
  - inStrain
  - scipy.stats.hypergeom
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hypergeometric-enrichment-testing

## Summary

A statistical classification step in which, for each microbial species, a hypergeometric test (e.g. scipy.stats.hypergeom) asks whether pairwise strain comparisons that are most similar by SynTracker (synteny-based APSS) or by inStrain (popANI) are over-represented in the shared comparison background, distinguishing hypermutator from hyper-recombinant species.

## When to use

Use when two independent strain-similarity metrics (e.g. APSS from SynTracker and popANI from inStrain) have been computed on the same sample pairs and you must identify species whose within-species diversity arises preferentially from point mutations versus structural differences.

## When NOT to use

- When the two tools' comparison sets do not overlap substantially (fewer than the ~12,000 shared comparisons), since enrichment is defined only against the shared background
- When strain similarity is measured by a single metric only — the hypermutator vs hyper-recombinant contrast requires both a synteny-based (APSS) and an SNP-based (popANI) top-5% set
- When comparing isolates rather than metagenomic sample pairs, where the comparison background and 5% selection logic do not apply as designed

## Inputs

- SynTracker APSS table (avg_synteny_scores_all_regions.csv per genome and in summary_output/) for pairwise sample comparisons
- inStrain popANI values for every pairwise comparison of each species
- Per-species mapping of sample pairs to species (intersected comparison sets, >12,000 shared comparisons)
- Per-sample metagenomic assemblies from three-country gut cohorts (mother-infant pairs and non-native-born subjects filtered out)

## Outputs

- species_enrichment_results.csv with per-species hypergeometric enrichment p-values and classification
- Per-species labels: hypermutator (diversity from point mutations) vs hyper-recombinant (diversity from structural differences)
- Figure-style enrichment plot equivalent to Fig. 4a,b

## How to apply

First intersect the SynTracker and inStrain comparison sets per species, retaining only the >12,000 comparisons shared by both tools and excluding sample pairs with fewer than n regions. For each tool, rank the shared comparisons by similarity and take the most similar 5% as the success set. For each species, run a hypergeometric test (e.g. scipy.stats.hypergeom) asking whether SynTracker-similar or inStrain-similar comparisons are over-represented relative to the shared background. Classify species by which tool's top-5% set they enrich into: hypermutators (e.g. Phocaeicola vulgatus, Bacteroides fragilis, Alistipes putredinis) versus hyper-recombinants (e.g. Phocaeicola massiliensis, Streptococcus thermophilus, Streptococcus gallolyticus), and write species_enrichment_results.csv.

## Related tools

- **SynTracker** (Provides synteny-based APSS similarity scores whose top-5% comparisons define the structural-difference (hyper-recombinant) success set) — https://github.com/leylabmpi/SynTracker
- **inStrain** (Provides popANI values whose top-5% comparisons define the point-mutation (hypermutator) success set)
- **scipy.stats.hypergeom** (Performs the per-species hypergeometric enrichment test of tool-similar comparisons against the shared background)

## Examples

```
from scipy.stats import hypergeom; p = hypergeom.sf(k-1, N_shared, n_top5pct_tool, n_species_comparisons)
```

## Evaluation signals

- Total shared comparison counts match expectations: ~40,000 for inStrain vs ~19,000 for SynTracker
- Recovered hypermutator set includes Phocaeicola vulgatus, Bacteroides fragilis and Alistipes putredinis, and hyper-recombinant set includes Phocaeicola massiliensis, Streptococcus thermophilus and Streptococcus gallolyticus, matching Fig. 4a,b
- Only comparisons present in BOTH tools' outputs (intersected, >12,000 per the workflow) enter the background; pairs with fewer than n regions are excluded
- Each species receives exactly one enrichment direction (SynTracker-similar or inStrain-similar) or neither, based on the hypergeometric p-value against the shared background
- Success sets are exactly the most similar 5% of the shared comparisons per tool

## Limitations

- Strain delineation thresholds (e.g. inStrain popANI 99.999%, APSS 0.95) are subjective and species-dependent, as different taxa accumulate genomic differences at different rates
- The APSS 0.95 cutoff was based on averaging different per-species cutoffs and may not be optimal for all settings; researchers must set context-appropriate cutoffs
- SynTracker's APSS depends on region subsampling; without a fixed seed, average synteny scores may change between runs due to the subsampling (default seed=1 enables reproducibility)

## Evidence

- [abstract] SynTracker, combined with an SNP-based tool, detects strains enriched in either point mutations or structural changes, providing insights into microbial evolution in situ.: "SynTracker, combined with an SNP-based tool, detects strains enriched in either point mutations or structural changes, providing insights into microbial evolution in situ."
- [discussion] Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis.: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
- [discussion] we set an APSS score cutoff of 0.95 to designate the same strain in the human gut: "we set an APSS score cutoff of 0.95 to designate the same strain in the human gut"
- [readme] By default, a seed=1 is set to enable reproducibility between different runs.: "This means that the average synteny scores may change between SynTracker runs due to the subsampling. By default, a seed=1 is set to enable reproducibility between different runs."
