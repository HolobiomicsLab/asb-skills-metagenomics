---
name: biomass-trend-significance-testing
description: Use when you have qPCR-derived total microbial abundance estimates for
  paired samples (e.g., day 27 vs day 362 anaerobic digester biomass) and need to
  test temporal trends before and after GCN correction.
when_to_use_negative:
- The abundance data are already trait-corrected (e.g. previously run through CopyRighter)
  — re-correcting would double-weight by inverse GCN
- Samples are not paired (independent groups), in which case a paired t-test is inappropriate
  and a different test is required
- The amplicon profile was assigned with a taxonomy other than Greengenes, since CopyRighter's
  pre-computed GCN data files cover the Oct 2012 Greengenes taxonomy
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0699
- http://edamontology.org/topic_3299
tools:
- name: CopyRighter
  role: Corrects the qPCR total microbial abundance by the average community 16S rRNA
    gene copy number alongside the amplicon profile, producing corrected totals used
    in the t-test
  repo: github.com/fangly/AmpliCopyrighter
- name: Perl
  role: Runtime dependency (>= 5.10.0) for executing the copyrighter script and its
    Bio::Community-based pipeline
- name: Bio::Community
  role: Perl library used by CopyRighter to read/write community files and handle
    OTU ID to Greengenes ID mapping (bc_use_repr_id)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:7bbfe55ad472a372c509d39a12e210aa33b9ca61db2dfe218fd886fc08678b18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Perl
  - make
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0699
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# biomass-trend-significance-testing

## Summary

Statistical skill for testing whether total microbial abundance (qPCR-derived biomass) changes significantly over time, comparing results with and without 16S rRNA gene copy number (GCN) correction via CopyRighter. It reveals how GCN bias can create or remove apparent biomass trends.

## When to use

Use when you have qPCR-derived total microbial abundance estimates for paired samples (e.g., day 27 vs day 362 anaerobic digester biomass) and need to test temporal trends before and after GCN correction.

## When NOT to use

- The abundance data are already trait-corrected (e.g. previously run through CopyRighter) — re-correcting would double-weight by inverse GCN
- Samples are not paired (independent groups), in which case a paired t-test is inappropriate and a different test is required
- The amplicon profile was assigned with a taxonomy other than Greengenes, since CopyRighter's pre-computed GCN data files cover the Oct 2012 Greengenes taxonomy

## Inputs

- qPCR total microbial abundance estimates (16S rRNA gene copies per sample) in a tab-delimited two-column file: community name and total abundance
- 16S rRNA amplicon OTU table with read counts and unaltered Greengenes taxonomic strings (biom, QIIME, or generic format), e.g. from SRR1145444
- CopyRighter pre-computed Greengenes GCN trait file (phylogenetically-based estimates for all OTUs/taxa)

## Outputs

- Corrected total abundance file (e.g. out_copyrighted_total.tsv) and absolute abundance per taxon (out_copyrighted_combined.txt)
- Average community GCN per sample (reported on stdout, e.g. 2.69 and 1.61)
- Paired t-test P-values for uncorrected and corrected biomass trends (uncorrected P = 0.0288; corrected P = 0.413)

## How to apply

First correct the 16S amplicon profile and associated qPCR total abundance with CopyRighter, passing the qPCR numbers via the -t option so the tool divides total abundance by the average community GCN (computed as 2.69 for day 27 and 1.61 for day 362 in the anaerobic digester example). Then run paired t-tests on the biomass values across sampling dates in both directions: uncorrected totals (reported P = 0.0288, significant decrease) and CopyRighter-corrected totals (reported P = 0.413, not significant), with corrected abundance averaging 4.86 x 10^9 genomes/ml. The rationale is that amplicon-based total abundance estimates are biased by GCN variation spanning 1 to 15 copies in Bacteria, so a seemingly significant biomass trend may be an artifact of shifting community composition rather than true biomass change. Judge success by reproducing the significance flip (P < 0.05 uncorrected vs P ~ 0.4 corrected) and the reported average GCN values per time point.

## Related tools

- **CopyRighter** (Corrects the qPCR total microbial abundance by the average community 16S rRNA gene copy number alongside the amplicon profile, producing corrected totals used in the t-test) — github.com/fangly/AmpliCopyrighter
- **Perl** (Runtime dependency (>= 5.10.0) for executing the copyrighter script and its Bio::Community-based pipeline)
- **Bio::Community** (Perl library used by CopyRighter to read/write community files and handle OTU ID to Greengenes ID mapping (bc_use_repr_id))

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -t qpcr_totals.tsv -v
```

## Evaluation signals

- Corrected average GCN matches the reported values of 2.69 (day 27) and 1.61 (day 362) per sampling date
- Uncorrected paired t-test reproduces the significant decrease (P = 0.0288)
- CopyRighter-corrected paired t-test reproduces the non-significant result (P = 0.413)
- Corrected qPCR-based total microbial abundance averages approximately 4.86 x 10^9 genomes/ml across the two time points
- The corrected output files (out_copyrighted_total.tsv, out_copyrighted_combined.txt) are produced when -t is supplied, with relative abundances expressed in percent

## Limitations

- CopyRighter's pre-computed GCN estimates currently cover only the Greengenes taxonomy; Silva and other taxonomies are anticipated but not yet supported, so biomass correction is limited to Greengenes-assigned profiles
- GCN values are phylogenetic estimates rather than direct measurements, so corrected biomass trends inherit estimation uncertainty from close-relative inference
- Only 16S rRNA GCN is supported; correction of GC percent bias, genome length bias, and fungal ITS variation remains hypothetical pending appropriate pre-computed estimates

## Evidence

- [abstract] qPCR correction changes trend significance: "We created CopyRighter, new software which uses these estimates to correct 16S rRNA amplicon microbial profiles and associated quantitative (q)PCR total abundance."
- [discussion] phylogenetic-level correction recommendation: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction"
- [intro] GCN range biasing total abundance: "This order of magnitude range biases both amplicon microbial profiles and estimates of total"
- [readme] -t option corrects qPCR totals: [evidence span withheld — anchor status: external_doc]
- [readme] average trait reported on stdout: [evidence span withheld — anchor status: external_doc]
