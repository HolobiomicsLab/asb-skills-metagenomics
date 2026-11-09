---
name: maximum-likelihood-model-comparison
description: Use when a continuous trait measured across taxa (e.g., 16S rRNA gene
  copy number estimates from CopyRighter's Greengenes files) must be tested for phylogenetic
  signal on a tree, and the strength of that signal needs to be compared between a
  phylogeny and a taxonomic tree.
when_to_use_negative:
- Input is an amplicon OTU table needing GCN abundance correction — use the copyrighter
  correction step instead; lambda estimation only quantifies trait signal, it does
  not correct profiles
- The trait data are not mapped to the Greengenes taxonomy/phylogeny — CopyRighter
  GCN estimates currently cover Greengenes only and are not yet extended to other
  taxonomies such as Silva
- The trait of interest is binary or discrete rather than a continuous quantitative
  estimate like GCN, for which the continuous-trait lambda model is inappropriate
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_0621
tools:
- name: CopyRighter
  role: Provides the pre-computed, phylogenetically based GCN estimate data files
    for all Greengenes OTUs and taxa that serve as the trait values analyzed for phylogenetic
    signal
  repo: https://github.com/fangly/AmpliCopyrighter
- name: Perl
  role: Scripting environment (>= 5.10.0) used to load the Greengenes data files and
    tree, map GCN onto tree nodes, and run the maximum-likelihood lambda estimation
- name: Bio::Community
  role: Perl library used to read the CopyRighter Greengenes community/trait data
    files and handle OTU IDs and taxonomic strings
  repo: https://github.com/fangly/AmpliCopyrighter
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:c003fb7ad1bfa2e30831bea0d66dfd2ed50fd2a70e31fc527082cfd99de54e02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fangly/AmpliCopyrighter
  tools:
  - Bio::Community
  - CopyRighter
  - Perl
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# maximum-likelihood-model-comparison

## Summary

Estimates Pagel's lambda by maximum likelihood to quantify the strength of phylogenetic signal in a continuously valued trait (here, empirically determined 16S rRNA gene copy numbers) on both a phylogenetic tree and a taxonomic tree, testing lambda against zero with a likelihood-ratio P value.

## When to use

Use when a continuous trait measured across taxa (e.g., 16S rRNA gene copy number estimates from CopyRighter's Greengenes files) must be tested for phylogenetic signal on a tree, and the strength of that signal needs to be compared between a phylogeny and a taxonomic tree.

## When NOT to use

- Input is an amplicon OTU table needing GCN abundance correction — use the copyrighter correction step instead; lambda estimation only quantifies trait signal, it does not correct profiles
- The trait data are not mapped to the Greengenes taxonomy/phylogeny — CopyRighter GCN estimates currently cover Greengenes only and are not yet extended to other taxonomies such as Silva
- The trait of interest is binary or discrete rather than a continuous quantitative estimate like GCN, for which the continuous-trait lambda model is inappropriate

## Inputs

- CopyRighter-provided data files of 16S rRNA gene copy number (GCN) estimates for all OTUs in the Greengenes database
- CopyRighter-provided taxon-level GCN estimates (tab-delimited ID/taxstring-to-trait files as used by copyrighter -d)
- Greengenes phylogenetic tree (Newick) covering all bacterial and archaeal lineages
- Greengenes taxonomic tree built from Greengenes taxonomic strings (k__; p__; c__; o__; f__; g__; s__)

## Outputs

- phylosignal_lambda.tsv with lambda and likelihood-ratio P value pairs for the phylogenetic tree (lambda = 0.844, P = 2.13e-176) and the taxonomic tree (lambda = 0.546, P = 2.03e-38)

## How to apply

Load the trait estimates for all OTUs and taxa together with the Greengenes phylogenetic tree using Perl with Bio::Community. Map the empirical 16S rRNA gene copy number (GCN) values onto the tips and internal nodes of the phylogeny. Estimate Pagel's lambda by maximum likelihood and compute the likelihood-ratio P value testing lambda against zero; then repeat the maximum-likelihood lambda estimation on the Greengenes taxonomic tree using the taxon-level GCN values. The comparison reveals whether the phylogenetic signal is stronger on the full phylogeny than on the simplified taxonomic tree, which justifies tree-based inference of GCN. Accept the result when the computed lambda values match the references (0.844 phylogeny, 0.546 taxonomy) within numerical tolerance and the P values are of the same extreme magnitude as 2.13e-176 and 2.03e-38.

## Related tools

- **CopyRighter** (Provides the pre-computed, phylogenetically based GCN estimate data files for all Greengenes OTUs and taxa that serve as the trait values analyzed for phylogenetic signal) — https://github.com/fangly/AmpliCopyrighter
- **Perl** (Scripting environment (>= 5.10.0) used to load the Greengenes data files and tree, map GCN onto tree nodes, and run the maximum-likelihood lambda estimation)
- **Bio::Community** (Perl library used to read the CopyRighter Greengenes community/trait data files and handle OTU IDs and taxonomic strings) — https://github.com/fangly/AmpliCopyrighter

## Examples

```
perl -MBio::Community -e 'map GCN onto Greengenes tree; fit Pagel lambda by ML; write phylosignal_lambda.tsv'
```

## Evaluation signals

- Lambda on the Greengenes phylogenetic tree reproduces 0.844 within numerical tolerance
- Lambda on the Greengenes taxonomic tree reproduces 0.546 within numerical tolerance
- Likelihood-ratio P values are of the same extreme magnitude as 2.13e-176 (phylogeny) and 2.03e-38 (taxonomy), i.e. lambda is significantly greater than zero
- phylosignal_lambda.tsv contains exactly the two lambda/P pairs with finite values and valid headers
- Phylogeny lambda (0.844) exceeds taxonomy lambda (0.546), confirming signal is weaker in the simplified taxonomy

## Limitations

- The empirical GCN values analyzed are CopyRighter's estimates based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy, so the signal is conditioned on that database release
- Signal on the simplified taxonomic tree is weaker (lambda 0.546 vs 0.844), so taxonomic-level inference of GCN is less reliable than phylogenetic-level inference
- CopyRighter GCN estimates currently cover only Greengenes; extension to other taxonomies (e.g., Silva) is only anticipated, limiting where the tree comparison can be repeated

## Evidence

- [abstract] GCN is strongly linked to phylogeny: "16S rRNA gene copy number (GCN) is strongly linked to microbial phylogenetic taxonomy, potentially under-representing Archaea in amplicon microbial profiles"
- [discussion] Greengenes GCN data files available: "We have produced freely available data files of the phylogenetically-based estimates of GCN for all OTUs and taxa in the Greengenes database."
- [readme] README trait data file format: [evidence span withheld — anchor status: external_doc]
- [readme] README Greengenes taxstring format: [evidence span withheld — anchor status: external_doc]
