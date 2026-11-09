---
name: gene-copy-number-tree-annotation
description: Use when you have GCN trait estimates for Greengenes OTUs/taxa plus the
  Greengenes phylogeny or taxonomy, and need to quantify how strongly GCN tracks phylogeny
  (e.g. Pagel's lambda) before inferring GCN for unsequenced lineages.
when_to_use_negative:
- You want to correct an amplicon or metagenomic abundance profile — that is CopyRighter's
  correction mode, not the trait-onto-tree annotation/phylogenetic-signal estimation
  described here
- Your trait estimates come from a taxonomy other than Greengenes (e.g. Silva); the
  article notes CopyRighter does not yet cover other taxonomies and extension is only
  anticipated
- Your trait values lack a phylogenetic or taxonomic tree structure to map them onto
  — lambda estimation requires a tree
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3174
tools:
- name: CopyRighter
  role: Provides the pre-computed phylogenetically-based GCN estimate data files for
    all Greengenes OTUs and taxa that are mapped onto the tree and used as the trait
    input for lambda estimation
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl library used with CopyRighter data files to load and handle Greengenes
    OTU/taxon community and trait records when mapping GCN onto the tree
- name: Perl
  role: Scripting environment (>= 5.10.0) in which the tree-annotation and lambda-estimation
    workflow is implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:2e3bd24423537bb33f8a54319023fcfbf403474d79011d8dc820b6039ce2c222
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Perl
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-copy-number-tree-annotation

## Summary

Maps CopyRighter's phylogenetically-based 16S rRNA gene copy number (GCN) estimates onto the Greengenes phylogenetic tree and taxonomic tree, then quantifies the strength of phylogenetic signal with Pagel's lambda to justify tree-based GCN inference.

## When to use

Use when you have GCN trait estimates for Greengenes OTUs/taxa plus the Greengenes phylogeny or taxonomy, and need to quantify how strongly GCN tracks phylogeny (e.g. Pagel's lambda) before inferring GCN for unsequenced lineages.

## When NOT to use

- You want to correct an amplicon or metagenomic abundance profile — that is CopyRighter's correction mode, not the trait-onto-tree annotation/phylogenetic-signal estimation described here
- Your trait estimates come from a taxonomy other than Greengenes (e.g. Silva); the article notes CopyRighter does not yet cover other taxonomies and extension is only anticipated
- Your trait values lack a phylogenetic or taxonomic tree structure to map them onto — lambda estimation requires a tree

## Inputs

- CopyRighter-provided Greengenes data file of 16S rRNA gene copy number estimates for all OTUs (tab-delimited ID/trait file, e.g. OTU ID 4 -> 1.51098055313977)
- CopyRighter-provided Greengenes data file of taxon-level GCN estimates keyed by Greengenes taxstring (e.g. 'k__Archaea; p__; c__; o__; f__; g__; s__' -> 1.57262)
- Greengenes phylogenetic tree (Newick)
- Greengenes taxonomic tree derived from Greengenes 2012/10 taxonomic strings

## Outputs

- phylosignal_lambda.tsv containing lambda and likelihood-ratio P value pairs for the phylogenetic tree (0.844, 2.13e-176) and taxonomic tree (0.546, 2.03e-38)
- Tree/node-annotated mapping of GCN values onto tips and internal nodes of the Greengenes phylogeny

## How to apply

Load the CopyRighter-distributed Greengenes data files of GCN estimates for all OTUs and taxa together with the Greengenes phylogenetic tree, using Perl with Bio::Community. Map the empirical GCN values onto the tips and internal nodes of the Greengenes phylogeny, then estimate Pagel's lambda by maximum likelihood and compute the likelihood-ratio P value testing lambda against zero. Repeat the maximum-likelihood lambda estimation on the Greengenes taxonomic tree using the taxon-level GCN values (tab-delimited ID/taxstring-to-trait files as shipped with CopyRighter). Compare computed values against the reported references: lambda = 0.844 (P = 2.13e-176) on the phylogeny and lambda = 0.546 (P = 2.03e-38) on the taxonomy. The rationale is that related species have similar GCN, so a strong phylogenetic signal (high lambda, extreme P) validates inferring GCN from phylogeny and tree-based taxonomy, with phylogenetic-level correction preferred for accuracy.

## Related tools

- **CopyRighter** (Provides the pre-computed phylogenetically-based GCN estimate data files for all Greengenes OTUs and taxa that are mapped onto the tree and used as the trait input for lambda estimation) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl library used with CopyRighter data files to load and handle Greengenes OTU/taxon community and trait records when mapping GCN onto the tree)
- **Perl** (Scripting environment (>= 5.10.0) in which the tree-annotation and lambda-estimation workflow is implemented)

## Examples

```
perl -MBio::Community map_gcn_to_tree.pl --data 16S_copies_gg.txt --tree greengenes.tre && perl estimate_lambda.pl --tree greengenes.tre --traits gcn_by_node.txt --out phylosignal_lambda.tsv
```

## Evaluation signals

- Computed lambda on the Greengenes phylogenetic tree matches 0.844 within numerical tolerance
- Computed lambda on the Greengenes taxonomic tree matches 0.546 within numerical tolerance
- Likelihood-ratio P values are of the same extreme magnitude as 2.13e-176 (phylogeny) and 2.03e-38 (taxonomy)
- Output file phylosignal_lambda.tsv contains exactly two lambda/P pairs, one per tree (phylogeny and taxonomy)
- Trait values read from the CopyRighter data file match the expected format: tab-delimited, two columns (ID or taxstring, 16S rRNA count), with comment lines starting with '#' ignored

## Limitations

- CopyRighter currently provides GCN estimates only for the Greengenes taxonomy; extension to other taxonomies such as Silva is anticipated but not yet available
- The taxonomic tree yields a weaker signal (lambda = 0.546) than the full phylogeny (lambda = 0.844), so taxonomy-based GCN inference is less precise than phylogenetic-level inference
- GCN data files distributed with CopyRighter are based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy, so annotation reflects that database snapshot

## Evidence

- [results] The lambda statistic for empirical GCN is 0.844 (P = 2.13e-176) on the Greengenes phylogenetic tree and 0.546 (P = 2.03e-38) on the Greengenes taxonomic tree: [evidence span withheld — anchor status: not_found]
- [discussion] We have produced freely available data files of the phylogenetically-based estimates of GCN for all OTUs and taxa in the Greengenes database.: "We have produced freely available data files of the phylogenetically-based estimates of GCN for all OTUs and taxa in the Greengenes database."
- [readme] Data files of 16S rRNA gene copy number and genome length (based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy) are distributed with CopyRighter.: [evidence span withheld — anchor status: external_doc]
- [intro] Since related species have similar GCN, it is often possible to accurately estimate GCN of an uncultured organism if a closely related sequenced relative exists: "Since related species have similar GCN [8,9], it is often possible to accurately estimate GCN of an uncultured organism if a closely related sequenced relative exists"
