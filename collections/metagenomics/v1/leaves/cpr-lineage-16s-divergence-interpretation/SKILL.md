---
name: cpr-lineage-16s-divergence-interpretation
description: Use when a metagenomic or 16S amplicon study asks which bacterial lineages
  would be missed by common 16S primers (e.g. 515F/806R), especially for divergent
  CPR phyla like Microgenomates, Saccharibacteria, WWE3, or WS6.
when_to_use_negative:
- The input is already a 16S amplicon feature table or OTU abundance matrix — this
  skill evaluates primer binding on assembled 16S gene sequences, not count data
- The sequences have not had introns/insertions removed — 31% of CPR 16S genes carry
  ≥10 bp insertions (up to 2,004 bp), so unspliced sequences will falsely appear primer-incompatible
- The question is about shotgun-relative abundance of CPR organisms rather than amplicon-survey
  detectability
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: USEARCH
  role: De-redundantizes curated 16S rRNA gene sequences by clustering at 97% identity
    (cluster_smallmem, query_cov 0.50, target_cov 0.50) before primer binding assessment
- name: Primer Prospector
  role: Assesses whether 515F and 806R primers would bind each retained 16S cluster
    representative under default parameters
- name: fix_assembly_errors
  role: Curation of assembled 16S rRNA gene sequences to eliminate assembly errors
    prior to primer binding analysis
  repo: https://github.com/christophertbrown/fix_assembly_errors
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:ba10e9b1781d9b670fa24ecd39392cab0fb5bcc002ef850f45b36f09e5db18d4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/christophertbrown/fix_assembly_errors
  tools:
  - Primer Prospector
  - USEARCH
  - fix_assembly_errors
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cpr-lineage-16s-divergence-interpretation

## Summary

Interprets how divergent 16S rRNA gene sequences in candidate phyla radiation (CPR) lineages cause evasion of amplicon-survey primers, quantifying per-lineage and overall detection loss for primers like 515F/806R from the Brown et al. 2015 Rifle groundwater study.

## When to use

Use when a metagenomic or 16S amplicon study asks which bacterial lineages would be missed by common 16S primers (e.g. 515F/806R), especially for divergent CPR phyla like Microgenomates, Saccharibacteria, WWE3, or WS6.

## When NOT to use

- The input is already a 16S amplicon feature table or OTU abundance matrix — this skill evaluates primer binding on assembled 16S gene sequences, not count data
- The sequences have not had introns/insertions removed — 31% of CPR 16S genes carry ≥10 bp insertions (up to 2,004 bp), so unspliced sequences will falsely appear primer-incompatible
- The question is about shotgun-relative abundance of CPR organisms rather than amplicon-survey detectability

## Inputs

- Curated, assembly-error-corrected bacterial 16S rRNA gene sequences (≥800 bp) from metagenomic assemblies
- Lineage/phylum assignments for each 16S sequence (e.g. Microgenomates, Saccharibacteria, WWE3, WS6)
- Primer sequences for amplicon-survey primers such as 515F and 806R

## Outputs

- primer_binding_analysis.csv with per-lineage primer-evasion percentages (Microgenomates, Saccharibacteria, WWE3, WS6) and overall percentage of bacterial sequences missed
- Per-sequence 515F/806R binding outcome table (bound vs. non-binding) for cluster representatives spanning the 515-806 region

## How to apply

Start from curated assembled 16S rRNA gene sequences (e.g. the 1,543 bacterial 16S genes ≥800 bp from Rifle groundwater metagenomes) and de-redundantize by clustering at 97% sequence identity with USEARCH cluster_smallmem using query_cov 0.50 and target_cov 0.50, retaining only representatives spanning the 515-806 E. coli region. Assess 515F and 806R binding against each retained sequence with Primer Prospector under default parameters. Aggregate per-sequence binding failures into per-lineage evasion rates, being careful to remove rRNA insertions (≥10 bp) first since 31% of CPR 16S genes carry large (mean 519 bp) introns that are spliced out of transcripts. Interpret results against the reported pattern: ~50% Microgenomates, ~50% Saccharibacteria, 60% WWE3, 100% WS6 evasion, and ~20% of all bacteria detected in the study (including organisms outside the CPR) missed by 515F/806R.

## Related tools

- **USEARCH** (De-redundantizes curated 16S rRNA gene sequences by clustering at 97% identity (cluster_smallmem, query_cov 0.50, target_cov 0.50) before primer binding assessment)
- **Primer Prospector** (Assesses whether 515F and 806R primers would bind each retained 16S cluster representative under default parameters)
- **fix_assembly_errors** (Curation of assembled 16S rRNA gene sequences to eliminate assembly errors prior to primer binding analysis) — https://github.com/christophertbrown/fix_assembly_errors

## Examples

```
usearch -cluster_smallmem curated_16s.fa -id 0.97 -query_cov 0.50 -target_cov 0.50 -centroids reps_97.fa
```

## Evaluation signals

- Computed evasion rates reproduce the reported pattern: ~50% Microgenomates, ~50% Saccharibacteria, 60% WWE3, and 100% of WS6 sequences fail 515F/806R binding
- Overall non-binding rate across all bacterial clusters is approximately 20%, including organisms outside the CPR
- Only cluster representatives spanning the 515-806 region of the E. coli 16S rRNA gene are included; incomplete sequences are excluded from the denominator
- Clustering parameters match the paper (97% identity, query_cov 0.50, target_cov 0.50) so redundancy does not bias per-lineage percentages
- Insertions ≥10 bp have been removed before binding assessment so intron-carrying genes are not counted as primer mismatches

## Limitations

- Results are specific to the sampled Rifle, Colorado groundwater community; evasion percentages are 'probable' estimates ('would probably not bind') and may differ for other environments
- Insertion-bearing 16S genes (31% of CPR 16S genes, up to 2,004 bp) must be spliced before analysis, since insertions are not retained in transcribed RNAs
- Primer evasion implies missed detection in typical cultivation-independent surveys but does not quantify organism abundance, which requires shotgun coverage or RNA measurements

## Evidence
