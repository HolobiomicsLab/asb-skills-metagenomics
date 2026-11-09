---
name: phylum-relative-abundance-computation
description: Use when a 16S rRNA amplicon OTU table (biom/QIIME format with read counts)
  must be GCN-corrected and aggregated to phylum level to report median relative abundances
  and phylum shifts (e.g. Bacteroidetes, Firmicutes) across a cohort.
when_to_use_negative:
- The input community file contains percentages instead of raw read counts, or taxa
  have been re-formatted from their original Greengenes taxonomic strings (CopyRighter
  requires counts and UNALTERED assignments)
- The taxonomy is not Greengenes (e.g. Silva); CopyRighter's distributed GCN estimates
  cover the Greengenes taxonomy and extension to other taxonomies is only anticipated,
  not available
- The goal is enterotype/dominant-genus clustering rather than phylum-level abundance
  summary, or the analysis requires absolute (qPCR-combined) abundances without a
  total-abundance file being supplied via -t
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CopyRighter
  role: Corrects OTU relative abundances by weighting read counts by the inverse lineage-specific
    16S rRNA gene copy number before phylum aggregation
  repo: https://github.com/fangly/AmpliCopyrighter
- name: Perl >= 5.10.0
  role: Runtime environment in which CopyRighter is installed and executed
- name: Bio::Community
  role: Perl dependency of CopyRighter for community file handling (e.g. bc_use_repr_id
    for OTU ID lookup)
- name: Greengenes database
  role: Taxonomy/reference for the phylogenetically-based GCN estimate files for OTUs
    and taxa shipped with CopyRighter
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:5e3cdadc2bbd973510a3ec7b9be3ccdfc8a47ad736519752af74b95905d39625
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fangly/AmpliCopyrighter
  tools:
  - Bio::Community
  - CopyRighter
  - Greengenes database
  - Perl >= 5.10.0
  - make
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# phylum-relative-abundance-computation

## Summary

Compute and compare phylum-level relative abundance profiles from 16S rRNA amplicon OTU tables, before and after CopyRighter gene copy number (GCN) correction, to quantify lineage-level shifts such as the Bacteroidetes and Firmicutes changes in the twin gut cohort.

## When to use

Use when a 16S rRNA amplicon OTU table (biom/QIIME format with read counts) must be GCN-corrected and aggregated to phylum level to report median relative abundances and phylum shifts (e.g. Bacteroidetes, Firmicutes) across a cohort.

## When NOT to use

- The input community file contains percentages instead of raw read counts, or taxa have been re-formatted from their original Greengenes taxonomic strings (CopyRighter requires counts and UNALTERED assignments)
- The taxonomy is not Greengenes (e.g. Silva); CopyRighter's distributed GCN estimates cover the Greengenes taxonomy and extension to other taxonomies is only anticipated, not available
- The goal is enterotype/dominant-genus clustering rather than phylum-level abundance summary, or the analysis requires absolute (qPCR-combined) abundances without a total-abundance file being supplied via -t

## Inputs

- 16S rRNA amplicon OTU table in biom, QIIME, GAAS, Unifrac, or generic tabular format containing read counts (not percentages) with unaltered Greengenes taxonomic assignments
- CopyRighter trait-value data file of phylogenetically-based 16S rRNA gene copy number estimates for Greengenes OTUs and taxa (or COPYRIGHTER_DB environment variable)

## Outputs

- GCN-corrected community file with relative abundances expressed in percent
- Phylum-level relative abundance profiles (uncorrected and corrected) per sample
- Per-phylum median relative abundances across the cohort (e.g. Bacteroidetes 21.7%, Firmicutes 76.0% after correction)

## How to apply

Load the amplicon OTU table (read counts, unaltered Greengenes taxonomic strings) and run CopyRighter with phylogenetic-level GCN correction, weighting each taxon's counts by the inverse of its lineage-specific 16S rRNA copy number from the Greengenes GCN estimate files shipped with the tool. Recompute relative abundances after correction, then aggregate corrected and uncorrected profiles to phylum level and compute per-phylum median relative abundances across samples. In the twin cohort use case, verify the correction reproduces the reported median shifts: Bacteroidetes 31.1% -> 21.7% and Firmicutes 67.1% -> 76.0%. Use the verbose (-v) mode to confirm trait values were assigned to taxa as intended before trusting the aggregated phylum profile.

## Related tools

- **CopyRighter** (Corrects OTU relative abundances by weighting read counts by the inverse lineage-specific 16S rRNA gene copy number before phylum aggregation) — https://github.com/fangly/AmpliCopyrighter
- **Perl >= 5.10.0** (Runtime environment in which CopyRighter is installed and executed)
- **Bio::Community** (Perl dependency of CopyRighter for community file handling (e.g. bc_use_repr_id for OTU ID lookup))
- **Greengenes database** (Taxonomy/reference for the phylogenetically-based GCN estimate files for OTUs and taxa shipped with CopyRighter)

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -v
```

## Evaluation signals

- Corrected phylum-level medians match the reported Bacteroidetes 21.7% and Firmicutes 76.0% (from uncorrected 31.1% and 67.1%) within reasonable tolerance
- Output file is produced in the same format as the input, with relative abundances expressed in percent and community sums preserved
- With -v verbose mode, each taxon receives the expected inverse-GCN weight assignment (desc or id lookup per the -l option)
- Corrected abundances are lower for high-GCN lineages and higher for low-GCN (e.g. ~1-copy) lineages relative to the uncorrected profile
- Per-sample phylum sums are consistent and no taxon was dropped due to failed GCN lookup

## Limitations

- GCN information is lacking for most microbial species, so estimates are phylogenetically inferred from close relatives and may be inaccurate where no close sequenced relative exists
- CopyRighter currently covers GCN for the 16S rRNA gene only within the Greengenes taxonomy; correction of GC percent bias, genome length bias, and fungal ITS variation is only hypothetical pending pre-computed estimates
- Correction effects are non-trivial to predict for communities with tens to thousands of species

## Evidence

- [readme] CopyRighter corrects relative abundances via inverse GCN weighting: "Using this information, CopyRigher corrects microbial relative abundance by applying a weight proportional to the inverse of the estimated copy number to each species."
- [readme] Input must contain read counts with unaltered taxonomy: [evidence span withheld — anchor status: external_doc]
- [abstract] GCN correction yields more accurate relative abundance estimates: "Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance"
- [discussion] Recommend phylogenetic-level correction on every sample: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction"
