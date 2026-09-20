---
name: snp-based-popani-strain-comparison
description: Use when per-sample metagenomic assemblies of the same species are available
  and the goal is to distinguish within-species genomic diversity arising from point
  mutations versus structural differences.
when_to_use_negative:
- The input consists of unassembled reads or low-data contexts (phages, plasmids)
  where the synteny-based tool is preferred over SNP-based comparison
- The organism accumulates structural variation rather than SNPs, so a purely SNP-based
  popANI cutoff would misclassify same-strain pairs
- You need a universal same/different strain designation; the article warns such thresholds
  (e.g. popANI 99.999%, APSS 0.95) are subjective and species-dependent
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: inStrain
  role: Computes popANI for every pairwise comparison of each species from the same
    metagenomic assemblies, providing the SNP-based similarity ranking
- name: SynTracker
  role: Provides the synteny-based APSS comparison set that is intersected with the
    popANI comparisons so both tools score the same >12,000 sample pairs
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: Underlies SynTracker's region detection (97% minimal identity, 70% minimal
    query coverage) used to build the intersected comparison background
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:1f75478ec7dc0d5f92a46464760cf23c86b105d3d663acc52610bd09f61bc1b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - SynTracker
  - inStrain
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# snp-based-popani-strain-comparison

## Summary

Computes popANI-based (SNP/point-mutation) similarity for conspecific strain pairs from metagenomic assemblies using inStrain, then intersects the comparison set with SynTracker's APSS-based comparisons and uses top-5% enrichment (hypergeometric test) to classify species as hypermutators versus hyper-recombinants.

## When to use

Use when per-sample metagenomic assemblies of the same species are available and the goal is to distinguish within-species genomic diversity arising from point mutations versus structural differences.

## When NOT to use

- The input consists of unassembled reads or low-data contexts (phages, plasmids) where the synteny-based tool is preferred over SNP-based comparison
- The organism accumulates structural variation rather than SNPs, so a purely SNP-based popANI cutoff would misclassify same-strain pairs
- You need a universal same/different strain designation; the article warns such thresholds (e.g. popANI 99.999%, APSS 0.95) are subjective and species-dependent

## Inputs

- Per-sample metagenomic assemblies (one FASTA of assembled contigs per sample)
- Per-species reference genome directories (FASTA files, complete or multi-contig)
- SynTracker APSS output tables (e.g. avg_synteny_scores_all_regions.csv per genome and summary_output/) for the intersected comparison set

## Outputs

- Pairwise popANI values for every comparison of each species (inStrain output)
- Intersected SynTracker/inStrain comparison table per species (>12,000 shared comparisons)
- Per-species hypergeometric enrichment p-values for the top-5% similarity sets
- species_enrichment_results.csv classifying species as hypermutators or hyper-recombinants

## How to apply

Run inStrain on the per-sample metagenomic assemblies to compute popANI for every pairwise comparison of each species, yielding roughly 40,000 comparisons. Intersect these with SynTracker's APSS comparison set per species, retaining only the >12,000 comparisons shared by both tools and excluding sample pairs with fewer than n regions. For each tool, rank the shared comparisons by similarity and select the most similar 5% as the success set. For each species, perform a hypergeometric test (e.g. scipy.stats.hypergeom) asking whether SNP-similar (inStrain) or synteny-similar (SynTracker) comparisons are over-represented relative to the shared background. Classify enrichments: hypermutators (Phocaeicola vulgatus, Bacteroides fragilis, Alistipes putredinis) versus hyper-recombinants (Phocaeicola massiliensis, Streptococcus thermophilus, Streptococcus gallolyticus), as in Fig. 4a,b. Note that strain-delineation thresholds such as inStrain's popANI 99.999% are subjective and species-dependent; prefer continuous measures over same/different strain calls.

## Related tools

- **inStrain** (Computes popANI for every pairwise comparison of each species from the same metagenomic assemblies, providing the SNP-based similarity ranking)
- **SynTracker** (Provides the synteny-based APSS comparison set that is intersected with the popANI comparisons so both tools score the same >12,000 sample pairs) — https://github.com/leylabmpi/SynTracker
- **BLAST** (Underlies SynTracker's region detection (97% minimal identity, 70% minimal query coverage) used to build the intersected comparison background)

## Examples

```
from scipy.stats import hypergeom; p = hypergeom.sf(k - 1, M_total_shared, n_top5pct, N_species_hits)
```

## Evaluation signals

- Total comparison counts validate: ~40,000 for inStrain versus ~19,000 for SynTracker, with >12,000 comparisons shared by both tools
- Recovered hypermutator species set matches Fig. 4a: Phocaeicola vulgatus, Bacteroides fragilis, Alistipes putredinis
- Recovered hyper-recombinant species set matches Fig. 4b: Phocaeicola massiliensis, Streptococcus thermophilus, Streptococcus gallolyticus
- Sample pairs with fewer than n regions are excluded from downstream analysis before ranking and testing
- Combined single-species analysis recovers E. coli as a hypermutator, consistent with prior observations

## Limitations

- The popANI 99.999% cutoff used by inStrain is subjective and depends on the species being considered, as different taxa accumulate genomic differences at different rates
- The authors recommend avoiding same/different strain classifications because of inherent subjectivity, favoring continuous genomic difference measures instead
- A small number of regions per pairwise comparison mostly results in lower accuracy; a trade-off exists between accuracy and sample size

## Evidence

- [discussion] inStrain popANI cutoff and subjectivity: "The widely used software inStrain uses a popANI cutoff of 99.999%, which is based on comparisons of strains in (1) mock communities"
- [abstract] combined SNP- and synteny-based detection of evolutionary modes: "SynTracker, combined with an SNP-based tool, detects strains enriched in either point mutations or structural changes, providing insights into microbial evolution in situ."
- [discussion] E. coli hypermutator validation: "our combined analysis of single species revealed E. coli as a hypermutator, in line with previous observations"
- [discussion] species-dependent thresholds: "Such thresholds are subjective and depend on the species being considered, as different taxa accumulate genomic differences at different rates"
- [readme] excluding pairs with too few regions: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
