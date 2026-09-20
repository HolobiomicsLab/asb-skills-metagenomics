---
name: 16s-copy-number-estimation
description: Use when inferring gene family content or 16S copy number for unsequenced
  organisms on a Greengenes/IMG 16S reference tree, or normalizing 16S OTU tables
  by predicted copy number.
when_to_use_negative:
- The organisms of interest already have complete reference genomes — use their direct
  IMG/KEGG genome annotations instead of inferring traits
- You need viral or eukaryotic metagenome components — PICRUSt only covers bacterial
  and archaeal 16S sequences
- Marker gene sequences are identical among strains — the method cannot distinguish
  strain-level variation
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: PICRUSt
  role: Software package implementing the extended ancestral-state reconstruction
    workflow and distributing the pre-calculated predicted gene content and 16S copy
    number file
  repo: http://picrust.github.com
- name: APE R library v2.8
  role: Provides the ACE ML, ACE REML and ACE PIC ancestral state reconstruction methods
    used to infer gene family states at internal nodes
- name: COUNT package v11.0502
  role: Provides the Wagner Parsimony alternative for ancestral state reconstruction
    of gene families
- name: tax2tree
  role: Source of the 16S-consistent formatting of the Greengenes marker gene reference
    tree used as input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:59cf492856bf27bdd9872a17aec2500cfd841908b21d5b57fadb3fdd2adb1d39
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
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# 16s-copy-number-estimation

## Summary

An extended ancestral state reconstruction method (PICRUSt) that predicts gene family content and 16S rRNA copy number for all tips on a 16S marker gene reference tree, enabling metagenome prediction for organisms without sequenced genomes.

## When to use

Use when inferring gene family content or 16S copy number for unsequenced organisms on a Greengenes/IMG 16S reference tree, or normalizing 16S OTU tables by predicted copy number.

## When NOT to use

- The organisms of interest already have complete reference genomes — use their direct IMG/KEGG genome annotations instead of inferring traits
- You need viral or eukaryotic metagenome components — PICRUSt only covers bacterial and archaeal 16S sequences
- Marker gene sequences are identical among strains — the method cannot distinguish strain-level variation

## Inputs

- 16S 'tax2tree' version of the Greengenes marker gene reference tree
- Gene-by-genome table of KEGG Ortholog (KO) and COG annotations from IMG v3.5
- Pruned marker gene tree with tips labeled by sequenced reference genome
- QIIME 1.5.0-dev closed-reference OTU table (for downstream 16S copy-number normalization)

## Outputs

- Pre-calculated tab-delimited file with predicted gene family abundances for all tips in the marker reference tree
- Predicted 16S copy number per tip, formatted for distribution with the PICRUSt package
- Ancestral gene family state estimates (and confidence intervals) at internal nodes
- Copy-number-normalized OTU table for downstream metagenome prediction

## How to apply

Load the 16S 'tax2tree' version of the Greengenes marker gene tree and the gene-by-genome table of KEGG Ortholog (KO) and COG annotations from IMG v3.5, then format and prune the tree so every tip with a sequenced reference genome carries its gene family content vector. Apply ancestral state reconstruction to the pruned tree — ACE ML, ACE REML or ACE PIC from the APE R library (v2.8), or Wagner Parsimony from the COUNT package (v11.0502) — to infer gene family states at internal nodes. Generate the predicted content (including 16S copy number) for every tip as an average of the contents of extant and inferred ancestral genomes, weighting each contributor exponentially by the reciprocal of its phylogenetic distance to the tip. Write the result as the pre-calculated tab-delimited file distributed with the PICRUSt package; downstream, these 16S copy number predictions normalize the OTU table before metagenome prediction. Quantify uncertainty (confidence intervals, e.g. Supplemental Fig. 7) and use NSTI to flag samples with poor reference coverage.

## Related tools

- **PICRUSt** (Software package implementing the extended ancestral-state reconstruction workflow and distributing the pre-calculated predicted gene content and 16S copy number file) — http://picrust.github.com
- **APE R library v2.8** (Provides the ACE ML, ACE REML and ACE PIC ancestral state reconstruction methods used to infer gene family states at internal nodes)
- **COUNT package v11.0502** (Provides the Wagner Parsimony alternative for ancestral state reconstruction of gene families)
- **tax2tree** (Source of the 16S-consistent formatting of the Greengenes marker gene reference tree used as input)

## Examples

```
R: ace(trait_vector, pruned_16S_tree, method = "REML", CI = TRUE)  # APE v2.8 ACE REML as used by PICRUSt gene content inference
```

## Evaluation signals

- Output covers all tips in the marker reference tree, not only genomes with sequenced reference data
- Predicted genome contents are consistent with the exponential reciprocal phylogenetic-distance weighting scheme (each extant and ancestral contributor weighted by 1/distance)
- Predicted 16S copy numbers are usable to normalize the OTU table as the first step of metagenome prediction
- Uncertainty is quantified per prediction (confidence intervals; wide CIs flag genes with poor support)
- NSTI values are computed per sample and correctly summarize the phylogenetic distance of community members to the nearest reference genome

## Limitations

- Some instances of gene gain, loss or lateral transfer will be poorly predicted, yielding broad confidence intervals (e.g. Supplemental Fig. 7)
- Accuracy depends on availability of related reference genomes; phylogenetic 'gaps' for some environments require additional genome sequences
- Only bacterial and archaeal 16S marker gene sequences are included — viral and eukaryotic components are not inferred
- Prediction requires NSTI (or a similar measure) to quantify expected accuracy; NSTI varies across environments with reference database coverage

## Evidence

- [discussion] A prediction of gene content is generated using an average of the contents of extant and inferred ancestral genomes, weighted exponentially by the reciprocal of phylogenetic distance: "A prediction of gene content is generated using an average of the contents of extant and inferred ancestral genomes, weighted exponentially by the reciprocal of phylogenetic distance"
- [discussion] First, PICRUSt normalizes the OTU table by the 16S copy number predictions: "First, PICRUSt normalizes the OTU table by the 16S copy number predictions"
- [discussion] An ancestral state reconstruction method is then applied to the pruned tree: "An ancestral state reconstruction method is then applied to the pruned tree"
- [abstract] PICRUSt uses an extended ancestral-state reconstruction algorithm to predict which gene families are present: "PICRUSt uses an extended ancestral-state reconstruction algorithm to predict which gene families are present"
- [readme] The primary application is to predict gene family abundance (e.g. the metagenome) in environmental DNA samples for which only marker gene (e.g. 16S rRNA gene) data are available.: "The primary application is to predict gene family abundance (e.g. the metagenome) in environmental DNA samples for which only marker gene (e.g. 16S rRNA gene) data are available."
