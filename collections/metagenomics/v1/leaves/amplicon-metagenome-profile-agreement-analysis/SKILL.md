---
name: amplicon-metagenome-profile-agreement-analysis
description: Use when paired amplicon (16S) and metagenomic profiles exist for the
  same mock or known communities and you need to test whether GCN correction (or another
  bias correction) brings the amplicon profile closer to the metagenomic or expected
  composition.
when_to_use_negative:
- Your study relies on taxonomies other than Greengenes (e.g. Silva) — CopyRighter's
  pre-computed GCN estimates cover the Greengenes taxonomy; Silva support was only
  anticipated in the discussion
- You need correction of GC percent bias, genome length bias, or fungal ITS variation
  — these are only hypothetical pending appropriate pre-computed estimates
- The input community file contains percentages rather than read counts, or taxa have
  altered taxonomic assignments
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: CopyRighter
  role: Corrects amplicon profiles for lineage-specific 16S rRNA gene copy number
    (phylogenetic-level and taxonomic-level correction) prior to agreement comparison
  repo: github.com/fangly/AmpliCopyrighter
- name: Grinder
  role: Simulates the matched 16S rRNA amplicon and shotgun metagenomic reads for
    the in silico low/medium/high richness mock communities
- name: Bio::Community
  role: Perl community-file library underlying CopyRighter input/output; bc_use_repr_id
    can replace arbitrary OTU IDs with Greengenes IDs for trait lookup
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:6cb19bd1fb3c6f70b4097d35e88e22383df52f23b20bcdf5fe748b90072393fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Grinder
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# amplicon-metagenome-profile-agreement-analysis

## Summary

Quantify how closely 16S rRNA gene amplicon microbial profiles agree with matched metagenomic (shotgun) profiles of the same mock communities, before and after GCN correction with CopyRighter. Agreement serves as validation evidence that gene copy number correction makes amplicon-based relative abundances more accurate.

## When to use

Use when paired amplicon (16S) and metagenomic profiles exist for the same mock or known communities and you need to test whether GCN correction (or another bias correction) brings the amplicon profile closer to the metagenomic or expected composition.

## When NOT to use

- Your study relies on taxonomies other than Greengenes (e.g. Silva) — CopyRighter's pre-computed GCN estimates cover the Greengenes taxonomy; Silva support was only anticipated in the discussion
- You need correction of GC percent bias, genome length bias, or fungal ITS variation — these are only hypothetical pending appropriate pre-computed estimates
- The input community file contains percentages rather than read counts, or taxa have altered taxonomic assignments

## Inputs

- 16S rRNA gene amplicon profile (OTU table in biom, QIIME, GAAS, Unifrac, or generic tabular format) with read counts and unaltered taxonomic assignments
- Matched shotgun metagenomic abundance profile for the same community
- Known expected community composition (in silico or in vitro mock)
- Pre-computed Greengenes GCN estimate files (OTU-ID and taxonomic-string level, tab-delimited two-column format)
- Total abundance file (e.g. 16S rRNA qPCR numbers), optional for absolute abundance

## Outputs

- GCN-corrected amplicon community files with relative abundances in percent (default out_copyrighted.txt)
- Agreement metrics between amplicon and metagenomic profiles, with and without correction (boxplots/statistical comparison across richness tiers)
- Optionally corrected total abundance (out_copyrighted_total.tsv) and combined absolute abundance files (out_copyrighted_combined.txt)
- Verbose trait-value assignment log (-v) showing which GCN value each taxon received

## How to apply

Simulate or obtain matched datasets for each community: 16S rRNA gene amplicon reads and shotgun metagenomic reads generated with Grinder for in silico mock communities spanning low, medium and high richness tiers (10 to 1,000 species). Build OTU/taxonomic abundance profiles from amplicon reads against the Greengenes reference, ensuring taxa carry unaltered Greengenes taxonomic strings (e.g. 'k__Bacteria; p__; ...; s__') and the table contains read counts, not percentages. Run CopyRighter on each amplicon profile under both phylogenetic-level and taxonomic-level correction using the pre-computed Greengenes GCN files (GCN spans 1-15 in Bacteria, up to 5 in Archaea), then compute agreement metrics of corrected and uncorrected amplicon profiles against both the expected community compositions and the simulated metagenomic profiles. Rationale: GCN skews amplicon profiles toward high-copy taxa, so correction should increase amplicon-vs-metagenome agreement; the article's validation found corrected profiles significantly closer, with phylogenetic-level correction generally slightly more accurate than taxonomic-level correction.

## Related tools

- **CopyRighter** (Corrects amplicon profiles for lineage-specific 16S rRNA gene copy number (phylogenetic-level and taxonomic-level correction) prior to agreement comparison) — github.com/fangly/AmpliCopyrighter
- **Grinder** (Simulates the matched 16S rRNA amplicon and shotgun metagenomic reads for the in silico low/medium/high richness mock communities)
- **Bio::Community** (Perl community-file library underlying CopyRighter input/output; bc_use_repr_id can replace arbitrary OTU IDs with Greengenes IDs for trait lookup)

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -l desc -v
```

## Evaluation signals

- GCN-corrected amplicon profiles show higher agreement with the expected compositions than uncorrected profiles, across the low, medium and high richness tiers
- Corrected amplicon profiles show higher agreement with the simulated metagenomic profiles than uncorrected ones (boxplot comparison of agreement values)
- Phylogenetic-level correction agreement is at least as good as, and generally slightly better than, taxonomic-level correction
- Input tables contain raw read counts (not percentages) and taxa retain unaltered Greengenes taxonomic strings; verbose mode (-v) confirms taxa received sensible GCN values spanning the reported range (1-15 in Bacteria, up to 5 in Archaea)
- Corrected relative abundances are expressed in percent and output format matches the input community-file format

## Limitations

- CopyRighter's pre-computed GCN estimates are based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy; other taxonomies (e.g. Silva) are not yet covered
- Correction effects are non-trivial to predict for communities with tens to thousands of species
- Currently limited to 16S rRNA GCN; correction of GC percent bias, genome length bias, and fungal ITS variation awaits appropriate pre-computed estimates
- No version information stated beyond the README's copyrighter version 0.46

## Evidence

- [abstract] abstract: "Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance and improves the agreement between"
- [intro] intro: "GCN variation spans over an order of magnitude, from 1 to 15 in Bacteria, but only up to 5 in Archaea"
- [discussion] discussion: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction, to obtain the highest accuracy possible."
- [supplementary] supplementary: "Boxplot of the agreement between in silico 16S rRNA gene amplicon and metagenomic mock datasets with and without Copyrighter correction."
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [methods] methods: "To install CopyRighter globally on your system, run the following commands"
