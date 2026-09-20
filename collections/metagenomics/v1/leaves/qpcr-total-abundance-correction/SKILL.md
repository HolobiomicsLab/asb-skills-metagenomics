---
name: qpcr-total-abundance-correction
description: Use when qPCR-derived total microbial abundance must be combined with
  a 16S rRNA amplicon profile and the qPCR numbers need correcting for lineage-specific
  16S rRNA gene copy number bias.
when_to_use_negative:
- The input community file contains percentages rather than read counts, or taxa strings
  have been altered, since CopyRighter requires raw counts and unaltered Greengenes
  taxonomic assignments
- The total abundance comes from a non-16S-rRNA qPCR target or a taxonomic framework
  other than Greengenes (e.g. Silva), for which CopyRighter GCN estimates are not
  yet available
- No qPCR total abundance was measured, in which case only relative abundance correction
  applies
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0688
- http://edamontology.org/topic_3697
tools:
- name: CopyRighter
  role: Corrects the qPCR total abundance by dividing by the community-average 16S
    rRNA gene copy number computed from the GCN-corrected amplicon profile, producing
    corrected totals and per-taxon absolute abundances
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl library used by CopyRighter to read and write the community file formats
    (biom, QIIME, generic) for the input profile and qPCR totals
- name: Perl >= 5.10.0
  role: Runtime environment in which CopyRighter is installed and executed (perl Makefile.PL;
    make; make install)
- name: make
  role: Build step for installing CopyRighter from source (perl Makefile.PL, make,
    make install)
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:ed0369e3268ec05395f67d9a4bdf7fcb2a0352436dfdcc3d663c1c4ef32478d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Perl >= 5.10.0
  - make
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0688
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# qpcr-total-abundance-correction

## Summary

Corrects 16S rRNA qPCR-based total microbial abundance estimates for gene copy number (GCN) bias by dividing the qPCR total by the community-average 16S rRNA gene copy number computed from the CopyRighter-corrected amplicon profile.

## When to use

Use when qPCR-derived total microbial abundance must be combined with a 16S rRNA amplicon profile and the qPCR numbers need correcting for lineage-specific 16S rRNA gene copy number bias.

## When NOT to use

- The input community file contains percentages rather than read counts, or taxa strings have been altered, since CopyRighter requires raw counts and unaltered Greengenes taxonomic assignments
- The total abundance comes from a non-16S-rRNA qPCR target or a taxonomic framework other than Greengenes (e.g. Silva), for which CopyRighter GCN estimates are not yet available
- No qPCR total abundance was measured, in which case only relative abundance correction applies

## Inputs

- Community/OTU table with read counts and unaltered Greengenes 2012/10 taxonomic strings (biom, QIIME, GAAS, Unifrac, or generic tabular format)
- Tab-delimited two-column qPCR total abundance file (community name, total abundance), passed via -t
- Greengenes-based pre-computed GCN estimate file (or COPYRIGHTER_DB environment variable pointing to it)

## Outputs

- Corrected total microbial abundance file (e.g. out_copyrighted_total.tsv)
- Combined absolute abundance table per taxon (e.g. out_copyrighted_combined.txt)
- GCN-corrected relative abundance community file (percent), reported alongside the community-average trait value on stdout

## How to apply

Provide CopyRighter with the OTU/community table with unaltered Greengenes taxonomic assignments (read counts, not percentages) via -i, and a tab-delimited two-column file of community name and qPCR total abundance via -t. CopyRighter weights each OTU's counts by the inverse of its lineage-specific GCN (using Greengenes phylogenetic-level estimates, preferred over taxonomic-level for highest accuracy), assigns the community-average GCN to unassigned OTUs, then divides the qPCR total by this community-average GCN. It emits out_copyrighted_total.tsv (corrected totals) and out_copyrighted_combined.txt (absolute abundance per taxon). Validate against the anaerobic digester benchmark: after correction the day 27 to day 362 paired t-test should shift from P = 0.0288 (significant, uncorrected) to P = 0.413 (not significant), with corrected totals averaging 4.86e9 genomes/ml and community GCN averages of 2.69 (day 27) and 1.61 (day 362).

## Related tools

- **CopyRighter** (Corrects the qPCR total abundance by dividing by the community-average 16S rRNA gene copy number computed from the GCN-corrected amplicon profile, producing corrected totals and per-taxon absolute abundances) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl library used by CopyRighter to read and write the community file formats (biom, QIIME, generic) for the input profile and qPCR totals)
- **Perl >= 5.10.0** (Runtime environment in which CopyRighter is installed and executed (perl Makefile.PL; make; make install))
- **make** (Build step for installing CopyRighter from source (perl Makefile.PL, make, make install))

## Examples

```
copyrighter -i otu_table.qiime -t qpcr_totals.tsv -o out_copyrighted.txt -v
```

## Evaluation signals

- out_copyrighted_total.tsv and out_copyrighted_combined.txt are produced when -t is supplied, and the corrected total equals the raw qPCR total divided by the reported community-average GCN
- On the anaerobic digester benchmark, the corrected day 27/day 362 paired t-test yields P = 0.413 (vs uncorrected P = 0.0288) with corrected totals averaging 4.86 × 10^9 genomes/ml
- The reported average community GCN matches the expected values of 2.69 (day 27) and 1.61 (day 362)
- Running with -v shows each taxon receiving the intended lineage-specific GCN assignment; unassigned OTUs receive the community-average GCN
- CopyRighter completes without error and the output parses as a valid community table in the same format as the input

## Limitations

- GCN estimates are currently limited to the 16S rRNA gene with Greengenes taxonomy; other taxonomies such as Silva are not yet covered
- OTUs lacking any taxonomic assignment are assigned the community-average GCN, so very poorly assigned profiles dilute correction accuracy
- Phylogenetic-level correction is recommended over taxonomic-level for highest accuracy; correction effects are non-trivial to predict for communities with tens to thousands of species

## Evidence

- [readme] qPCR totals divided by community GCN average: "Total abundance can optionally be provided (-t option), corrected and combined with relative abundance estimates to get the absolute abundance of each species."
- [abstract] uses CopyRighter to correct qPCR totals with amplicon profile: "We created CopyRighter, new software which uses these estimates to correct 16S rRNA amplicon microbial profiles and associated quantitative (q)PCR total abundance."
- [discussion] phylogenetic-level correction recommended: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction, to obtain the highest accuracy possible."
- [readme] input must contain read counts and unaltered taxa: [evidence span withheld — anchor status: external_doc]
- [other] unassigned OTUs get community-average GCN: [evidence span withheld — anchor status: not_found]
