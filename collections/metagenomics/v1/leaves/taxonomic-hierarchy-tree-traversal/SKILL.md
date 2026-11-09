---
name: taxonomic-hierarchy-tree-traversal
description: Use when GCN (or another trait) must be mapped onto the Greengenes phylogeny
  or taxonomic tree to test phylogenetic signal or infer trait values for taxa lacking
  sequenced relatives.
when_to_use_negative:
- The input community file already contains measured GCN per OTU and only abundance
  correction (not trait inference) is needed — use CopyRighter correction directly
  instead
- The taxonomy is not Greengenes; CopyRighter currently ships GCN estimates only for
  the Oct 2012 Greengenes taxonomy and does not yet cover Silva or other taxonomies
- The analysis requires trait values for fungal ITS or GC percent bias — only 16S
  rRNA GCN and genome length pre-computed estimates are available
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0084
tools:
- name: CopyRighter
  role: Provides the pre-computed Greengenes GCN data files (OTU-ID-level and taxstring-level)
    that are mapped onto the phylogenetic and taxonomic trees
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl library used to load and manipulate the Greengenes community/tree data
    during traversal
- name: Perl >= 5.10.0
  role: Runtime environment for the tree-traversal and lambda-estimation scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:a5ab00ed129b92e87ea7fbdc0bb02d68504fc40cecb68f6972d31d1a6bcff9bc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Perl >= 5.10.0
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# taxonomic-hierarchy-tree-traversal

## Summary

Traverse the Greengenes phylogenetic tree and taxonomic hierarchy to map 16S rRNA gene copy number (GCN) estimates onto tips and internal nodes, enabling quantification of phylogenetic signal (Pagel's lambda) and trait inference for uncultured lineages.

## When to use

Use when GCN (or another trait) must be mapped onto the Greengenes phylogeny or taxonomic tree to test phylogenetic signal or infer trait values for taxa lacking sequenced relatives.

## When NOT to use

- The input community file already contains measured GCN per OTU and only abundance correction (not trait inference) is needed — use CopyRighter correction directly instead
- The taxonomy is not Greengenes; CopyRighter currently ships GCN estimates only for the Oct 2012 Greengenes taxonomy and does not yet cover Silva or other taxonomies
- The analysis requires trait values for fungal ITS or GC percent bias — only 16S rRNA GCN and genome length pre-computed estimates are available

## Inputs

- CopyRighter-provided Greengenes data file of 16S rRNA gene copy number estimates for all OTUs (tab-delimited, ID + 16S rRNA count columns)
- CopyRighter-provided Greengenes data file of taxon-level GCN estimates (tab-delimited taxstring + 16S rRNA count columns, e.g. 'k__Bacteria; p__Proteobacteria; ...')
- Greengenes phylogenetic tree (Newick)
- Greengenes taxonomic tree derived from Greengenes taxonomic strings

## Outputs

- phylosignal_lambda.tsv containing lambda and P value pairs (phylogeny: lambda = 0.844, P = 2.13e-176; taxonomy: lambda = 0.546, P = 2.03e-38)
- Trait (GCN) values assigned to tree tips and internal nodes

## How to apply

Load the CopyRighter-provided Greengenes data files of GCN estimates for all OTUs and taxa together with the Greengenes phylogenetic tree using Perl with Bio::Community. Map the empirical GCN values onto the tips and internal nodes of the phylogeny, then estimate Pagel's lambda by maximum likelihood and compute the likelihood-ratio P value testing lambda against zero. Repeat the maximum-likelihood lambda estimation on the Greengenes taxonomic tree using taxon-level GCN values to compare signal strength between the full phylogeny and the simplified taxonomy. The rationale is that since related species have similar GCN, tree traversal supports reliable inference of copy number for uncultured organisms from phylogeny or tree-based taxonomy.

## Related tools

- **CopyRighter** (Provides the pre-computed Greengenes GCN data files (OTU-ID-level and taxstring-level) that are mapped onto the phylogenetic and taxonomic trees) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl library used to load and manipulate the Greengenes community/tree data during traversal)
- **Perl >= 5.10.0** (Runtime environment for the tree-traversal and lambda-estimation scripts)

## Examples

```
perl -MBio::Community -e 'map GCN onto Greengenes tree; estimate Pagel lambda by ML' ; write phylosignal_lambda.tsv
```

## Evaluation signals

- Computed lambda on the Greengenes phylogenetic tree matches 0.844 within numerical tolerance
- Computed lambda on the Greengenes taxonomic tree matches 0.546 within numerical tolerance
- Likelihood-ratio P values are of the same extreme magnitude as 2.13e-176 (phylogeny) and 2.03e-38 (taxonomy)
- Taxonomic-tree lambda (0.546) is weaker than phylogenetic-tree lambda (0.844), consistent with the simplified taxonomy retaining less signal
- Output file phylosignal_lambda.tsv contains both lambda/P pairs in the expected schema

## Limitations

- GCN estimates are based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy; other taxonomies (e.g. Silva) are not yet covered
- The phylogenetic signal is weaker on the simplified taxonomic tree (lambda = 0.546 vs 0.844), so taxonomic-level inference is less reliable than phylogenetic-level inference
- CopyRighter is currently limited to 16S rRNA gene GCN; genome-length is also provided but GC percent and fungal ITS correction await pre-computed estimates

## Evidence

- [other] The lambda statistic for empirical GCN is 0.844 (P = 2.13e-176) on the Greengenes phylogenetic tree and 0.546 (P = 2.03e-38) on the Greengenes taxonomic tree: [evidence span withheld — anchor status: not_found]
- [intro] Since related species have similar GCN [8,9], it is often possible to accurately estimate GCN of an uncultured organism if a closely related sequenced relative exists: "Since related species have similar GCN [8,9], it is often possible to accurately estimate GCN of an uncultured organism if a closely related sequenced relative exists"
- [discussion] We have produced freely available data files of the phylogenetically-based estimates of GCN for all OTUs and taxa in the Greengenes database.: "We have produced freely available data files of the phylogenetically-based estimates of GCN for all OTUs and taxa in the Greengenes database."
- [readme] The script bc_use_repr_id of Bio::Community can help in replacing arbitrary OTU IDs by their corresponding Greengenes ID.: [evidence span withheld — anchor status: external_doc]
- [readme] Data files of 16S rRNA gene copy number and genome length (based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy) are distributed with CopyRighter.: "Data files of 16S rRNA gene copy number and genome length (based on IMG
        4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy) are
        distributed with CopyRighter."
