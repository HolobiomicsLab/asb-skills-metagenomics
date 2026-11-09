---
name: 16s-phylogenetic-tree-inference
description: Use when raw 16S amplicon reads must be converted into a phylogenetic
  tree to accompany an OTU/feature table and taxonomy table (e.g., to build a phyloseq
  object like MiSurv's biom.Rdata example data).
when_to_use_negative:
- The tree file (tree.tre) or phyloseq object (biom.Rdata) is already available in
  the MiSurvGit Data folder and only needs loading
- Input is shotgun metagenomics data rather than 16S rRNA amplicon reads (the article
  names MEGAN/MetaPhlAn for shotgun)
- A different OTU-clustering reference or similarity threshold than 97% open-reference
  against Greengenes is required
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: QIIME2
  role: Quality-filters the 16S reads (>75% length retention; removes >3 consecutive
    Phred<20 bases) and performs 97% open-reference OTU picking against Greengenes
    to yield representative sequences for tree building
  repo: https://github.com/wg99526/MiSurvGit
- name: Greengenes database
  role: Reference database for open-reference OTU picking at 97% sequence similarity
- name: FastTree
  role: Constructs the phylogenetic tree of representative OTU sequences exported
    as Newick tree.tre
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:43e57de3d08c13a90d8365d1df48fed85b23274bcc2556a733963b7bbf3ef509
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - FastTree
  - Greengenes database
  - MiSurv
  - QIIME2
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# 16S-phylogenetic-tree-inference

## Summary

Reconstructs the phylogenetic tree of 16S rRNA representative sequences from QIITA study 10508 raw reads using QIIME 2.0 processing (97% open-reference OTU picking against Greengenes, ChimeraSlayer chimera removal) and FastTree, exporting a Newick tree.tre that matches the MiSurv example processed data.

## When to use

Use when raw 16S amplicon reads must be converted into a phylogenetic tree to accompany an OTU/feature table and taxonomy table (e.g., to build a phyloseq object like MiSurv's biom.Rdata example data).

## When NOT to use

- The tree file (tree.tre) or phyloseq object (biom.Rdata) is already available in the MiSurvGit Data folder and only needs loading
- Input is shotgun metagenomics data rather than 16S rRNA amplicon reads (the article names MEGAN/MetaPhlAn for shotgun)
- A different OTU-clustering reference or similarity threshold than 97% open-reference against Greengenes is required

## Inputs

- Raw 16S rRNA amplicon sequence reads from QIITA study 10508 (https://qiita.ucsd.edu/study/description/10508)
- Greengenes reference database (for 97% open-reference OTU picking)

## Outputs

- Newick-format phylogenetic tree file (tree.tre) of representative OTU sequences
- OTU count table (otu.tab.txt) from the same pipeline run
- Taxonomy table (tax.tab.txt) from RDP classification with chimeras removed
- Optionally, a combined phyloseq object (biom.Rdata) packing all four files

## How to apply

Download the raw 16S rRNA sequence reads from QIITA study 10508 (study ID 10508 at qiita.ucsd.edu). Quality-filter in QIIME 2.0: retain reads with >75% of the original length and drop reads with more than three consecutive low-quality bases (Phred score <20). Build the feature table by 97% open-reference OTU picking against the Greengenes database, then assign taxonomy with the RDP classifier while removing chimeras using ChimeraSlayer. Construct the phylogenetic tree from the representative OTU sequences with FastTree and export it as a Newick file (tree.tre) alongside otu.tab.txt and tax.tab.txt. Judge success by structural consistency of the three outputs with the deposited files in the MiSurvGit Data folder: same sample/OTU dimensions and a valid Newick tree format.

## Related tools

- **QIIME2** (Quality-filters the 16S reads (>75% length retention; removes >3 consecutive Phred<20 bases) and performs 97% open-reference OTU picking against Greengenes to yield representative sequences for tree building) — https://github.com/wg99526/MiSurvGit
- **Greengenes database** (Reference database for open-reference OTU picking at 97% sequence similarity)
- **FastTree** (Constructs the phylogenetic tree of representative OTU sequences exported as Newick tree.tre)

## Evaluation signals

- tree.tre exists and parses as a Newick-format phylogenetic tree
- Tree tips match the OTU/feature IDs in otu.tab.txt and tax.tab.txt (same OTU dimensions across the three files)
- Reproduced tree.tre, otu.tab.txt, and tax.tab.txt are structurally consistent with the deposited reference files in the MiSurvGit Data folder (file existence plus content comparison)
- Chimeric sequences were removed (ChimeraSlayer step ran) before representative sequences were used for tree construction
- Phylogenetic tree, feature table, and taxonomy table can be jointly loaded into a phyloseq object like the deposited biom.Rdata

## Limitations

- Validation is by structural consistency (sample/OTU dimensions, tree format), not by exact bitwise reproduction of the deposited files
- No software version numbers for QIIME 2.0 components, RDP classifier, ChimeraSlayer, or FastTree are given, which may affect exact reproducibility
- No changelog is provided in the repository, so pipeline-version drift cannot be audited from the repo alone

## Evidence

- [discussion] Feature table, taxonomic annotation, and phylogenetic tree construction with QIIME 2.0: "We used QIIME 2.0 [16] to obtain the feature table, taxonomic annotations, and phylogenetic tree"
- [discussion] 97% open-reference OTU picking against Greengenes: "the feature table using the 97% sequence similarity through open reference piking based on the Greengenes database"
- [discussion] Quality filtering thresholds applied before tree inference: [evidence span withheld — anchor status: not_found]
- [readme] Deposited processed-data files to validate the reconstructed tree against: "The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508 (https://qiita.ucsd.edu/study/description/10508)."
