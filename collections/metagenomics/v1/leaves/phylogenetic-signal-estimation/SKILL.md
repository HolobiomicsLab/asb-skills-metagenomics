---
name: phylogenetic-signal-estimation
description: Use when you need to test whether a quantitative trait estimated for
  OTUs or taxa (e.g., 16S rRNA GCN from CopyRighter's Greengenes data files) shows
  significant phylogenetic autocorrelation on a tree.
when_to_use_negative:
- The input is an amplicon profile to be corrected for GCN bias — use CopyRighter's
  copyrighter correction workflow instead; this skill only validates the phylogenetic
  signal underlying the GCN estimates
- The trait data are not anchored to Greengenes OTU IDs or Greengenes taxstrings,
  since CopyRighter only distributes GCN data files based on the Oct 2012 Greengenes
  taxonomy and other taxonomies (e.g. Silva) are not yet covered
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: CopyRighter
  role: Source of the phylogenetically-based GCN estimate data files for all OTUs
    and taxa in the Greengenes database that are mapped onto the trees
  repo: github.com/fangly/AmpliCopyrighter
- name: Perl
  role: Scripting environment (>= 5.10.0) used with Bio::Community to load Greengenes
    data files and tree, and to run the maximum-likelihood lambda estimation
- name: Bio::Community
  role: Perl library used to load and handle the CopyRighter Greengenes GCN data files
    and community/tree objects
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:cde38f444f2c8a8c4aebe5690cd316bd8c0cb4ab08acbb089c2a15c52f485829
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Perl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# phylogenetic-signal-estimation

## Summary

Quantifies the strength of phylogenetic signal (Pagel's lambda) in empirically determined 16S rRNA gene copy numbers (GCN) mapped onto the Greengenes phylogenetic tree and taxonomic tree, establishing that GCN can be reliably inferred from phylogeny.

## When to use

Use when you need to test whether a quantitative trait estimated for OTUs or taxa (e.g., 16S rRNA GCN from CopyRighter's Greengenes data files) shows significant phylogenetic autocorrelation on a tree.

## When NOT to use

- The input is an amplicon profile to be corrected for GCN bias — use CopyRighter's copyrighter correction workflow instead; this skill only validates the phylogenetic signal underlying the GCN estimates
- The trait data are not anchored to Greengenes OTU IDs or Greengenes taxstrings, since CopyRighter only distributes GCN data files based on the Oct 2012 Greengenes taxonomy and other taxonomies (e.g. Silva) are not yet covered

## Inputs

- CopyRighter-provided data file of 16S rRNA gene copy number estimates for all Greengenes OTUs (tab-delimited: Greengenes ID + GCN, e.g. OTU ID 4 -> 1.51098055313977)
- CopyRighter-provided data file of GCN estimates at Greengenes taxon level (taxstring + GCN, e.g. 'k__Archaea; p__Crenarchaeota; ... g__Cenarchaeum; s__symbiosum -> 1.00000')
- Greengenes phylogenetic tree (Newick) covering the inferred bacterial and archaeal lineages
- Greengenes taxonomic tree derived from the Oct 2012 Greengenes taxonomy

## Outputs

- phylosignal_lambda.tsv containing the lambda / P-value pair for the phylogenetic tree (0.844 / 2.13e-176) and the taxonomic tree (0.546 / 2.03e-38)
- Maximum-likelihood lambda estimates with likelihood-ratio P values testing lambda = 0

## How to apply

Load the CopyRighter-provided Greengenes GCN estimate files for all OTUs and taxa, together with the Greengenes phylogenetic tree, using Perl with Bio::Community. Map the empirical GCN values onto the tips and internal nodes of the Greengenes phylogeny, then estimate Pagel's lambda by maximum likelihood and compute the likelihood-ratio P value testing lambda against zero (phylogeny: lambda = 0.844, P = 2.13e-176). Repeat the maximum-likelihood lambda estimation on the Greengenes taxonomic tree using the taxon-level GCN values (taxonomy: lambda = 0.546, P = 2.03e-38). The rationale is that since related species have similar GCN, lambda close to 1 validates tree-based inference of GCN for uncultured lineages; the weaker lambda on the simplified taxonomic tree shows phylogenetic-level correction is preferable.

## Related tools

- **CopyRighter** (Source of the phylogenetically-based GCN estimate data files for all OTUs and taxa in the Greengenes database that are mapped onto the trees) — github.com/fangly/AmpliCopyrighter
- **Perl** (Scripting environment (>= 5.10.0) used with Bio::Community to load Greengenes data files and tree, and to run the maximum-likelihood lambda estimation)
- **Bio::Community** (Perl library used to load and handle the CopyRighter Greengenes GCN data files and community/tree objects)

## Evaluation signals

- Computed lambda on the Greengenes phylogenetic tree matches the reported value 0.844 within numerical tolerance
- Computed lambda on the Greengenes taxonomic tree matches the reported value 0.546 within numerical tolerance
- Likelihood-ratio P values are of the same extreme magnitude as the reported 2.13e-176 (phylogeny) and 2.03e-38 (taxonomy), indicating lambda is significantly greater than zero
- phylosignal_lambda.tsv contains both lambda/P pairs in the expected schema
- The lambda for the phylogenetic tree exceeds the lambda for the taxonomic tree, consistent with the finding that the signal is weaker in the simplified taxonomy

## Limitations

- The GCN estimates are tied to the Greengenes database; extension to other taxonomies such as Silva was only anticipated, not implemented
- Only empirically determined GCN mapped onto Greengenes OTU IDs and taxstrings can be used; alternative data files must be tab-delimited with an ID or taxonomic string (col 1) and trait estimate (col 2)
- A strong phylogenetic signal validates tree-based GCN inference only where closely related sequenced relatives exist; inference quality degrades for lineages without close relatives

## Evidence

- [discussion] phylogenetic signal: "We have produced freely available data files of the phylogenetically-based estimates of GCN for all OTUs and taxa in the Greengenes database."
- [intro] GCN-phylogeny link: "Since related species have similar GCN [8,9], it is often possible to accurately estimate GCN of an uncultured organism if a closely related sequenced relative exists"
- [abstract] Greengenes phylogenetic framework: "we inferred the GCN of all bacterial and archaeal lineages in the Greengenes database within a phylogenetic framework"
- [readme] trait data file format: [evidence span withheld — anchor status: external_doc]
- [readme] inference rationale: "it is possible to infer the copy number of unsequenced microbial species, based on that of close relatives that have been fully sequenced."
