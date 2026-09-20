---
name: phylogenetic-independent-contrasts-ancestral-state-estimation
description: Use when Greengenes OTUs or taxa lack sequenced genomes (hence no measured
  16S rRNA copy number) and a phylogenetically-based GCN estimate is needed for CopyRighter
  correction.
when_to_use_negative:
- The taxon of interest already has a fully sequenced genome with a measured 16S rRNA
  copy number — use the measured value directly rather than the PIC estimate
- Your amplicon profiles use a non-Greengenes taxonomy (e.g. Silva), since CopyRighter's
  pre-computed estimates cover only the Oct 2012 Greengenes taxonomy
- You need corrections other than 16S rRNA copy number (e.g. GC percent or genome
  length bias) without appropriate pre-computed trait estimates
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3302
tools:
- name: CopyRighter
  role: Pipeline whose preprocessing scripts implement the PIC-based GCN inference;
    distributes phylogenetically-based GCN data files for all Greengenes OTUs and
    taxa
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl library used to load the Greengenes OTU tree and taxonomy and to map
    OTU IDs
- name: Perl >= 5.10.0
  role: Runtime environment for the CopyRighter preprocessing scripts that compute
    the PIC estimates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:33101cdf3b03a6d5a19b4aee98f5370fcb50df925838d1360a1719f9db79374b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Perl >= 5.10.0
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3302
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# phylogenetic-independent-contrasts-ancestral-state-estimation

## Summary

A CopyRighter preprocessing method that infers 16S rRNA gene copy number (GCN) for unsequenced Greengenes taxa by mapping measured GCN of curated genomes onto a 99%-identity OTU tree and propagating estimates to internal nodes and unsequenced leaves via phylogenetically independent contrasts (PIC) with rerooting. This fills the GCN gap that otherwise blocks lineage-specific correction of amplicon and qPCR profiles.

## When to use

Use when Greengenes OTUs or taxa lack sequenced genomes (hence no measured 16S rRNA copy number) and a phylogenetically-based GCN estimate is needed for CopyRighter correction.

## When NOT to use

- The taxon of interest already has a fully sequenced genome with a measured 16S rRNA copy number — use the measured value directly rather than the PIC estimate
- Your amplicon profiles use a non-Greengenes taxonomy (e.g. Silva), since CopyRighter's pre-computed estimates cover only the Oct 2012 Greengenes taxonomy
- You need corrections other than 16S rRNA copy number (e.g. GC percent or genome length bias) without appropriate pre-computed trait estimates

## Inputs

- Greengenes 99%-identity OTU tree and taxonomy
- Measured 16S rRNA gene copy numbers for curated genomes (mapped onto OTU leaves)

## Outputs

- Tab-delimited GCN estimates table (e.g. gcn_estimates.tsv) with per-OTU and per-taxon 16S rRNA copy number values for all bacterial and archaeal lineages
- Trait data file usable via CopyRighter's -d option, with ID/taxstring (col 1) and 16S rRNA count (col 2)

## How to apply

Load the Greengenes 99%-identity OTU tree and taxonomy under Perl (>= 5.10.0) using Bio::Community. Map each curated genome's measured 16S rRNA gene copy number onto the leaf of its corresponding OTU in the tree, then reroot the tree as required and compute ancestral-node GCN estimates with phylogenetically independent contrasts (PIC), exploiting the observation that related species have similar GCN. Aggregate node-level estimates into per-OTU and per-taxon GCN values covering all bacterial and archaeal lineages, and write the resulting table (e.g. gcn_estimates.tsv) as a tab-delimited two-column file with ID or taxonomic string in column 1 and trait estimate in column 2. Validate that the table covers all Greengenes OTUs/taxa and that inferred values fall in the biologically reported range of 1 to 15 in Bacteria and up to 5 in Archaea.

## Related tools

- **CopyRighter** (Pipeline whose preprocessing scripts implement the PIC-based GCN inference; distributes phylogenetically-based GCN data files for all Greengenes OTUs and taxa) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl library used to load the Greengenes OTU tree and taxonomy and to map OTU IDs)
- **Perl >= 5.10.0** (Runtime environment for the CopyRighter preprocessing scripts that compute the PIC estimates)

## Examples

```
perl -e 'use Bio::Community;' # load Greengenes 99% OTU tree + genome GCN, reroot, run PIC, and write gcn_estimates.tsv
```

## Evaluation signals

- The output table covers all Greengenes OTUs and taxa (bacterial and archaeal lineages)
- Inferred GCN values fall within the biologically reported range of 1 to 15 in Bacteria and up to 5 in Archaea
- Estimates are formatted as a tab-delimited two-column file (ID or taxonomic string, trait estimate) with comment lines starting with '#' ignored
- Trait values are produced at both OTU-ID level (e.g. 4 -> 1.51098055313977) and taxstring level (e.g. k__Archaea; p__; ... -> 1.57262)

## Limitations

- Estimates are phylogenetic inferences, not measurements, for unsequenced taxa; accuracy depends on the existence of closely related sequenced relatives
- Pre-computed GCN data files are based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy; extension to other taxonomies such as Silva was only anticipated, not implemented
- Since related species have similar but not identical GCN, variation between strains of the same species (as noted for the Ribosomal RNA Database) is not captured

## Evidence

- [intro] PIC with rerooting on Greengenes tree: "Since related species have similar GCN [8,9], it is often possible to accurately estimate GCN of an uncultured organism if a closely related sequenced relative exists"
- [abstract] Coverage of all Greengenes lineages: "we inferred the GCN of all bacterial and archaeal lineages in the Greengenes database within a phylogenetic framework"
- [discussion] Freely available GCN estimate files: "We have produced freely available data files of the phylogenetically-based estimates of GCN for all OTUs and taxa in the Greengenes database."
- [readme] Output file format for trait estimates: [evidence span withheld — anchor status: external_doc]
- [intro] GCN biological range: "GCN variation spans over an order of magnitude, from 1 to 15 in Bacteria, but only up to 5 in Archaea"
