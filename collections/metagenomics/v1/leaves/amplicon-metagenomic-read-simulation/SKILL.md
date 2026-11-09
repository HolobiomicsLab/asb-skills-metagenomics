---
name: amplicon-metagenomic-read-simulation
description: Use when a validation study needs communities of known composition to
  benchmark amplicon profiling or GCN correction — e.g. to test whether CopyRighter-corrected
  16S profiles match expected and metagenomic profiles.
when_to_use_negative:
- Real environmental sequencing data — simulation is only useful when the true community
  composition is known (mock communities); for real samples, compare amplicon vs.
  metagenomic profiles directly instead
- GCN correction itself — this skill is the upstream read-simulation step that produces
  benchmark inputs; correction is done downstream with CopyRighter
- Studies targeting non-16S markers (e.g. fungal ITS) — the Greengenes-based simulation
  and correction framework described here covers Bacteria (GCN 1–15) and Archaea (up
  to 5) only
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Grinder
  role: Simulates 16S rRNA gene amplicon reads and shotgun metagenomic reads for every
    in silico mock community
- name: Bio::Community
  role: Perl library used to build and manipulate the community abundance profiles
    produced from the simulated amplicon reads
- name: CopyRighter
  role: 'Downstream consumer of the simulated amplicon profiles: corrected vs. uncorrected
    profiles are compared against expected compositions and metagenomic profiles to
    validate the simulation benchmark'
  repo: github.com/fangly/AmpliCopyrighter
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:cd556acadd7dc7bbf46b85591f3176d35231e5b78c5b2aa542357b9f176c23e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Grinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# amplicon-metagenomic-read-simulation

## Summary

Generate in silico mock communities (low/medium/high richness, 10–1,000 species) and simulate both 16S rRNA gene amplicon reads and shotgun metagenomic reads with Grinder, producing paired ground-truth datasets to benchmark GCN-corrected amplicon profiles against known compositions and metagenomic references.

## When to use

Use when a validation study needs communities of known composition to benchmark amplicon profiling or GCN correction — e.g. to test whether CopyRighter-corrected 16S profiles match expected and metagenomic profiles.

## When NOT to use

- Real environmental sequencing data — simulation is only useful when the true community composition is known (mock communities); for real samples, compare amplicon vs. metagenomic profiles directly instead
- GCN correction itself — this skill is the upstream read-simulation step that produces benchmark inputs; correction is done downstream with CopyRighter
- Studies targeting non-16S markers (e.g. fungal ITS) — the Greengenes-based simulation and correction framework described here covers Bacteria (GCN 1–15) and Archaea (up to 5) only

## Inputs

- In silico mock community definitions (species lists with expected relative abundances), in low/medium/high richness tiers of 10–1,000 species
- Greengenes reference database for OTU/taxonomic assignment of simulated amplicon reads

## Outputs

- Simulated 16S rRNA gene amplicon read datasets per mock community (Grinder)
- Simulated shotgun metagenomic read datasets per mock community (Grinder)
- OTU/taxonomic abundance profiles built from simulated amplicon reads against Greengenes (read counts, unaltered taxonomic strings)
- Rank-abundance plots characterizing the low/medium/high richness tiers

## How to apply

Define in silico mock communities in three richness tiers — low, medium and high — spanning 10 to 1,000 species each, with known expected compositions. Use Grinder to simulate 16S rRNA gene amplicon reads and, separately, shotgun metagenomic reads for every community. Build OTU/taxonomic abundance profiles from the simulated amplicon reads against the Greengenes reference (e.g. Greengenes 2012/10 format), keeping read counts rather than percentages so profiles remain compatible with downstream correction. Validate by comparing corrected and uncorrected amplicon profiles against the expected community compositions and the simulated metagenomic profiles; the design is powerful precisely because the true composition is known by construction. Rank-abundance plots of each tier summarize the simulated community structure.

## Related tools

- **Grinder** (Simulates 16S rRNA gene amplicon reads and shotgun metagenomic reads for every in silico mock community)
- **Bio::Community** (Perl library used to build and manipulate the community abundance profiles produced from the simulated amplicon reads)
- **CopyRighter** (Downstream consumer of the simulated amplicon profiles: corrected vs. uncorrected profiles are compared against expected compositions and metagenomic profiles to validate the simulation benchmark) — github.com/fangly/AmpliCopyrighter

## Evaluation signals

- Corrected amplicon profiles show higher agreement with the expected community compositions than uncorrected profiles, matching the reported in silico validation result
- CopyRighter correction improves agreement between the in silico 16S rRNA amplicon and metagenomic mock datasets (supplementary boxplot comparison)
- Rank-abundance distributions of the simulated communities match the intended low, medium and high richness tiers (10–1,000 species)
- Amplicon profile inputs contain read counts (not percentages) with unaltered taxonomic assignments in Greengenes 2012/10 format, so CopyRighter can consume them
- Simulated GCN regime stays within the biological span the correction covers: 1–15 copies in Bacteria, up to 5 in Archaea

## Limitations

- With communities of tens to thousands of species, prediction of the effects of GCN correction is non-trivial, so simulated benchmark results may not transfer directly to complex real communities
- The Greengenes-based GCN estimates used with the simulated profiles are lineage-specific; extending to other taxonomies (e.g. Silva) was only anticipated, not available
- Simulation and correction are limited to the 16S rRNA gene; GC percent bias, genome length bias and fungal ITS variation would require different pre-computed estimates
- No Grinder or CopyRighter version is stated in the article text (README documents copyrighter version 0.46)

## Evidence

- [supplementary] Rank-abundance plot of the low, medium and high richness in silico mock communities generated with Grinder.: "Rank-abundance plot of the low, medium and high richness in silico mock communities generated with Grinder."
- [supplementary] Boxplot of the agreement between in silico 16S rRNA gene amplicon and metagenomic mock datasets with and without Copyrighter correction.: "agreement between in silico 16S rRNA gene amplicon and metagenomic mock datasets with and without Copyrighter correction"
- [intro] GCN variation spans over an order of magnitude, from 1 to 15 in Bacteria, but only up to 5 in Archaea: "GCN variation spans over an order of magnitude, from 1 to 15 in Bacteria, but only up to 5 in Archaea"
- [abstract] Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance: "Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance"
- [readme] README input-format requirement: read counts, not percentages, with unaltered taxonomic assignments.: [evidence span withheld — anchor status: external_doc]
