---
name: phylogenetic-vs-taxonomic-abundance-correction
description: Use when 16S rRNA amplicon abundance profiles need GCN bias correction
  and a decision must be made between phylogenetic-level and taxonomic-level CopyRighter
  correction, or when validating corrected profiles against known community compositions.
when_to_use_negative:
- The taxonomy is not Greengenes — CopyRighter does not yet cover taxonomies such
  as Silva; extension is only anticipated
- The input table contains percentages rather than read counts, or taxonomic strings
  have been altered, since CopyRighter requires counts and unaltered assignments
- The bias of interest is GC percent, genome length, or fungal ITS variation without
  appropriate pre-computed trait estimates — CopyRighter currently provides 16S rRNA
  GCN and genome length data only
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3174
tools:
- name: CopyRighter
  role: Applies lineage-specific GCN correction to amplicon profiles at either phylogenetic
    or taxonomic level
  repo: github.com/fangly/AmpliCopyrighter
- name: Grinder
  role: Simulates 16S rRNA amplicon and shotgun metagenomic reads for the 90 in silico
    mock communities used in validation
- name: Bio::Community
  role: Perl library dependency handling community file I/O; bc_use_repr_id can replace
    OTU IDs with Greengenes IDs
- name: Greengenes
  role: Reference taxonomy/database for which phylogenetically-based GCN estimates
    for all OTUs and taxa are provided
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:79ca9acb872df73fd42a3c52b85fe7041b8c268074895179145dec2d4d29647b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Greengenes
  - Grinder
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# phylogenetic-vs-taxonomic-abundance-correction

## Summary

Compares CopyRighter's phylogenetic-level versus taxonomic-level 16S rRNA gene copy number (GCN) correction of amplicon microbial profiles, validating against expected in silico mock community compositions. Phylogenetic correction is generally slightly more accurate.

## When to use

Use when 16S rRNA amplicon abundance profiles need GCN bias correction and a decision must be made between phylogenetic-level and taxonomic-level CopyRighter correction, or when validating corrected profiles against known community compositions.

## When NOT to use

- The taxonomy is not Greengenes — CopyRighter does not yet cover taxonomies such as Silva; extension is only anticipated
- The input table contains percentages rather than read counts, or taxonomic strings have been altered, since CopyRighter requires counts and unaltered assignments
- The bias of interest is GC percent, genome length, or fungal ITS variation without appropriate pre-computed trait estimates — CopyRighter currently provides 16S rRNA GCN and genome length data only

## Inputs

- 16S rRNA amplicon OTU/taxonomic abundance table with read counts (biom, QIIME, GAAS, Unifrac, or generic tabular format), taxa with unaltered Greengenes taxonomic assignments
- Pre-computed Greengenes GCN estimate files (phylogenetic and taxonomic variants, distributed with CopyRighter or via COPYRIGHTER_DB)
- Optional: total 16S rRNA qPCR abundance file (tab-delimited community name + total abundance)
- Known expected community composition (for mock-community validation)

## Outputs

- GCN-corrected community file(s) with relative abundance in percent, same format as input (default out_copyrighted.txt)
- Corrected qPCR total abundance (out_copyrighted_total.tsv) and absolute abundance per taxon (out_copyrighted_combined.txt) when -t is used
- Comparison metrics quantifying agreement of corrected vs uncorrected profiles against expected compositions and metagenomic profiles
- Average trait (GCN) value per community reported on standard output

## How to apply

Build an OTU/taxonomic abundance profile from 16S rRNA amplicon reads against the Greengenes reference, ensuring the input community file contains read counts (not percentages) with unaltered taxonomic assignments (e.g., Greengenes 2012/10 strings). Run CopyRighter twice on each profile: once with phylogenetic-level correction and once with taxonomic-level correction, using the pre-computed Greengenes GCN estimate files (GCN spans 1-15 in Bacteria, up to 5 in Archaea). Compare both corrected profiles against the uncorrected profile and, where ground truth exists (e.g., Grinder-simulated in silico mock communities spanning 10 to 1,000 species), quantify agreement with the expected composition and with simulated metagenomic profiles. Prefer phylogenetic-level correction, as the article recommends it over taxonomic-level correction to obtain the highest accuracy; validate by confirming corrected profiles show higher agreement than uncorrected ones.

## Related tools

- **CopyRighter** (Applies lineage-specific GCN correction to amplicon profiles at either phylogenetic or taxonomic level) — github.com/fangly/AmpliCopyrighter
- **Grinder** (Simulates 16S rRNA amplicon and shotgun metagenomic reads for the 90 in silico mock communities used in validation)
- **Bio::Community** (Perl library dependency handling community file I/O; bc_use_repr_id can replace OTU IDs with Greengenes IDs)
- **Greengenes** (Reference taxonomy/database for which phylogenetically-based GCN estimates for all OTUs and taxa are provided)

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -v
```

## Evaluation signals

- For the 90 in silico mock communities (10-1,000 species), corrected amplicon profiles must show significantly higher agreement with the expected compositions than uncorrected profiles
- Phylogenetic-level correction should generally be slightly more accurate than taxonomic-level correction
- Corrected profiles should also improve agreement between 16S amplicon and simulated metagenomic mock datasets
- Run with -v and verify trait value assignments — confirm taxa are matched by the intended lookup mode ('desc' vs 'id') with sensible GCN values (1-15 in Bacteria, up to 5 in Archaea)
- Input validation: the input file must contain read counts, not percentages, and unaltered taxonomic strings; outputs are expressed in percent relative abundance

## Limitations

- CopyRighter correction is currently limited to the 16S rRNA gene; correcting GC percent bias, genome length bias, or fungal ITS variation requires appropriate pre-comulated estimates not yet distributed
- GCN coverage is tied to Greengenes; other taxonomies (e.g., Silva) are not yet supported
- With communities of tens to thousands of species, the effects of correction are non-trivial to predict
- No version stated for general discussion of the tool beyond README version 0.46; GCN of uncultured organisms is inferred from close relatives and may be inaccurate without a closely related sequenced relative

## Evidence

- [abstract] abstract: "Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance and improves the agreement between"
- [discussion] discussion: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction, to obtain the highest accuracy possible."
- [supplementary] supplementary: "Boxplot of the agreement between in silico 16S rRNA gene amplicon and metagenomic mock datasets with and without Copyrighter correction."
- [intro] intro: "GCN variation spans over an order of magnitude, from 1 to 15 in Bacteria, but only up to 5 in Archaea [5]."
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [discussion] discussion (readme-adjacent scope): "We anticipate that the CopyRighter methodology and software described here will be extended to cover GCN in other taxonomies (for example, Silva [65])"
