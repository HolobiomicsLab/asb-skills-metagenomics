---
name: insilico-mock-community-design
description: Use when validating an amplicon profile correction method (e.g., GCN
  correction) and you need simulated communities with known ground truth, plus matched
  amplicon and metagenomic read datasets from the same underlying composition.
when_to_use_negative:
- Your profiles come from real environmental or host-associated samples with no known
  expected composition — no ground truth exists to score agreement against
- Your question concerns non-16S marker genes or fungal ITS variation, for which CopyRighter
  has no pre-computed trait estimates (only hypothetical pending appropriate estimates)
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: Grinder
  role: Simulates 16S rRNA amplicon and shotgun metagenomic reads for each in silico
    mock community, including generating the rank-abundance structure of low/medium/high
    richness communities
- name: CopyRighter
  role: Applies phylogenetic-level and taxonomic-level GCN correction to the simulated
    amplicon profiles being validated
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl library underlying CopyRighter's community file handling (biom, QIIME,
    generic tabular formats) during correction of the mock-community profiles
  repo: github.com/fangly/AmpliCopyrighter
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:97392ccc897d6b5429977df7931d3545ac8650be0e9d26ec3f87a5440c1ba5ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Grinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# insilico-mock-community-design

## Summary

Design and simulate in silico mock communities spanning defined richness tiers (10 to 1,000 species), generating 16S rRNA gene amplicon and shotgun metagenomic reads with Grinder so that gene copy number (GCN) correction methods like CopyRighter can be validated against known expected community compositions.

## When to use

Use when validating an amplicon profile correction method (e.g., GCN correction) and you need simulated communities with known ground truth, plus matched amplicon and metagenomic read datasets from the same underlying composition.

## When NOT to use

- Your profiles come from real environmental or host-associated samples with no known expected composition — no ground truth exists to score agreement against
- Your question concerns non-16S marker genes or fungal ITS variation, for which CopyRighter has no pre-computed trait estimates (only hypothetical pending appropriate estimates)

## Inputs

- Reference taxonomy/sequence collection (Greengenes) from which to sample species
- Mock community definitions per richness tier (species membership and expected relative abundances), 10 to 1,000 species per community
- CopyRighter pre-computed Greengenes GCN estimate files (phylogenetic-level and taxonomic-level)

## Outputs

- Simulated 16S rRNA gene amplicon read sets per mock community (Grinder)
- Simulated shotgun metagenomic read sets per mock community (Grinder)
- OTU/taxonomic abundance tables derived from amplicon reads against Greengenes
- CopyRighter-corrected community files (phylogenetic-level and taxonomic-level)
- Agreement statistics comparing corrected vs uncorrected profiles against expected compositions and metagenomic profiles (e.g., rank-abundance plots, agreement boxplots)

## How to apply

Define mock communities in three richness tiers (low, medium, high) spanning 10 to 1,000 species each — the CopyRighter paper used 90 such communities. For every community, simulate 16S rRNA gene amplicon reads and shotgun metagenomic reads with Grinder, drawing from Greengenes reference taxa so the true expected composition is known by construction. Build OTU/taxonomic abundance profiles from the simulated amplicon reads against Greengenes, then feed each profile to the correction tool (CopyRighter, using pre-computed Greengenes GCN estimate files that span 1-15 in Bacteria and up to 5 in Archaea) under both phylogenetic-level and taxonomic-level correction. Rationale: because the ground truth is known, any difference between corrected/uncorrected amplicon profiles and the expected compositions (or the simulated metagenomic profiles) directly quantifies correction accuracy. Validation passes when corrected profiles show higher agreement with the expected compositions than uncorrected ones, mirroring the reported result where GCN-corrected profiles were significantly closer to expected profiles.

## Related tools

- **Grinder** (Simulates 16S rRNA amplicon and shotgun metagenomic reads for each in silico mock community, including generating the rank-abundance structure of low/medium/high richness communities)
- **CopyRighter** (Applies phylogenetic-level and taxonomic-level GCN correction to the simulated amplicon profiles being validated) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl library underlying CopyRighter's community file handling (biom, QIIME, generic tabular formats) during correction of the mock-community profiles) — github.com/fangly/AmpliCopyrighter

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic
```

## Evaluation signals

- Across all 90 communities, corrected amplicon profiles must show significantly higher agreement with the expected compositions than uncorrected profiles
- Corrected profiles must also show improved agreement with the simulated metagenomic profiles (as shown in agreement boxplots of in silico 16S amplicon vs metagenomic mock datasets)
- Phylogenetic-level correction should generally be at least as accurate as taxonomic-level correction
- Richness tiers must cover the full design range (10 to 1,000 species), and rank-abundance plots should confirm low/medium/high structure per tier
- Input community files must contain read counts (not percentages) with unaltered taxonomic assignments for correction to be valid

## Limitations

- Correction effects are non-trivial to predict for communities with tens to thousands of species, so in silico results may not transfer simply to complex real samples
- GCN estimates used span 1-15 in Bacteria and up to 5 in Archaea and are tied to the Greengenes database; other taxonomies (e.g., Silva) are not yet covered
- CopyRighter is currently limited to 16S rRNA gene copy number (and genome length for metagenomes); GC percent bias and fungal ITS variation are only hypothetical pending appropriate pre-computed estimates

## Evidence

- [supplementary] rank-abundance structure: "Rank-abundance plot of the low, medium and high richness in silico mock communities generated with Grinder."
- [abstract] validation finding: "Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance"
- [intro] correction setup: "GCN variation spans over an order of magnitude, from 1 to 15 in Bacteria, but only up to 5 in Archaea"
- [readme] input requirements: [evidence span withheld — anchor status: external_doc]
- [discussion] recommendation: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction"
