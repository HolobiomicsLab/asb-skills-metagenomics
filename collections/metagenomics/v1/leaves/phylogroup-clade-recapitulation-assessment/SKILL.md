---
name: phylogroup-clade-recapitulation-assessment
description: Use when you have pairwise synteny or strain-distance scores for genomes
  with a published phylogroup/phylogeny classification, and need to quantify whether
  the derived tree recovers those clades at statistical significance.
when_to_use_negative:
- Pairs of samples or genomes have fewer than n regions available, since such pairs
  are excluded from downstream analysis by SynTracker
- The comparison target is SNP-level sequence divergence rather than structural/syntenic
  similarity (SNP-based tools like MIDAS or StrainPhlAn are the appropriate comparison)
- The genomes span multiple species or lack a conserved reference genome, since SynTracker
  compares conspecific strains against a per-species reference
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: SynTracker
  role: Computes per-region pairwise synteny scores and subsampled APSS values between
    all genome pairs against a reference genome
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: SynTracker builds a BLAST database of target genomes and searches central
    regions with minimal identity 97% and minimal query coverage 70%
- name: phangorn R package
  role: Builds unweighted pair group method with arithmetic mean (UPGMA) phylogenetic
    trees from synteny distances at each subsampling depth
- name: TreeDist R package
  role: Calculates Information-based generalized Robinson-Foulds distances between
    synteny trees and the pruned Mash tree for significance testing
- name: Mash
  role: Provides the reference phylogroup classification/tree against which APSS-based
    trees are compared
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:ce696defee43806dc126f16d9c3a4fb240fea3e58dc0cf10542f2abb65dd015b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - Mash
  - SynTracker
  - TreeDist R package
  - phangorn R package
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# phylogroup-clade-recapitulation-assessment

## Summary

Tests whether APSS-based synteny phylogenies recapitulate a reference phylogroup classification (e.g., the Mash-based tree of 14 E. coli phylogroups across 140 genomes) by building UPGMA trees at multiple subsampling depths and scoring them against the pruned reference tree with Information-based generalized Robinson-Foulds distances and label-shuffling P-values.

## When to use

Use when you have pairwise synteny or strain-distance scores for genomes with a published phylogroup/phylogeny classification, and need to quantify whether the derived tree recovers those clades at statistical significance.

## When NOT to use

- Pairs of samples or genomes have fewer than n regions available, since such pairs are excluded from downstream analysis by SynTracker
- The comparison target is SNP-level sequence divergence rather than structural/syntenic similarity (SNP-based tools like MIDAS or StrainPhlAn are the appropriate comparison)
- The genomes span multiple species or lack a conserved reference genome, since SynTracker compares conspecific strains against a per-species reference

## Inputs

- Collection of assembled genomes or metagenomic assemblies (one FASTA per genome/sample) grouped by known phylogroup
- Reference genome FASTA (e.g., E. coli str. K-12 substrain MG1655, NCBI reference sequence NC_000913.3)
- Pruned reference phylogenetic tree in Newick format (e.g., Supplementary_Data_Fig_3c_10k_newick.nwk) containing only the analyzed genomes
- SynTracker per-region synteny scores table ([genome name]_synteny_scores_per_region.csv)

## Outputs

- APSS tables at each subsampling depth ([genome name]_avg_synteny_scores_[n]_regions.csv)
- UPGMA synteny-based phylogenetic trees per subsampling depth (phangorn)
- Per-depth Information-based generalized Robinson-Foulds distances and empirical P-values written to ecoli_info_rf_tree_distances.csv

## How to apply

Run SynTracker (python syntracker.py) against a single reference genome (e.g., E. coli K-12 MG1655, NC_000913.3) with default parameters (BLAST identity 97%, coverage 70%, region length 5000) to obtain per-region pairwise synteny scores. Prune the reference tree (e.g., the published Mash-based tree, Supplementary_Data_Fig_3c_10k_newick.nwk) so only external nodes of the analyzed genomes remain. For each subsampling depth (n = 20, 40, 60, 80, 100, 200 regions per pair), convert APSS to synteny distances as |APSS − 1| and build UPGMA trees with the phangorn R package. Compute Information-based generalized Robinson-Foulds distances between each synteny tree and the pruned Mash tree using the TreeDist R package, shuffling tree labels 100,000 iterations per comparison to derive empirical P-values. The skill succeeds when P < 1e-5 at every depth and phylogroup clades are recovered even when less than 2% of the genome is sampled (as shown for 140 E. coli genomes, ten per phylogroup).

## Related tools

- **SynTracker** (Computes per-region pairwise synteny scores and subsampled APSS values between all genome pairs against a reference genome) — https://github.com/leylabmpi/SynTracker
- **BLAST** (SynTracker builds a BLAST database of target genomes and searches central regions with minimal identity 97% and minimal query coverage 70%)
- **phangorn R package** (Builds unweighted pair group method with arithmetic mean (UPGMA) phylogenetic trees from synteny distances at each subsampling depth)
- **TreeDist R package** (Calculates Information-based generalized Robinson-Foulds distances between synteny trees and the pruned Mash tree for significance testing)
- **Mash** (Provides the reference phylogroup classification/tree against which APSS-based trees are compared)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Empirical P-value < 1e-5 at every subsampling depth (100,000 label shuffles per comparison)
- Phylogroup clades from the pruned Mash tree are recapitulated even when < 2% of the genome is sampled (e.g., n = 20 regions)
- Synteny distances equal exactly |APSS − 1| for every pair before tree building
- The pruned reference tree contains only external nodes corresponding to the analyzed genomes (e.g., 140 E. coli genomes, ten per phylogroup)
- APSS output tables exist for the expected default subsampling depths (40, 60, 80, 100, 200 regions per pair) plus all-regions

## Limitations

- Pairs with fewer than n regions per pairwise comparison are excluded, so shallow subsampling depths may drop genome pairs
- Subsampling with the default seed=1 is reproducible, but using --no_seed means APSS values may change between runs
- A small number of regions per pairwise comparison mostly results in lower accuracy; a trade-off exists between accuracy and sample size
- Strain/phylogroup delineation thresholds are subjective and species-dependent; results are relative to the chosen reference classification

## Evidence

- [discussion] pruning of the Mash-based tree so that only external nodes corresponding to the 140 E. coli genomes analyzed with SynTracker were kept: "pruning of the Mash-based tree so that only external nodes corresponding to the 140 E. coli genomes analyzed with SynTracker were kept"
- [discussion] APSS to synteny distances equal to the absolute value of APSS − 1: "APSS values were converted to synteny distances, which are equal to the absolute value of APSS − 1"
- [discussion] unweighted pair group method with arithmetic mean phylogenetic trees using the 'phangorn' R package: "unweighted pair group method with arithmetic mean phylogenetic trees using the ‘phangorn’ R package51"
- [discussion] calculating the Information-based generalized RFDs using the 'TreeDist' R package: "calculating the Information-based generalized RFDs using the ‘TreeDist’ R package52"
- [discussion] n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores: "n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores"
- [readme] gives the APSS (Average Pairwise Synteny Score) that was calculated by subsampling N regions per pair of samples: "gives the APSS 
(Average Pairwise Synteny Score) that was calculated by subsampling N regions per pair of samples"
