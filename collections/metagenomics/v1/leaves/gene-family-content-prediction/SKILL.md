---
name: gene-family-content-prediction
description: Use when a workflow needs predicted gene family content for unsequenced
  taxa on a 16S marker gene reference tree, e.g. preparing the pre-calculated PICRUSt
  gene content file before metagenome inference.
when_to_use_negative:
- The taxa of interest already have complete reference genomes — use their actual
  annotated gene content instead of inference.
- 'The community includes viral or eukaryotic components: PICRUSt only includes bacterial
  and archaeal 16S marker gene sequences, so viral/eukaryotic metagenome components
  are not inferred.'
- Strain-level functional variation matters and the marker gene sequence is identical
  among strains — the method cannot distinguish variation at the strain level.
edam_operation: http://edamontology.org/operation_0544
edam_topics:
- http://edamontology.org/topic_3299
- http://edamontology.org/topic_0622
tools:
- name: PICRUSt
  role: Software package that implements the gene content inference step and distributes
    the pre-calculated predicted gene family and 16S copy number file
  repo: http://picrust.github.com
- name: APE R library v2.8
  role: Provides the ACE ML, ACE REML and ACE PIC ancestral state reconstruction methods
    applied to the pruned tree
- name: COUNT package v11.0502
  role: Provides the Wagner Parsimony alternative for inferring gene family states
    at internal nodes
- name: tax2tree
  role: Source of the 16S 'tax2tree' version of Greengenes used as the marker gene
    reference tree
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:6e67164f73466ae46777920ca16dec4f5aaaddac8fc107e57c4e19331a061747
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
  edam_operation: http://edamontology.org/operation_0544
  edam_topics:
  - http://edamontology.org/topic_3299
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# gene-family-content-prediction

## Summary

Predict the gene family (KO/COG) content and 16S copy number of organisms lacking sequenced genomes by applying ancestral state reconstruction to the Greengenes marker gene reference tree and combining extant and ancestral genomes with exponential reciprocal phylogenetic-distance weighting, as implemented in PICRUSt.

## When to use

Use when a workflow needs predicted gene family content for unsequenced taxa on a 16S marker gene reference tree, e.g. preparing the pre-calculated PICRUSt gene content file before metagenome inference.

## When NOT to use

- The taxa of interest already have complete reference genomes — use their actual annotated gene content instead of inference.
- The community includes viral or eukaryotic components: PICRUSt only includes bacterial and archaeal 16S marker gene sequences, so viral/eukaryotic metagenome components are not inferred.
- Strain-level functional variation matters and the marker gene sequence is identical among strains — the method cannot distinguish variation at the strain level.

## Inputs

- 16S 'tax2tree' version of the Greengenes marker gene reference tree
- Gene-by-genome table of KEGG Ortholog (KO) and COG annotations from IMG v3.5
- Phylogenetic distances among tips on the marker gene tree

## Outputs

- Pre-calculated tab-delimited file of predicted gene family (KO) abundances for all tips in the marker reference tree
- Predicted 16S copy number for every tip in the marker reference tree
- Quantified uncertainty (confidence intervals / NSTI basis) associated with predicted genome contents

## How to apply

Load the 16S 'tax2tree' version of the Greengenes tree together with the gene-by-genome table of KEGG Ortholog (KO) and COG annotations obtained from IMG v3.5, then format and prune the marker gene tree so every tip with a sequenced reference genome carries its gene family content vector. Apply an ancestral state reconstruction method to the pruned tree — ACE ML, ACE REML or ACE PIC from the APE R library (v2.8), or Wagner Parsimony from the COUNT package (v11.0502) — to infer gene family states at internal nodes. Generate the predicted gene content for every tip as an average of the contents of extant and inferred ancestral genomes, weighting each contributor exponentially by the reciprocal of its phylogenetic distance to the tip. Finally write the pre-calculated tab-delimited file of predicted gene family abundances and 16S copy number predictions for all tips in the marker reference tree, formatted for distribution with the PICRUSt package.

## Related tools

- **PICRUSt** (Software package that implements the gene content inference step and distributes the pre-calculated predicted gene family and 16S copy number file) — http://picrust.github.com
- **APE R library v2.8** (Provides the ACE ML, ACE REML and ACE PIC ancestral state reconstruction methods applied to the pruned tree)
- **COUNT package v11.0502** (Provides the Wagner Parsimony alternative for inferring gene family states at internal nodes)
- **tax2tree** (Source of the 16S 'tax2tree' version of Greengenes used as the marker gene reference tree)

## Evaluation signals

- Output covers all tips in the marker reference tree (both sequenced and unsequenced).
- Predicted genome contents are consistent with the exponential reciprocal phylogenetic-distance weighting scheme described in the paper.
- Predicted 16S copy numbers accompany the gene family abundances in the tab-delimited pre-calculated file.
- Confidence intervals/NSTI-based uncertainty are quantified, with wide intervals flagging poorly constrained (e.g. laterally transferred) genes.
- Holdout validation: exclude a sequenced genome (or all genomes within a given 16S-tree phylogenetic distance) from the reference gene-by-genome table and confirm prediction accuracy on the held-out genome.

## Limitations

- Laterally transferred genes and some instances of gene gain/loss will be poorly predicted, yielding broad confidence intervals.
- Accuracy depends on the availability of related reference genomes; some environments have phylogenetic 'gaps' in the reference database requiring additional genome sequences.
- The reference set covers only bacterial and archaeal genomes — viral and eukaryotic metagenome components are not inferred.
- The software cannot distinguish variation at the strain level if the marker gene sequence used is identical among strains.

## Evidence
