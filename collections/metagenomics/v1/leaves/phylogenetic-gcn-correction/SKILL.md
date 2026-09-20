---
name: phylogenetic-gcn-correction
description: Use when 16S rRNA amplicon count tables (biom, QIIME, generic) need GCN-bias
  correction, or when qPCR-derived total abundance must be adjusted by average community
  GCN before downstream diversity or enterotype analyses.
when_to_use_negative:
- The input table already contains percentages or altered taxonomic strings — CopyRighter
  requires raw read counts and unaltered assignments
- The community was profiled with a taxonomy other than Greengenes (e.g. Silva) —
  GCN estimates are currently only distributed for the Greengenes database
- The bias of interest is GC percent, genome length beyond metagenomic trait data,
  or fungal ITS variation — these corrections are only hypothetical pending appropriate
  pre-computed estimates
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0699
- http://edamontology.org/topic_3174
tools:
- name: CopyRighter
  role: 'Core tool: reweights OTU relative abundances by the inverse lineage-specific
    GCN and corrects associated qPCR total abundance; run at phylogenetic level for
    highest accuracy'
  repo: https://github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl dependency providing community file handling; bc_use_repr_id can replace
    arbitrary OTU IDs with Greengenes IDs for ID-based trait lookup
- name: Perl
  role: Runtime (>= 5.10.0) for the copyrighter script; installed with perl Makefile.PL
    / make / make install
- name: make
  role: Build step in the installation of CopyRighter and its CPAN dependencies
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:48ddc8520fd26bb53146f3f294d00a370fd8f0604bd709d5c11d31a844fd0f58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fangly/AmpliCopyrighter
  tools:
  - Bio::Community
  - CopyRighter
  - Perl
  - make
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0699
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# phylogenetic-gcn-correction

## Summary

Corrects 16S rRNA amplicon microbial profiles and associated qPCR total abundance for 16S rRNA gene copy number (GCN) bias using CopyRighter, which reweights OTU abundances by the inverse of phylogenetically-inferred, lineage-specific GCN estimates from the Greengenes database.

## When to use

Use when 16S rRNA amplicon count tables (biom, QIIME, generic) need GCN-bias correction, or when qPCR-derived total abundance must be adjusted by average community GCN before downstream diversity or enterotype analyses.

## When NOT to use

- The input table already contains percentages or altered taxonomic strings — CopyRighter requires raw read counts and unaltered assignments
- The community was profiled with a taxonomy other than Greengenes (e.g. Silva) — GCN estimates are currently only distributed for the Greengenes database
- The bias of interest is GC percent, genome length beyond metagenomic trait data, or fungal ITS variation — these corrections are only hypothetical pending appropriate pre-computed estimates

## Inputs

- 16S rRNA amplicon OTU/community table in biom, QIIME, GAAS, Unifrac, or generic tabular format, with read counts and unaltered Greengenes taxonomic strings
- Pre-computed GCN trait-value file for Greengenes OTUs/taxa (tab-delimited, ID or taxstring in col 1, 16S rRNA count in col 2), shipped with CopyRighter
- Optional tab-delimited total abundance file (community name, total abundance), e.g. 16S rRNA qPCR numbers

## Outputs

- Trait-corrected community file with relative abundances in percent (same format as input)
- Corrected total microbial abundance file (e.g. out_copyrighted_total.tsv) when -t is given
- Absolute abundance per taxon file (e.g. out_copyrighted_combined.txt) when -t is given
- Average trait (GCN) value per community reported on standard output

## How to apply

Install CopyRighter and its dependencies (Perl >= 5.10.0, make, Bio::Community) via 'perl Makefile.PL', 'make', and 'make install' (or local::lib without admin rights). Provide an input community file with read counts (not percentages) and unaltered Greengenes taxonomic strings (e.g. 'k__Bacteria; p__Proteobacteria; ...; s__siberiense'), and use the pre-computed phylogenetically-based GCN files for Greengenes OTUs and taxa distributed with the tool (-d or the COPYRIGHTER_DB environment variable). Run CopyRighter with phylogenetic-level rather than taxonomic-level lookup to weight each taxon's reads by the inverse of its estimated copy number; optionally pass a tab-delimited total-abundance file with -t so the average GCN corrects qPCR numbers and absolute abundances are emitted. Validate with verbose mode (-v) that trait assignments match taxa as intended, then check outputs such as the reported average GCNs of 2.69 (day 27) and 1.61 (day 362) for the anaerobic digester profile, or the twin-cohort phylum shifts (Bacteroidetes 31.1%->21.7%, Firmicutes 67.1%->76.0%) and the three-rather-than-two enterotype reclassification.

## Related tools

- **CopyRighter** (Core tool: reweights OTU relative abundances by the inverse lineage-specific GCN and corrects associated qPCR total abundance; run at phylogenetic level for highest accuracy) — https://github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl dependency providing community file handling; bc_use_repr_id can replace arbitrary OTU IDs with Greengenes IDs for ID-based trait lookup)
- **Perl** (Runtime (>= 5.10.0) for the copyrighter script; installed with perl Makefile.PL / make / make install)
- **make** (Build step in the installation of CopyRighter and its CPAN dependencies)

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -t qPCR_totals.tsv -v
```

## Evaluation signals

- Average community GCN values match reported references (e.g. 2.69 for day 27 and 1.61 for day 362 anaerobic digester samples)
- Corrected qPCR trend reproduces the reported significance flip: uncorrected P = 0.0288 (significant) vs corrected P = 0.413 (not significant), paired t-test
- Corrected phylum medians match reported shifts (Bacteroidetes 31.1% -> 21.7%; Firmicutes 67.1% -> 76.0%) in the twin gut cohort
- Per-sample Berger-Parker dominance index differences before vs after correction fall within the reported 0-23.3% range
- Verbose mode (-v) confirms each taxon received its intended GCN assignment; output relative abundances sum to 100%

## Limitations

- GCN estimates are currently limited to the Greengenes taxonomy; extension to Silva and other taxonomies is only anticipated, not available
- Only 16S rRNA GCN is supported; correction of GC percent bias, genome length bias, and fungal ITS variation requires appropriate pre-computed estimates that do not yet exist
- GCN information is lacking for most microbial species, so uncultured organisms rely on estimates from close sequenced relatives; communities with tens to thousands of species make correction effects non-trivial to predict

## Evidence

- [abstract] We created CopyRighter, new software which uses these estimates to correct 16S rRNA amplicon microbial profiles and associated quantitative (q)PCR total abundance.: "We created CopyRighter, new software which uses these estimates to correct 16S rRNA amplicon microbial profiles and associated quantitative (q)PCR total abundance."
- [discussion] We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction, to obtain the highest accuracy possible."
- [readme] The file must contain read counts (not percentages) and taxa must have UNALTERED taxonomic assignments.: [evidence span withheld — anchor status: external_doc]
- [readme] File containing the total microbial abundance to be corrected by the average trait value, e.g. 16S rRNA quantitative PCR numbers: [evidence span withheld — anchor status: external_doc]
- [readme] Data files of 16S rRNA gene copy number and genome length (based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy) are distributed with CopyRighter.: [evidence span withheld — anchor status: external_doc]
