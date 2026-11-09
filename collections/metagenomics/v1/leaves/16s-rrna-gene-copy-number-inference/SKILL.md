---
name: 16s-rrna-gene-copy-number-inference
description: Use when a 16S rRNA amplicon or qPCR analysis needs GCN estimates for
  bacterial or archaeal taxa lacking a sequenced genome, and a phylogenetic tree with
  taxonomy (e.g. Greengenes 99%-identity OTUs) is available.
when_to_use_negative:
- The target taxonomy is not Greengenes (e.g. Silva); coverage extension is only anticipated
  in the article
- All taxa of interest already have sequenced genomes with directly measured GCN,
  making phylogenetic inference unnecessary
- Bias in the profile is not due to 16S rRNA gene copy number (e.g. GC percent, genome
  length, or fungal ITS variation, which lack appropriate pre-computed estimates)
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3293
tools:
- name: CopyRighter
  role: Distributes the phylogenetically-based GCN estimate files used for lineage-specific
    correction of amplicon and qPCR profiles
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl library used to load and manipulate the Greengenes OTU tree and taxonomy;
    its bc_use_repr_id script can map OTU IDs to Greengenes IDs
  repo: github.com/fangly/AmpliCopyrighter
- name: Perl >= 5.10.0
  role: Runtime environment in which the CopyRighter preprocessing/GCN inference scripts
    execute
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:351bf0d5879f610ffd5b1432cb8dd3ff1b3d920782ee6f70192301047cdf55e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Perl >= 5.10.0
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# 16s-rrna-gene-copy-number-inference

## Summary

Infers 16S rRNA gene copy number (GCN) of unsequenced Greengenes taxa by mapping curated genome GCN values onto an OTU tree and propagating them with phylogenetically independent contrasts (PIC), since GCN is strongly linked to microbial phylogeny.

## When to use

Use when a 16S rRNA amplicon or qPCR analysis needs GCN estimates for bacterial or archaeal taxa lacking a sequenced genome, and a phylogenetic tree with taxonomy (e.g. Greengenes 99%-identity OTUs) is available.

## When NOT to use

- The target taxonomy is not Greengenes (e.g. Silva); coverage extension is only anticipated in the article
- All taxa of interest already have sequenced genomes with directly measured GCN, making phylogenetic inference unnecessary
- Bias in the profile is not due to 16S rRNA gene copy number (e.g. GC percent, genome length, or fungal ITS variation, which lack appropriate pre-computed estimates)

## Inputs

- Greengenes 99%-identity OTU tree and taxonomy
- Curated genomes with measured 16S rRNA gene copy numbers (e.g. IMG 4.0 genomes)
- Taxonomic strings in Greengenes format (k__; p__; c__; o__; f__; g__; s__)

## Outputs

- Per-OTU and per-taxon GCN estimates table (gcn_estimates.tsv; tab-delimited two columns: ID/taxstring and 16S rRNA count)

## How to apply

Load the Greengenes 99%-identity OTU tree and taxonomy with Bio::Community under Perl (>= 5.10.0). Map each curated genome's measured 16S rRNA copy number onto the leaf of its corresponding OTU in the tree, then reroot the tree as needed and compute ancestral-node GCN estimates with phylogenetically independent contrasts (PIC), exploiting the fact that related species have similar GCN. Aggregate node-level estimates into per-OTU and per-taxon GCN values covering all bacterial and archaeal lineages, and write the table (e.g. gcn_estimates.tsv) as a tab-delimited two-column file of ID or taxonomic string and trait estimate. Validate that the table covers all Greengenes OTUs/taxa and that inferred GCN values fall within the biologically reported range of 1 to 15 in Bacteria and up to 5 in Archaea.

## Related tools

- **CopyRighter** (Distributes the phylogenetically-based GCN estimate files used for lineage-specific correction of amplicon and qPCR profiles) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl library used to load and manipulate the Greengenes OTU tree and taxonomy; its bc_use_repr_id script can map OTU IDs to Greengenes IDs) — github.com/fangly/AmpliCopyrighter
- **Perl >= 5.10.0** (Runtime environment in which the CopyRighter preprocessing/GCN inference scripts execute)

## Examples

```
perl -MBio::Community -e '...'  # preprocessing scripts under Perl >= 5.10.0; estimates shipped as -d <data> file: copyrighter -i otu_table.qiime -o out_copyrighted.txt -d 16S_copies_greengenes.txt -v
```

## Evaluation signals

- The GCN estimates table covers all Greengenes OTUs and taxa (both bacterial and archaeal lineages)
- All inferred GCN values fall within the biologically reported range: 1 to 15 in Bacteria and up to 5 in Archaea
- Values match the distributed CopyRighter data files (IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy), e.g. OTU IDs with counts like 1.51098055313977
- The output file is tab-delimited with two columns (ID/taxonomic string, 16S rRNA count) and correctly ignores empty and comment (#) lines

## Limitations

- CopyRighter currently covers only the Greengenes taxonomy; extension to other taxonomies such as Silva is only anticipated, not implemented
- Since related species have similar GCN, estimates rely on a closely related sequenced relative existing; accuracy degrades for deep branches without close relatives
- The provided estimates are limited to 16S rRNA gene GCN; GC percent, genome length, and fungal ITS corrections are only hypothetical pending appropriate pre-computed estimates

## Evidence

- [intro] PIC on Greengenes tree: "Since related species have similar GCN [8,9], it is often possible to accurately estimate GCN of an uncultured organism if a closely related sequenced relative exists"
- [discussion] Coverage and range validation: "We have produced freely available data files of the phylogenetically-based estimates of GCN for all OTUs and taxa in the Greengenes database."
- [intro] GCN range: "GCN variation spans over an order of magnitude, from 1 to 15 in Bacteria, but only up to 5 in Archaea [5]."
- [readme] README data file format: [evidence span withheld — anchor status: external_doc]
- [methods] Toolchain: "Perl (>= 5.10.0) <http://www.perl.com/download.csp>"
