---
name: ancestral-state-reconstruction-gene-content
description: Use when you must infer gene content (e.g. KO/COG abundances, 16S copy
  number) for organisms without sequenced reference genomes from a 16S phylogenetic
  tree plus a gene-by-genome annotation table, as required to build PICRUSt reference
  files.
when_to_use_negative:
- The input is already shotgun metagenome data annotated to KOs (e.g. via HUMAnN)
  — no gene content prediction is needed; this step is only for marker-gene-only samples.
- The marker gene tree lacks tips with known reference genome gene content, so ancestral
  states cannot be anchored by extant genomes.
- The trait of interest is dominated by lateral gene transfer (e.g. some environmental
  information processing genes), where prediction will be poor with broad confidence
  intervals.
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: APE R library
  role: Provides the ACE ML, ACE REML and ACE PIC ancestral state reconstruction methods
    used to infer gene family states at internal nodes of the pruned marker gene tree
- name: COUNT package
  role: Alternative engine for ancestral state reconstruction via Wagner Parsimony
    on the gene-by-genome table
- name: PICRUSt
  role: Package into which the pre-calculated predicted gene content and 16S copy
    number file is distributed and consumed by downstream metagenome prediction
  repo: http://picrust.github.com
- name: tax2tree
  role: Produces the 16S tax2tree version of the Greengenes tree used as the marker
    gene reference tree input
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:5626756240758ffd8d3a949c812539535211d932ad2aa901af9a7f36a837e42c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - APE R library
  - COUNT package
  - Greengenes
  - HUMAnN
  - PICRUSt
  - QIIME 1.5.0-dev
  - STAMP
  - tax2tree
  - uclust
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ancestral-state-reconstruction-gene-content

## Summary

Predicts gene family content (KO/COG) and 16S copy number for every tip of a marker-gene (Greengenes 16S) reference tree by applying ancestral state reconstruction (APE ACE ML/REML/PIC or COUNT Wagner Parsimony) to IMG-derived gene-by-genome annotations and averaging extant plus ancestral genomes weighted exponentially by the reciprocal of phylogenetic distance.

## When to use

Use when you must infer gene content (e.g. KO/COG abundances, 16S copy number) for organisms without sequenced reference genomes from a 16S phylogenetic tree plus a gene-by-genome annotation table, as required to build PICRUSt reference files.

## When NOT to use

- The input is already shotgun metagenome data annotated to KOs (e.g. via HUMAnN) — no gene content prediction is needed; this step is only for marker-gene-only samples.
- The marker gene tree lacks tips with known reference genome gene content, so ancestral states cannot be anchored by extant genomes.
- The trait of interest is dominated by lateral gene transfer (e.g. some environmental information processing genes), where prediction will be poor with broad confidence intervals.

## Inputs

- 16S 'tax2tree' version of the Greengenes marker gene reference tree
- Gene-by-genome table of KEGG Ortholog (KO) and COG annotations from IMG v3.5
- Taxonomic labels at phylum and genus level per genome from NCBI Taxonomy (optional, for aggregating holdout accuracies)

## Outputs

- Pre-calculated tab-delimited file of predicted gene family abundances (KO/COG) for all tips in the marker reference tree, formatted for distribution with the PICRUSt package
- Per-tip 16S copy number predictions
- Per-genome and per-functional-category holdout accuracy tables (e.g. genome_holdout_accuracies.csv)

## How to apply

Load the 16S 'tax2tree' version of the Greengenes reference tree and the gene-by-genome table of KEGG Ortholog (KO) and COG annotations from IMG v3.5. Prune the tree so every tip with a sequenced reference genome carries its gene family content vector, then apply ancestral state reconstruction — ACE ML, ACE REML or ACE PIC from the APE R library (v2.8), or Wagner Parsimony from the COUNT package (v11.0502) — to infer gene family states at internal nodes. For each tip, generate predicted gene content as an average of the contents of extant and inferred ancestral genomes, weighting each contributor exponentially by the reciprocal of its phylogenetic distance to the tip. Emit a pre-calculated tab-delimited file of predicted gene family abundances and 16S copy number predictions for all tips in the marker reference tree, formatted for distribution with the PICRUSt package. Validate via leave-one-genome-out holdout (expected mean accuracy ~0.95 for bacteria, ~0.94 for archaea) and note that laterally transferred gene categories (e.g. environmental information processing) score lower (~0.95) than housekeeping functions (~0.99).

## Related tools

- **APE R library** (Provides the ACE ML, ACE REML and ACE PIC ancestral state reconstruction methods used to infer gene family states at internal nodes of the pruned marker gene tree)
- **COUNT package** (Alternative engine for ancestral state reconstruction via Wagner Parsimony on the gene-by-genome table)
- **PICRUSt** (Package into which the pre-calculated predicted gene content and 16S copy number file is distributed and consumed by downstream metagenome prediction) — http://picrust.github.com
- **tax2tree** (Produces the 16S tax2tree version of the Greengenes tree used as the marker gene reference tree input)

## Examples

```
Rscript -e "library(ape); ace(gene_trait_vector, pruned_16S_tree, type='discrete', method='ML')"
```

## Evaluation signals

- Output file covers all tips in the marker reference tree and is formatted for distribution with the PICRUSt package.
- Predicted genome contents are consistent with the exponential reciprocal phylogenetic-distance weighting scheme (average of extant and inferred ancestral genome contents).
- Leave-one-genome-out holdout over the IMG v3.5 table yields mean accuracy ~0.95 +/- 0.05 s.d. for bacteria (n=2,487) and ~0.94 +/- 0.04 s.d. for archaea (n=103).
- Per-category accuracies match reported ranges: genetic information processing ~0.99 +/- 0.03 s.d., environmental information processing ~0.95 +/- 0.04 s.d.

## Limitations

- Some instances of gene gain/loss or laterally transferred genes will be poorly predicted, with broad confidence intervals.
- Prediction accuracy depends on phylogenetic proximity of reference genomes to the sampled organisms; NSTI quantifies expected error, and gaps in the reference database for some environments require additional genomes.
- Strain-level variation cannot be distinguished if the marker gene sequence used is identical among strains.

## Evidence
