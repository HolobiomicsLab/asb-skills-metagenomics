---
name: primer-mismatch-evasion-quantification
description: Use when you have curated 16S rRNA gene sequences (or a new primer pair)
  and need to estimate what fraction of a community or lineage amplicon surveys would
  fail to detect due to primer-binding-site mismatches.
when_to_use_negative:
- Input is a metagenomic read set or genome bins rather than curated full-length 16S
  rRNA gene sequences spanning the primer regions
- 16S sequences still contain self-splicing introns/insertions ≥10 bp, which would
  falsely inflate apparent primer mismatch
- The goal is mapping metatranscriptomic reads to 16S genes rather than predicting
  amplicon-survey detectability
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0091
tools:
- name: USEARCH
  role: Clusters curated 16S rRNA gene sequences at 97% identity (cluster_smallmem,
    query_cov 0.50, target_cov 0.50) to remove redundancy before primer binding analysis
- name: Primer Prospector
  role: Assesses 515F/806R primer binding against each de-redundant 16S sequence with
    default parameters
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:5b7b421749fbbd6ce54f091196b25c9402d1528f2d88dc623afe216ded5d6a67
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Primer Prospector
  - USEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# primer-mismatch-evasion-quantification

## Summary

Quantifies the fraction of 16S rRNA gene sequences that would evade detection by the widely used 515F/806R amplicon-survey primers, by testing primer binding against curated, de-redundantized 16S sequences with Primer Prospector. Applied in Brown et al. (Nature, 2015) to show 50-100% of specific CPR phyla and ~20% of all bacteria in the study would be missed.

## When to use

Use when you have curated 16S rRNA gene sequences (or a new primer pair) and need to estimate what fraction of a community or lineage amplicon surveys would fail to detect due to primer-binding-site mismatches.

## When NOT to use

- Input is a metagenomic read set or genome bins rather than curated full-length 16S rRNA gene sequences spanning the primer regions
- 16S sequences still contain self-splicing introns/insertions ≥10 bp, which would falsely inflate apparent primer mismatch
- The goal is mapping metatranscriptomic reads to 16S genes rather than predicting amplicon-survey detectability

## Inputs

- Curated assembled 16S rRNA gene sequences (≥800 bp, introns/insertions ≥10 bp removed)
- Primer pair sequences (515F and 806R)
- Lineage/taxonomic assignment per 16S sequence

## Outputs

- primer_binding_analysis.csv: per-lineage evasion-rate summary (reproducing Extended Data Fig. 7)
- Per-sequence primer binding results from Primer Prospector
- Overall percentage of bacterial sequences missed by the primer pair

## How to apply

Start with assembled and curated 16S rRNA gene sequences (the study used 1,543 bacterial 16S genes ≥800 bp, curated to eliminate assembly errors). De-redundantize by clustering at 97% sequence identity with USEARCH cluster_smallmem using query_cov 0.50 and target_cov 0.50, retaining only representatives that span the 515-806 region of the E. coli 16S rRNA gene. Assess binding of 515F and 806R to each retained sequence using Primer Prospector with default parameters. Aggregate per-sequence binding outcomes into per-lineage evasion rates (Microgenomates, Saccharibacteria, WWE3, WS6) and compute the overall percentage of all bacterial sequences missed. The rationale is that divergent 16S sequences in the candidate phyla radiation (CPR) make amplicon surveys systematically underestimate these organisms.

## Related tools

- **USEARCH** (Clusters curated 16S rRNA gene sequences at 97% identity (cluster_smallmem, query_cov 0.50, target_cov 0.50) to remove redundancy before primer binding analysis)
- **Primer Prospector** (Assesses 515F/806R primer binding against each de-redundant 16S sequence with default parameters)

## Examples

```
usearch -cluster_smallmem curated16s.fa -id 0.97 -query_cov 0.50 -target_cov 0.50 -centroids nr16s.fa -uc clusters.uc
```

## Evaluation signals

- Computed evasion rates reproduce the reported pattern: ~50% Microgenomates, ~50% Saccharibacteria, 60% WWE3, and 100% WS6 evading 515F/806R
- Overall evasion across all bacteria detected in the study is ~20%
- All retained cluster representatives span the 515-806 region of the E. coli 16S rRNA gene; incomplete sequences excluded
- Insertions ≥10 bp (introns, reported in 31% of CPR 16S genes) are removed before binding analysis
- Clustering parameters match: 97% identity with query_cov 0.50 and target_cov 0.50

## Limitations

- Evasion rates are specific to the 515F/806R primer pair and to the bacterial community sampled (Rifle, Colorado aquifer); other primers or environments may differ
- Only sequences spanning the 515-806 region can be evaluated; divergent or partial sequences are excluded, potentially biasing estimates
- CPR 16S genes frequently contain large insertions (31% carry ≥10 bp insertions, up to 2,004 bp) that must be removed or binding analysis will be confounded

## Evidence
