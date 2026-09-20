---
name: ani-based-strain-refinement
description: Use when a MAG-level taxonomy classifier (e.g. kMetaShot) assigns MAGs
  only to a species/genus because the corresponding mock-community or reference genomes
  lack strain annotation, and strain-level resolution is required for benchmarking.
when_to_use_negative:
- When the classifier already resolved strain-level labels for the MAG (no unlabelled
  genomes to refine)
- When candidate genomes share ANI <97%, below the strain-delineation threshold used
  in the article — assignment would be unreliable
- When the goal is genus- or species-level benchmarking only, where strain refinement
  is unnecessary
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Mash
  role: Computes pairwise ANI values between MAGs and RefSeq strain genomes to pick
    the highest-ANI strain assignment
- name: kMetaShot
  role: Produces the initial strain/species-level classifications that ANI refinement
    resolves where reference strain annotation is missing
  repo: https://github.com/gdefazio/kMetaShot
- name: MetaBAT2
  role: Bins contigs into the MAGs that are the input to classification and refinement
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:ce3213714d4cbd563da0a199f6417265ebf1b0025de9987e13a577d6eea06eb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - CAMITAX
  - FastQC
  - GTDBtk
  - Mash
  - MegaHIT
  - MetaBAT2
  - Sickle
  - kMetaShot
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ANI-based strain refinement

## Summary

Refines strain-level taxonomic assignments of MAGs by comparing them against all RefSeq genomes of observed strains with Mash ANI, assigning each unlabelled MAG to the strain with the highest ANI (≥97%). This resolves cases where the classifier cannot distinguish closely related strains lacking reference strain annotation.

## When to use

Use when a MAG-level taxonomy classifier (e.g. kMetaShot) assigns MAGs only to a species/genus because the corresponding mock-community or reference genomes lack strain annotation, and strain-level resolution is required for benchmarking.

## When NOT to use

- When the classifier already resolved strain-level labels for the MAG (no unlabelled genomes to refine)
- When candidate genomes share ANI <97%, below the strain-delineation threshold used in the article — assignment would be unreliable
- When the goal is genus- or species-level benchmarking only, where strain refinement is unnecessary

## Inputs

- Binned MAG FASTA files (e.g. MetaBAT2 output, --maxP 99 --minS 98 -m 1500)
- Preliminary kMetaShot classifications with unresolved strain labels (ass2ref 0.2)
- RefSeq genomes of the observed strains in the mock community
- Mash sketches of MAGs and candidate RefSeq strain genomes

## Outputs

- Refined strain-level assignments per MAG (highest-ANI strain, ANI >=97%)
- Squared pairwise ANI matrices rendered as heat maps
- Recomputed evaluation metrics (sensitivity, precision, FDR, balanced accuracy, F1-score) before and after refinement

## How to apply

For each MAG whose strain-level assignment is unresolved — in the kMetaShot mock-community benchmark, 47 of the 64 mock genomes lacked strain annotation — retrieve all RefSeq genomes of the observed strains and compute pairwise ANI with Mash. Assign the MAG to the strain with the highest ANI, accepting the call only when ANI ≥97%. The article also uses squared ANI matrices for DBSCAN clustering at 97% during reference construction, and removes genomes with ANI <50% as outliers. Render the squared ANI matrices as heat maps to inspect clustering, then recompute sensitivity, precision, FDR, balanced accuracy, and F1-score with refined strain calls (e.g. kMetaShot strain-level sensitivity rose from ~82.0% on HMP genomes) to quantify the effect of refinement.

## Related tools

- **Mash** (Computes pairwise ANI values between MAGs and RefSeq strain genomes to pick the highest-ANI strain assignment)
- **kMetaShot** (Produces the initial strain/species-level classifications that ANI refinement resolves where reference strain annotation is missing) — https://github.com/gdefazio/kMetaShot
- **MetaBAT2** (Bins contigs into the MAGs that are the input to classification and refinement)

## Examples

```
mash dist mag.msh refseq_strains.msh | sort -g -k3 | head -n 1  # highest-ANI strain; accept if ANI >= 0.97
```

## Evaluation signals

- All strain assignments meet the ANI >=97% acceptance threshold
- Squared ANI matrix heat maps show MAGs clustering with their assigned strain genomes
- Post-refinement metrics (sensitivity, precision, FDR, balanced accuracy, F1-score) computed against the known community composition (48 bacteria and 16 archaea) improve or remain consistent versus pre-refinement calls
- Refined assignments are consistent across MAGs from both metaSPAdes and MegaHIT assemblies, reflecting kMetaShot's meta-assembler independence

## Limitations

- 47 of the 64 mock-community genomes lacked strain annotation, so refinement is required only for those; genomes with ANI <50% were considered outliers and removed from subsequent analysis
- kMetaShot requires at least 32 Gb RAM to load its reference, constraining where classification plus refinement can run
- The Docker container needs --shm-size=22g to run the kMetaShot reference properly

## Evidence

- [methods] For species lacking strain annotation (47 of the 64 mock genomes), refine strain assignments by comparing MAGs against all RefSeq genomes of the observed strains with Mash ANI: [evidence span withheld — anchor status: not_found]
- [methods] assigning the strain with the highest ANI (>=97%), rendering the squared ANI matrices as heat maps: [evidence span withheld — anchor status: not_found]
- [methods] squared matrices collecting pairwise ANI values between genomes are used to perform clustering, using DBSCAN: "squared matrices collecting pairwise ANI values between genomes are used to perform clustering, using DBSCAN"
- [methods] genomes that obtained an ANI <50% were considered outliers and removed from subsequent analysis: "genomes that obtained an ANI <50% were considered outliers and removed from subsequent analysis"
- [readme] ass2ref is a ratio between the number of MAG minimizers and the reference minimizers related to the assigned strain: "ass2ref is a ratio between the number of MAG minimizers and the reference minimizers related to the assigned strain"
