---
name: qiita-data-retrieval-and-comparison
description: Use when raw 16S rRNA reads for a QIITA-hosted study (e.g. ID 10508)
  must be retrieved, reprocessed into feature/taxonomy/tree files, and compared against
  deposited processed data.
when_to_use_negative:
- Input is already a processed feature table, taxonomy table, or phyloseq object (biom.Rdata)
  — only downstream MiSurv analysis is needed, not raw-read reconstruction
- The analysis goal is one of MiSurv's analytic Modules 1–4 (survival comparison,
  composition comparison, association testing, prediction), which operate on already-processed
  data
- Data are shotgun metagenomic or not hosted in QIITA with raw reads available, since
  this skill depends on reprocessing raw 16S amplicon reads
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME2
  role: Quality-filters the raw reads (>75% original length; Phred <20, >3 consecutive
    low-quality bases) and builds the feature table via 97% open-reference OTU picking
- name: Greengenes database
  role: Reference database against which the 97% open-reference OTU picking is performed
- name: MiSurv
  role: Provides the deposited processed example data (otu.tab.txt, sam.dat.txt, tax.tab.txt,
    tree.tre, biom.Rdata) used as the comparison target
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:8fffd450f1437f405216cd44b070b4de271a2256eac4c08fd4d2f8e200f609d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - Greengenes database
  - MiSurv
  - QIIME2
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# qiita-data-retrieval-and-comparison

## Summary

Reconstructs the provenance of MiSurv's example processed data by retrieving the raw 16S rRNA reads of QIITA study 10508, reprocessing them in QIIME 2.0 into a feature table, RDP taxonomy, and FastTree phylogeny, and structurally comparing the outputs against the reference deposits (otu.tab.txt, tax.tab.txt, tree.tre, biom.Rdata) in the MiSurvGit Data folder.

## When to use

Use when raw 16S rRNA reads for a QIITA-hosted study (e.g. ID 10508) must be retrieved, reprocessed into feature/taxonomy/tree files, and compared against deposited processed data.

## When NOT to use

- Input is already a processed feature table, taxonomy table, or phyloseq object (biom.Rdata) — only downstream MiSurv analysis is needed, not raw-read reconstruction
- The analysis goal is one of MiSurv's analytic Modules 1–4 (survival comparison, composition comparison, association testing, prediction), which operate on already-processed data
- Data are shotgun metagenomic or not hosted in QIITA with raw reads available, since this skill depends on reprocessing raw 16S amplicon reads

## Inputs

- Raw 16S rRNA sequence reads from QIITA study 10508 (https://qiita.ucsd.edu/study/description/10508)
- Greengenes reference database (97% open-reference OTU picking)
- Deposited processed reference files from the MiSurvGit Data folder: otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre, biom.Rdata

## Outputs

- OTU count table (otu.tab.txt)
- Taxonomy table (tax.tab.txt)
- Newick phylogenetic tree file (tree.tre)
- Structural comparison report against the deposited otu.tab.txt, tax.tab.txt, tree.tre, and biom.Rdata

## How to apply

Download the raw 16S rRNA sequence reads from QIITA study 10508 (https://qiita.ucsd.edu/study/description/10508) and quality-filter them in QIIME 2.0, retaining only reads with >75% of the original length and removing reads with more than three consecutive low-quality bases (Phred score <20). Build the feature table by open-reference OTU picking at 97% sequence similarity against the Greengenes database. Assign taxonomy with the RDP classifier while removing chimeric sequences using ChimeraSlayer, and construct the phylogenetic tree of representative sequences with FastTree. Export the outputs as an OTU count table (otu.tab.txt), a taxonomy table (tax.tab.txt), and a Newick tree file (tree.tre). Finally, compare these three files against the deposited processed data (otu.tab.txt, tax.tab.txt, tree.tre, and the phyloseq-format biom.Rdata) in the MiSurvGit Data folder, accepting the reproduction when sample/OTU dimensions and tree format are structurally consistent.

## Related tools

- **QIIME2** (Quality-filters the raw reads (>75% original length; Phred <20, >3 consecutive low-quality bases) and builds the feature table via 97% open-reference OTU picking)
- **Greengenes database** (Reference database against which the 97% open-reference OTU picking is performed)
- **MiSurv** (Provides the deposited processed example data (otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre, biom.Rdata) used as the comparison target) — https://github.com/wg99526/MiSurvGit

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")
```

## Evaluation signals

- The three reproduced files (otu.tab.txt, tax.tab.txt, tree.tre) exist and are structurally consistent with the deposited versions in the MiSurvGit Data folder
- Feature table and taxonomy table have the same sample/OTU dimensions as the deposited otu.tab.txt and tax.tab.txt
- tree.tre parses as a Newick-format tree matching the deposited tree.tre
- biom.Rdata loads as a phyloseq object consistent with the four individual deposited files
- Quality filtering used the article thresholds: retained reads with >75% of original length, removed reads with more than three consecutive low-quality bases (Phred <20); OTU picking at 97% similarity against Greengenes

## Limitations

- Raw data availability depends on QIITA hosting of study 10508 (https://qiita.ucsd.edu/study/description/10508); the workflow cannot run without access to those reads
- Chimera removal via ChimeraSlayer is coupled to the RDP classifier taxonomy step, so swapping taxonomic classifiers changes the chimera-filtering context
- The README reports no version numbers for the R packages used and no changelog, so exact reproduction of package states is not documented

## Evidence

- [methods] The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508: "The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508"
- [discussion] QIIME 2.0 used to obtain feature table, taxonomy, tree: "We used QIIME 2.0 [16] to obtain the feature table, taxonomic annotations, and phylogenetic tree"
- [discussion] Quality-filter thresholds on read length and low-quality bases: [evidence span withheld — anchor status: not_found]
- [discussion] 97% open-reference OTU picking against Greengenes: "the feature table using the 97% sequence similarity through open reference piking based on the Greengenes database"
- [readme] Deposited processed files in GitHub Data folder: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [readme] QIITA study 10508 URL in README: "The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508 (https://qiita.ucsd.edu/study/description/10508)."
