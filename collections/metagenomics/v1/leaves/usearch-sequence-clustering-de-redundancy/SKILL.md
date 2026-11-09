---
name: usearch-sequence-clustering-de-redundancy
description: Use when a curated set of near-full-length 16S rRNA gene sequences (or
  similar amplicon/marker sequences) contains redundant variants and downstream per-sequence
  analyses (e.g. 515F/806R primer binding) should be counted once per ~species-level
  cluster rather than once per assembled copy.
when_to_use_negative:
- The input is already a non-redundant representative set or a pre-clustered OTU/ASV
  table — re-clustering would discard legitimate within-cluster diversity needed for
  primer-binding counts.
- Sequences contain large ≥10 bp intron insertions that have not yet been removed,
  since insertions (present in 31% of CPR 16S rRNA genes here, up to 2,004 bp) would
  corrupt identity-based clustering.
- The goal is phyla-level counting, for which the paper used a 75% sequence identity
  clustering, not the 97% de-redundancy step.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0196
tools:
- name: USEARCH
  role: De-redundantizes curated 16S rRNA gene sequences via cluster_smallmem at 97%
    identity with 0.50 query/target coverage before primer-binding analysis
- name: Primer Prospector
  role: Consumes the 97% cluster representatives to assess 515F/806R primer binding
    per lineage
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:ecd553fa18640fb75b551c87cd41d6a6ab762d44ebccce1d68e8a9b0251bfcc9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Primer Prospector
  - USEARCH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# USEARCH sequence clustering de-redundancy

## Summary

De-redundantize curated sequence sets (here, 1,543 assembled and error-corrected bacterial 16S rRNA genes ≥800 bp) by clustering at 97% sequence identity with USEARCH cluster_smallmem, so that downstream primer-binding and diversity analyses operate on non-redundant cluster representatives.

## When to use

Use when a curated set of near-full-length 16S rRNA gene sequences (or similar amplicon/marker sequences) contains redundant variants and downstream per-sequence analyses (e.g. 515F/806R primer binding) should be counted once per ~species-level cluster rather than once per assembled copy.

## When NOT to use

- The input is already a non-redundant representative set or a pre-clustered OTU/ASV table — re-clustering would discard legitimate within-cluster diversity needed for primer-binding counts.
- Sequences contain large ≥10 bp intron insertions that have not yet been removed, since insertions (present in 31% of CPR 16S rRNA genes here, up to 2,004 bp) would corrupt identity-based clustering.
- The goal is phyla-level counting, for which the paper used a 75% sequence identity clustering, not the 97% de-redundancy step.

## Inputs

- Curated assembled 16S rRNA gene sequences in FASTA (≥800 bp, assembly errors corrected, introns/insertions ≥10 bp removed)
- USEARCH command-line binary

## Outputs

- Non-redundant set of 97%-identity cluster representative sequences (FASTA)
- Cluster membership file mapping each input sequence to its representative
- Downstream primer_binding_analysis.csv of per-lineage 515F/806R evasion rates

## How to apply

Take the curated, intron-stripped assembled 16S rRNA gene sequences and run USEARCH cluster_smallmem with a 97% identity threshold, matching the paper's de-redundancy step ('clustered at 97% sequence identity using USEARCH'), plus query_cov 0.50 and target_cov 0.50 so partial overlapping fragments do not spuriously merge. Retain only cluster representatives; in the paper's primer-evasion workflow these representatives were further filtered to those spanning the 515–806 E. coli 16S region before assessing 515F/806R binding with Primer Prospector. The 97% cutoff is the conventional operational-species threshold, appropriate because evasion rates per lineage (Microgenomates, Saccharibacteria, WWE3, WS6) are reported as fractions of sequence clusters, not raw assemblies. Note that a separate, coarser 75% identity clustering of the same sequences was used for phyla counting — do not mix the two thresholds.

## Related tools

- **USEARCH** (De-redundantizes curated 16S rRNA gene sequences via cluster_smallmem at 97% identity with 0.50 query/target coverage before primer-binding analysis)
- **Primer Prospector** (Consumes the 97% cluster representatives to assess 515F/806R primer binding per lineage)

## Examples

```
usearch -cluster_smallmem curated_16S.fa -id 0.97 -query_cov 0.50 -target_cov 0.50 -uc clusters.uc -centroids representatives.fa
```

## Evaluation signals

- Every input sequence ≥800 bp is assigned to exactly one cluster and appears in the membership output (no orphans dropped silently).
- All pairwise identities between distinct representatives are <0.97 under the 0.50/0.50 coverage constraints.
- The number of representatives spanning the E. coli 515–806 region is consistent with the count of complete-length clusters, with incomplete sequences excluded from the primer-binding denominator.
- Downstream evasion rates reproduce the reported pattern: ~50% of Microgenomates, ~50% of Saccharibacteria, 60% of WWE3, and 100% of WS6 sequences not bound by 515F/806R, and ~20% of all bacterial sequences missed overall.
- The 97% clustering result differs from the separate 75% clustering used for phyla counting — cluster counts at 97% should exceed those at 75%.

## Limitations

- Evasion percentages apply to sequences sampled in this Rifle, Colorado groundwater study; they are not universal estimates for all environments.
- Primer-binding failure does not capture false positives from intron-containing genes unless insertions are removed first; insertions are spliced out of transcribed rRNA and must be stripped from the DNA-derived sequences before clustering.
- cluster_smallmem is approximate (greedy) clustering; representative choice depends on input order, and results can shift with the 0.50 query/target coverage settings.

## Evidence
