---
name: phylogenetic-distance-weighting
description: Use when 16S marker gene OTUs must be mapped to predicted gene families
  (KO/COG) for organisms without reference genomes, and a phylogenetic reference tree
  with annotated tips (e.g. Greengenes + IMG) is available.
when_to_use_negative:
- The target organism's genome is already present in the IMG reference table, so observed
  gene content should be used directly instead of inferred
- Inference concerns viral or eukaryotic community components, since only bacterial
  and archaeal 16S marker sequences are included
- The marker gene sequences of the strains in question are identical, so strain-level
  variation cannot be distinguished
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: PICRUSt
  role: Implements the gene content inference step that weights extant and inferred
    ancestral genomes exponentially by reciprocal phylogenetic distance
  repo: http://picrust.github.com
- name: APE R library v2.8
  role: Provides ACE ML, ACE REML and ACE PIC ancestral state reconstruction of gene
    family states at internal nodes of the pruned tree
- name: COUNT package v11.0502
  role: Alternative ancestral state reconstruction method via Wagner Parsimony for
    inferring gene family states
- name: tax2tree
  role: Source of the 16S Greengenes reference tree version used to anchor the phylogenetic
    distances
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:94bdb7944682da6040e977ba028a60fef84b1bfd47ec33d665474780ee68874d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - APE R library
  - APE R library v2.8
  - COUNT package
  - COUNT package v11.0502
  - PICRUSt
  - tax2tree
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# phylogenetic-distance-weighting

## Summary

Predicts gene family content for organisms lacking sequenced reference genomes by averaging extant and ancestral (ancestral-state-reconstructed) genome contents on a marker gene tree, weighting each contributor exponentially by the reciprocal of its phylogenetic distance to the target tip — the core PICRUSt gene content inference mechanism.

## When to use

Use when 16S marker gene OTUs must be mapped to predicted gene families (KO/COG) for organisms without reference genomes, and a phylogenetic reference tree with annotated tips (e.g. Greengenes + IMG) is available.

## When NOT to use

- The target organism's genome is already present in the IMG reference table, so observed gene content should be used directly instead of inferred
- Inference concerns viral or eukaryotic community components, since only bacterial and archaeal 16S marker sequences are included
- The marker gene sequences of the strains in question are identical, so strain-level variation cannot be distinguished

## Inputs

- 16S 'tax2tree' version of the Greengenes marker gene reference tree
- Gene-by-genome table of KEGG Ortholog (KO) and COG annotations from IMG v3.5

## Outputs

- Predicted gene family content vectors for every tip in the marker reference tree
- Pre-calculated tab-delimited file of predicted gene family abundances and 16S copy number predictions distributed with PICRUSt

## How to apply

Load the 16S 'tax2tree' version of the Greengenes tree and the gene-by-genome KO/COG annotation table from IMG v3.5, and prune the tree so every tip with a sequenced genome carries its gene family content vector. Apply an ancestral state reconstruction method to the pruned tree — ACE ML, ACE REML or ACE PIC from the APE R library (v2.8), or Wagner Parsimony from the COUNT package (v11.0502) — to infer gene family states at internal nodes. Then generate the predicted gene content for each unsequenced tip as an average of the contents of extant and inferred ancestral genomes, weighting each contributor exponentially by the reciprocal of its phylogenetic distance to the tip. This exploits the correlation between gene content and phylogeny while producing quantified uncertainty for poorly constrained tips.

## Related tools

- **PICRUSt** (Implements the gene content inference step that weights extant and inferred ancestral genomes exponentially by reciprocal phylogenetic distance) — http://picrust.github.com
- **APE R library v2.8** (Provides ACE ML, ACE REML and ACE PIC ancestral state reconstruction of gene family states at internal nodes of the pruned tree)
- **COUNT package v11.0502** (Alternative ancestral state reconstruction method via Wagner Parsimony for inferring gene family states)
- **tax2tree** (Source of the 16S Greengenes reference tree version used to anchor the phylogenetic distances)

## Evaluation signals

- Predicted genome contents for all tips must be consistent with the exponential reciprocal phylogenetic-distance weighting scheme over extant and ancestral genomes
- The output covers all tips in the marker reference tree (a pre-calculated file is distributed with PICRUSt containing predicted genome contents for all tips)
- Leave-one-genome-out holdout accuracy on IMG genomes should approach the reported values: mean 0.95 +/- 0.05 s.d. for bacteria (n=2,487) and 0.94 +/- 0.04 s.d. for archaea (n=103)
- Accuracy should be highest for 'housekeeping' functions such as genetic information processing (mean 0.99 +/- 0.03 s.d.) and lower for laterally transferred categories such as environmental information processing (0.95 +/- 0.04 s.d.)
- Tips with distant or missing close reference genomes should carry wide confidence intervals reflecting high prediction uncertainty

## Limitations

- Some instances of gene gain/loss or laterally transferred genes will be poorly predicted, with wide confidence intervals indicating a high degree of uncertainty
- Accuracy depends on the availability of related reference genomes; phylogenetic gaps in the reference database require additional genome sequences for accurate prediction
- PICRUSt cannot distinguish variation at the strain level if the marker gene sequence used is identical among strains

## Evidence
