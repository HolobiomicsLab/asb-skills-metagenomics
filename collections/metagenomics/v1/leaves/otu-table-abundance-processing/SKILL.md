---
name: otu-table-abundance-processing
description: Use when an OTU/amplicon abundance table with unaltered Greengenes taxonomic
  assignments (read counts, not percentages) must be corrected for 16S rRNA gene copy
  number bias, or when 16S qPCR totals need GCN correction.
when_to_use_negative:
- The input table contains percentages rather than read counts, or taxa assignments
  have been altered from the Greengenes format CopyRighter expects
- The taxonomy is not Greengenes — CopyRighter does not yet cover other taxonomies
  such as Silva, which is only anticipated for future extension
- The bias of interest is GC percent, genome length, or fungal ITS variation without
  appropriate pre-computed trait estimates — CopyRighter is currently limited to 16S
  rRNA GCN correction
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: CopyRighter
  role: Core software that applies inverse-GCN weights to OTU read counts and optionally
    corrects qPCR totals; version 0.46
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl library used to read and write the input/output community (OTU) tables;
    its bc_use_repr_id script can replace arbitrary OTU IDs with Greengenes IDs
- name: Perl >= 5.10.0
  role: Runtime language in which CopyRighter is implemented and executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:4d1d6e4bc50553c1eeb05e9169359564acf71165d41bd8a01a1835e9fef50c23
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Perl >= 5.10.0
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# otu-table-abundance-processing

## Summary

Correct 16S rRNA gene copy number (GCN) bias in OTU abundance tables using CopyRighter, which weights each OTU's read counts by the inverse of its lineage-specific GCN estimated within the Greengenes taxonomy, and optionally corrects qPCR total abundance by the community-average GCN.

## When to use

Use when an OTU/amplicon abundance table with unaltered Greengenes taxonomic assignments (read counts, not percentages) must be corrected for 16S rRNA gene copy number bias, or when 16S qPCR totals need GCN correction.

## When NOT to use

- The input table contains percentages rather than read counts, or taxa assignments have been altered from the Greengenes format CopyRighter expects
- The taxonomy is not Greengenes — CopyRighter does not yet cover other taxonomies such as Silva, which is only anticipated for future extension
- The bias of interest is GC percent, genome length, or fungal ITS variation without appropriate pre-computed trait estimates — CopyRighter is currently limited to 16S rRNA GCN correction

## Inputs

- OTU/community table in biom, QIIME, GAAS, Unifrac, or generic (tabular site-by-species) format with read counts (not percentages) and unaltered Greengenes taxonomic assignments
- Trait (GCN) estimate data file: tab-delimited, two columns (OTU ID or Greengenes taxstring, 16S rRNA count), distributed with CopyRighter or via COPYRIGHTER_DB environment variable
- Optional tab-delimited qPCR total abundance file (community name, total abundance), passed with -t

## Outputs

- Trait-corrected community file (same format as input, e.g. corrected_community_profile.tsv) with relative abundance expressed in percent
- Average trait (GCN) value per community reported on standard output
- With -t: corrected total microbial abundance file (out_copyrighted_total.tsv) and per-taxon absolute abundance file (out_copyrighted_combined.txt)

## How to apply

Load the OTU table (biom, QIIME, GAAS, Unifrac, or generic tabular format) via Bio::Community readers in Perl; the table must contain read counts and unaltered Greengenes taxonomic strings (e.g. 'k__Bacteria; p__Proteobacteria; ...'). Run CopyRighter on each sample, looking up each OTU's pre-computed GCN from the Greengenes GCN estimate files (IMG 4.0 genomes mapped onto Oct 2012 Greengenes taxonomy), by default matching on taxonomic description ('-l desc') rather than OTU ID. Each OTU's read counts are weighted by the inverse of its GCN (ri = ci/gi, renormalized); OTUs without any taxonomic assignment receive the community-average GCN so they do not distort other OTUs' relative abundance. Prefer phylogenetic-level over taxonomic-level correction for the highest accuracy, and use verbose mode (-v) to verify trait assignments. Optionally pass qPCR totals with -t so the total is divided by the community-average GCN, producing corrected total and combined absolute-abundance files. CopyRighter version 0.46 is installed via perl Makefile.PL; make; make install (or local::lib without administrator privileges).

## Related tools

- **CopyRighter** (Core software that applies inverse-GCN weights to OTU read counts and optionally corrects qPCR totals; version 0.46) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl library used to read and write the input/output community (OTU) tables; its bc_use_repr_id script can replace arbitrary OTU IDs with Greengenes IDs)
- **Perl >= 5.10.0** (Runtime language in which CopyRighter is implemented and executed)

## Examples

```
copyrighter -i otu_table.qiime -o corrected_community_profile.tsv -v
```

## Evaluation signals

- The corrected output file exists, parses as a community table (biom/QIIME/generic), and contains the expected taxa
- Corrected relative abundances are expressed in percent and renormalized (weights ri = ci/gi normalized)
- Running CopyRighter in verbose mode (-v) confirms each taxon was matched to the intended GCN trait value (e.g. phylogenetic-level lookup by 'desc')
- CopyRighter completes without error on the example OTU table input
- The community-average GCN reported on standard output is consistent with the trait values in the Greengenes GCN data file, and unassigned OTUs were given this average rather than excluded

## Limitations

- CopyRighter only covers Greengenes taxonomies; extension to other taxonomies such as Silva is anticipated but not yet available
- Only 16S rRNA GCN correction is currently supported; correction of GC percent bias, genome length bias, and fungal ITS variation requires appropriate pre-computed estimates that do not yet exist
- OTUs lacking any taxonomic assignment must be assigned the community-average GCN; mismatches in taxonomic strings (e.g. missing whitespace after semicolons) will break trait lookup
- Communities contain tens to thousands of species, making the effects of correction non-trivial to predict

## Evidence

- [readme] Using this information, CopyRigher corrects microbial relative abundance by applying a weight proportional to the inverse of the estimated copy number to each species.: "Using this information, CopyRigher corrects microbial relative abundance by applying a weight proportional to the inverse of the estimated copy number to each species."
- [readme] The file must contain read counts (not percentages) and taxa must have UNALTERED taxonomic assignments.: [evidence span withheld — anchor status: external_doc]
- [discussion] We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction, to obtain the highest accuracy possible.: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction, to obtain the highest accuracy"
- [abstract] Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance: "Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance"
- [methods] Perl (>= 5.10.0) <http://www.perl.com/download.csp>: "Perl (>= 5.10.0) <http://www.perl.com/download.csp>"
