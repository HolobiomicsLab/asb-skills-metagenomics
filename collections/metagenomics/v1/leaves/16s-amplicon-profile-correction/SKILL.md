---
name: 16s-amplicon-profile-correction
description: Use when a 16S rRNA amplicon OTU table (biom, QIIME, GAAS, Unifrac or
  generic format) with read counts and unaltered Greengenes taxonomic assignments
  needs GCN bias correction, or when 16S qPCR total abundances must be corrected by
  community-average GCN.
when_to_use_negative:
- The input table contains percentages rather than raw read counts, or taxa have altered/non-Greengenes
  assignments — CopyRighter requires unaltered taxonomic strings and its GCN data
  currently cover Greengenes only (Silva extension is anticipated, not available)
- The goal is to correct GC percent bias, genome length bias, or fungal ITS variation
  — CopyRighter is currently limited to 16S rRNA gene GCN pending appropriate pre-computed
  estimates
- Only relative abundance without any GCN-bearing taxonomy is available (e.g. unclassified
  OTUs dominate), making community-average-GCN fallback the only correction possible
  and accuracy gains uncertain
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CopyRighter
  role: Core tool that looks up lineage-specific 16S rRNA GCN for each OTU and reweights
    amplicon read counts and qPCR totals
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl library used to read/write community files and, via bc_use_repr_id, to
    map arbitrary OTU IDs to Greengenes IDs for -l id lookups
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:2a2f9f710bef5416863d7200b997d581358009cf952287089ba111c2f563297d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# 16s-amplicon-profile-correction

## Summary

Corrects 16S rRNA gene copy number (GCN) bias in amplicon profiles and qPCR total abundance using CopyRighter, which weights each OTU's read counts by the inverse of its lineage-specific GCN inferred within a phylogenetic framework on the Greengenes database.

## When to use

Use when a 16S rRNA amplicon OTU table (biom, QIIME, GAAS, Unifrac or generic format) with read counts and unaltered Greengenes taxonomic assignments needs GCN bias correction, or when 16S qPCR total abundances must be corrected by community-average GCN.

## When NOT to use

- The input table contains percentages rather than raw read counts, or taxa have altered/non-Greengenes assignments — CopyRighter requires unaltered taxonomic strings and its GCN data currently cover Greengenes only (Silva extension is anticipated, not available)
- The goal is to correct GC percent bias, genome length bias, or fungal ITS variation — CopyRighter is currently limited to 16S rRNA gene GCN pending appropriate pre-computed estimates
- Only relative abundance without any GCN-bearing taxonomy is available (e.g. unclassified OTUs dominate), making community-average-GCN fallback the only correction possible and accuracy gains uncertain

## Inputs

- OTU/community file with read counts (not percentages) and unaltered Greengenes taxonomic assignments, in biom, QIIME, GAAS, Unifrac or generic tabular site-by-species format
- GCN trait data file (tab-delimited ID or taxstring with 16S rRNA count in column 2), either distributed with CopyRighter or provided via -d/COPYRIGHTER_DB
- Optional qPCR total abundance file (tab-delimited community name and total abundance) passed via -t

## Outputs

- Corrected community file (same format as input) with trait-corrected relative abundances expressed in percent
- Corrected total microbial abundance file (<output>_total.tsv) and combined absolute abundance file (<output>_combined.txt) when -t is used
- Average trait (GCN) value per community reported on standard output

## How to apply

Install CopyRighter (Perl >= 5.10.0, Bio::Community) via perl Makefile.PL; make; make install (use local::lib without administrator privileges). Run copyrighter on each sample, e.g. copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic; the input must contain read counts (not percentages) and unaltered Greengenes 2012/10 taxonomic strings (whitespace after each semicolon). Each OTU's pre-computed Greengenes GCN (from the data files shipped with CopyRighter or pointed to by the COPYRIGHTER_DB environment variable, or a custom -d file of tab-delimited ID/taxstring-to-trait pairs) is looked up and read counts are weighted by the inverse of that GCN; by default matching uses the taxonomic description (-l desc), but -l id with Bio::Community's bc_use_repr_id can replace arbitrary OTU IDs with their Greengenes IDs. Prefer phylogenetic-level over taxonomic-level correction for highest accuracy; unassigned OTUs receive the community-average GCN so they do not distort other OTUs' relative abundance. Optionally supply -t with a tab-delimited community-name/total-abundance file (e.g. 16S qPCR numbers) to divide totals by the community-average GCN, producing corrected totals and absolute abundances. Run with -v to display trait value assignments and confirm taxa are processed as intended.

## Related tools

- **CopyRighter** (Core tool that looks up lineage-specific 16S rRNA GCN for each OTU and reweights amplicon read counts and qPCR totals) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl library used to read/write community files and, via bc_use_repr_id, to map arbitrary OTU IDs to Greengenes IDs for -l id lookups)

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -v
```

## Evaluation signals

- Run with -v and inspect trait value assignments to confirm each taxon was matched as intended
- Output file exists, parses as a community table in the same format as input, and contains the expected taxa with relative abundances expressed in percent
- OTUs lacking taxonomic assignment received the community-average GCN and did not distort the relative abundance of other OTUs
- With -t, the corrected total (<output>_total.tsv) equals the qPCR total divided by the community-average GCN, and <output>_combined.txt yields absolute abundances
- On mock communities (in silico or in vitro), corrected profiles agree better with known compositions/metagenomic profiles than uncorrected ones

## Limitations

- Input must contain read counts, not percentages, and taxa must have UNALTERED taxonomic assignments
- GCN estimates cover only the Greengenes taxonomy (other taxonomies such as Silva are only anticipated future extensions); no software version is stated in the discussion
- Currently limited to 16S rRNA GCN correction — GC percent, genome length, and fungal ITS correction require appropriate pre-computed estimates that do not yet exist
- Communities with tens to thousands of species make the effects of correction non-trivial to predict

## Evidence

- [readme] Using this information, CopyRigher corrects microbial relative abundance by applying a weight proportional to the inverse of the estimated copy number to each species.: "Using this information, CopyRigher corrects microbial relative abundance by applying a weight proportional to the inverse of the estimated copy number to each species."
- [readme] The file must contain read counts (not percentages) and taxa must have UNALTERED taxonomic assignments.: [evidence span withheld — anchor status: external_doc]
- [discussion] We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction"
- [abstract] Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance: "Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance"
- [readme] The script bc_use_repr_id of Bio::Community can help in replacing arbitrary OTU IDs by their corresponding Greengenes ID.: [evidence span withheld — anchor status: external_doc]
