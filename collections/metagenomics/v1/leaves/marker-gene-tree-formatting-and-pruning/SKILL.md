---
name: marker-gene-tree-formatting-and-pruning
description: 'Use when building a PICRUSt-style reference pipeline: a marker gene
  phylogeny (e.g. Greengenes 16S tree) must be aligned with a gene-by-genome KO/COG
  table before ancestral state reconstruction or tip gene content prediction.'
when_to_use_negative:
- The input is a 16S OTU table of sample data rather than a reference genome set —
  downstream PICRUSt steps (copy-number normalization, metagenome prediction) expect
  an already-built, trait-annotated reference tree.
- The community or genomes of interest are viral or eukaryotic — PICRUSt only includes
  bacterial and archaeal marker gene sequences.
- A reference genome already exists for the organism's gene content, making prediction
  via the pruned tree unnecessary.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0661
tools:
- name: PICRUSt
  role: Package whose reference pipeline consumes the formatted/pruned marker gene
    tree and distributes the pre-calculated predicted gene content file
  repo: https://github.com/picrust/picrust
- name: tax2tree
  role: Source of the 16S-decorated Greengenes marker gene reference tree that is
    formatted and pruned
- name: APE R library v2.8
  role: Provides ACE ML, ACE REML and ACE PIC ancestral state reconstruction applied
    to the pruned tree
- name: COUNT package v11.0502
  role: Alternative Wagner Parsimony ancestral state reconstruction on the pruned
    tree
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:24d84ad577ec9590c408147758c2fecbee92afe4ab55596ab39c2d4a2ce2f702
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/picrust/picrust
  tools:
  - APE R library
  - APE R library v2.8
  - COUNT package
  - COUNT package v11.0502
  - PICRUSt
  - tax2tree
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0661
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# marker-gene-tree-formatting-and-pruning

## Summary

Formats and prunes the 16S marker gene reference tree (tax2tree Greengenes) so that every tip with a sequenced reference genome carries its gene family content vector of KEGG Ortholog (KO) and COG annotations, preparing the tree for ancestral state reconstruction in PICRUSt.

## When to use

Use when building a PICRUSt-style reference pipeline: a marker gene phylogeny (e.g. Greengenes 16S tree) must be aligned with a gene-by-genome KO/COG table before ancestral state reconstruction or tip gene content prediction.

## When NOT to use

- The input is a 16S OTU table of sample data rather than a reference genome set — downstream PICRUSt steps (copy-number normalization, metagenome prediction) expect an already-built, trait-annotated reference tree.
- The community or genomes of interest are viral or eukaryotic — PICRUSt only includes bacterial and archaeal marker gene sequences.
- A reference genome already exists for the organism's gene content, making prediction via the pruned tree unnecessary.

## Inputs

- 16S 'tax2tree' version of the Greengenes marker gene reference tree
- gene-by-genome table of KEGG Ortholog (KO) and COG annotations from IMG v3.5
- taxonomic labels for each genome (phylum and genus level, e.g. from NCBI Taxonomy)

## Outputs

- pruned marker gene tree in which every tip with a sequenced reference genome carries its gene family content vector
- pre-calculated tab-delimited file of predicted gene family abundances and 16S copy number predictions for all tips in the marker reference tree, formatted for distribution with PICRUSt

## How to apply

Download the 16S 'tax2tree' version of Greengenes and obtain KEGG Ortholog (KO) and COG gene annotations from IMG v3.5 as a gene-by-genome table. Format the marker gene tree so tip labels match genome identifiers, then prune the tree to the tips that have sequenced reference genomes, attaching each retained tip its gene family content vector. This pruned, trait-annotated tree is the required substrate for ancestral state reconstruction (ACE ML, ACE REML, ACE PIC from APE v2.8, or Wagner Parsimony from COUNT v11.0502), which infers gene family states at internal nodes. Predicted gene content for tips is then computed as an average of extant and ancestral genome contents weighted exponentially by the reciprocal of phylogenetic distance, and distributed as a pre-calculated file with the PICRUSt package.

## Related tools

- **PICRUSt** (Package whose reference pipeline consumes the formatted/pruned marker gene tree and distributes the pre-calculated predicted gene content file) — https://github.com/picrust/picrust
- **tax2tree** (Source of the 16S-decorated Greengenes marker gene reference tree that is formatted and pruned)
- **APE R library v2.8** (Provides ACE ML, ACE REML and ACE PIC ancestral state reconstruction applied to the pruned tree)
- **COUNT package v11.0502** (Alternative Wagner Parsimony ancestral state reconstruction on the pruned tree)

## Evaluation signals

- The pruned tree's tip set exactly matches the tips in the gene-by-genome IMG v3.5 table that have sequenced reference genomes.
- Every retained tip carries a complete gene family content vector of KO and COG annotations.
- The predicted gene content file covers all tips in the marker reference tree (not just sequenced tips).
- Predicted tip contents are consistent with the exponential reciprocal phylogenetic-distance weighting over extant and inferred ancestral genomes.
- Tree topology is unchanged from the downloaded tax2tree Greengenes tree except for pruning to reference-genome tips.

## Limitations

- Reference database has phylogenetic gaps for some environments; additional genome sequences may be needed for accurate prediction.
- Some gene gain/loss events and laterally transferred genes will be poorly predicted, yielding broad confidence intervals.
- No versioned release or changelog information for PICRUSt is provided with the pre-calculated predicted genome contents file.
- Prediction uncertainty should be quantified (e.g. NSTI or confidence intervals) since accuracy depends on phylogenetic proximity of reference genomes.

## Evidence

- [discussion] the 16S 'tax2tree' version of Greengenes45 was downloaded and used for all presented research: [evidence span withheld — anchor status: not_found]
