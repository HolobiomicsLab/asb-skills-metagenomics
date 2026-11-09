---
name: gene-copy-number-correction
description: Use when amplicon (16S/18S rRNA) or microarray profiles in biom, QIIME,
  GAAS, Unifrac, or generic tabular format with read counts and unaltered Greengenes
  taxonomic assignments must be corrected for GCN bias, or when qPCR total abundances
  need correction.
when_to_use_negative:
- The input profile is shotgun metagenomic data where genome-length bias (not 16S
  GCN bias) dominates — CopyRighter handles it only with the genome-length trait file,
  and this skill targets 16S GCN correction
- Taxonomic assignments are not Greengenes (e.g. Silva), since pre-computed GCN files
  currently cover only Greengenes and extension to other taxonomies is only anticipated
- The input file contains percentages rather than read counts or has altered taxonomic
  strings, which the README explicitly requires for valid correction
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: CopyRighter
  role: Applies the inverse-GCN weighting to amplicon profiles and qPCR totals using
    lineage-specific Greengenes GCN estimates
  repo: github.com/fangly/AmpliCopyrighter
- name: Grinder
  role: Simulates 16S rRNA amplicon and shotgun metagenomic reads of in silico mock
    communities to validate correction accuracy
- name: Bio::Community
  role: Perl library used to read/write community profile files and via bc_use_repr_id
    to map OTU IDs to Greengenes IDs
- name: Perl >= 5.10.0
  role: Runtime environment for installing and executing CopyRighter
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  - task_004
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:63db5c09ca0e7499838ee460da0037b9660bdef783e988495498ba8e1529f0e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - GNU make
  - Grinder
  - Perl >= 5.10.0
  - make
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gene-copy-number-correction

## Summary

Corrects 16S rRNA amplicon (and optionally qPCR total) abundance bias caused by lineage-specific gene copy number (GCN) variation using CopyRighter, weighting read counts by the inverse of pre-computed Greengenes GCN estimates to yield more accurate relative and absolute microbial profiles.

## When to use

Use when amplicon (16S/18S rRNA) or microarray profiles in biom, QIIME, GAAS, Unifrac, or generic tabular format with read counts and unaltered Greengenes taxonomic assignments must be corrected for GCN bias, or when qPCR total abundances need correction.

## When NOT to use

- The input profile is shotgun metagenomic data where genome-length bias (not 16S GCN bias) dominates — CopyRighter handles it only with the genome-length trait file, and this skill targets 16S GCN correction
- Taxonomic assignments are not Greengenes (e.g. Silva), since pre-computed GCN files currently cover only Greengenes and extension to other taxonomies is only anticipated
- The input file contains percentages rather than read counts or has altered taxonomic strings, which the README explicitly requires for valid correction

## Inputs

- Community profile with read counts (biom, QIIME, GAAS, Unifrac, or generic tabular site-by-species format) with unaltered Greengenes taxonomic strings
- Tab-delimited trait/GCN estimate file (ID or taxonomic string in col 1, trait value in col 2), or the distributed Greengenes GCN files via the COPYRIGHTER_DB environment variable
- Optional tab-delimited total abundance file (community name, qPCR total abundance) for the -t option

## Outputs

- Corrected community file (same format as input) with GCN-weighted relative abundances in percent
- Corrected total microbial abundance file (*.tsv) and absolute abundance per taxon file (*_combined.txt) when -t is used
- Community-average trait value reported on standard output

## How to apply

Install CopyRighter with Perl (>= 5.10.0) via 'perl Makefile.PL', 'make', and 'make install' (or local::lib without administrator privileges), which pulls in Bio::Community, Getopt::Euclid (>= 0.4.3), Method::Signatures and Scalar::Util. Prepare an input community profile containing raw read counts (not percentages) with UNALTERED Greengenes taxonomic strings (e.g. 'k__Bacteria; p__Proteobacteria; c__Alphaproteobacteria; ...'), optionally replacing arbitrary OTU IDs with Greengenes IDs using bc_use_repr_id so the '-l id' lookup can be used. Run CopyRighter with the distributed GCN data files (IMG 4.0 genomes mapped onto Oct 2012 Greengenes taxonomy; GCN spans 1–15 in Bacteria, up to 5 in Archaea), preferring phylogenetic-level over taxonomic-level correction for highest accuracy; unassigned OTUs receive the community-average GCN so they do not distort other OTUs' relative abundance. Use '-t' to divide qPCR total abundance by the community-average GCN and obtain absolute abundances, and use '-v' verbose mode to verify trait value assignments. Validate by checking that the corrected profile's Hellinger/euclidean distance to a known (mock) composition is smaller than the uncorrected distance, or that reported phylum shifts and diversity changes (e.g. twin cohort Bacteroidetes 31.1% -> 21.7%, Firmicutes 67.1% -> 76.0%, three rather than two enterotypes) are reproduced.

## Related tools

- **CopyRighter** (Applies the inverse-GCN weighting to amplicon profiles and qPCR totals using lineage-specific Greengenes GCN estimates) — github.com/fangly/AmpliCopyrighter
- **Grinder** (Simulates 16S rRNA amplicon and shotgun metagenomic reads of in silico mock communities to validate correction accuracy)
- **Bio::Community** (Perl library used to read/write community profile files and via bc_use_repr_id to map OTU IDs to Greengenes IDs)
- **Perl >= 5.10.0** (Runtime environment for installing and executing CopyRighter)

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -v
```

## Evaluation signals

- Corrected profile's Hellinger and euclidean distances to the expected mock composition (e.g. the 22-species in vitro mock SRR074387) are smaller than the uncorrected profile's distances
- In silico validation: corrected amplicon profiles agree better with expected compositions and simulated Grinder metagenomic profiles, with phylogenetic-level correction slightly more accurate than taxonomic-level
- Output file parses as a community table in the same format as the input, contains the expected taxa, and CopyRighter completes without error; -v shows taxa matched as intended
- Twin cohort re-analysis reproduces corrected phylum medians (Bacteroidetes 21.7%, Firmicutes 76.0%), Berger-Parker per-sample differences within 0–23.3%, and three rather than two enterotypes with 23.9% of samples reclassified
- Unassigned OTUs do not distort other OTUs' relative abundances (they receive the community-average GCN), and qPCR totals are divided by the community-average GCN when -t is used

## Limitations

- No version of CopyRighter beyond the README's 0.46 is stated in the article, and GCN coverage is limited to the Greengenes taxonomy; extension to other taxonomies (e.g. Silva) is only anticipated
- Currently limited to 16S rRNA GCN (and genome length); correction of GC percent bias, genome length bias, and fungal ITS variation is hypothetical pending appropriate pre-computed estimates
- Correction effects are non-trivial to predict for communities with tens to thousands of species, and unassigned OTUs can only be handled with a community-average GCN
- Input must contain read counts (not percentages) and taxa must have UNALTERED taxonomic assignments, otherwise lookups fail

## Evidence

- [supplementary] Weight each OTU's read counts by the inverse of its GCN: [evidence span withheld — anchor status: not_found]
- [readme] Input requirements: [evidence span withheld — anchor status: external_doc]
- [discussion] Recommendation on correction level: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction"
- [abstract] Validation result: "Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance"
- [methods] Installation steps: "perl Makefile.PL
     make"
