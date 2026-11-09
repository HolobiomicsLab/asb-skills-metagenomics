---
name: rdp-taxonomic-annotation-16s
description: Use when raw 16S rRNA amplicon reads need taxonomic annotation via RDP
  classification with ChimeraSlayer chimera removal, to yield a taxonomy table compatible
  with an OTU count table and Newick tree.
when_to_use_negative:
- Input is already a processed phyloseq object or feature table (e.g. biom.Rdata or
  otu.tab.txt) rather than raw reads
- Data are shotgun metagenomic reads, where taxonomic profiling tools like MetaPhlAn
  or MEGAN apply instead of 16S RDP annotation
- An ASV/DADA2-style amplicon workflow is required, since this skill is tied to 97%
  open-reference OTU picking against Greengenes
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: QIIME2
  role: Pipeline in which the RDP classifier and ChimeraSlayer chimera removal are
    run to obtain taxonomic annotations for the feature table
- name: RDP classifier
  role: Assigns taxonomy to representative OTU sequences
- name: ChimeraSlayer
  role: Removes chimeric sequences during taxonomic annotation
- name: Greengenes database
  role: Reference database underlying the 97% open-reference OTU picking whose representative
    sequences receive RDP taxonomy
- name: MiSurv
  role: Consumes the resulting tax.tab.txt (with otu.tab.txt, sam.dat.txt, tree.tre)
    as example processed data in the Data folder
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:5de66fee8c1113322c1f6a1e82263ae09a55955eb55599b9a342c6f2a63f7e49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - ChimeraSlayer
  - Greengenes database
  - MiSurv
  - QIIME2
  - RDP classifier
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rdp-taxonomic-annotation-16s

## Summary

Assigns taxonomy to 16S rRNA OTUs using the RDP classifier while removing chimeric sequences with ChimeraSlayer, as done in MiSurv for QIITA study 10508 to produce the deposited tax.tab.txt that feeds the phyloseq object (biom.Rdata).

## When to use

Use when raw 16S rRNA amplicon reads need taxonomic annotation via RDP classification with ChimeraSlayer chimera removal, to yield a taxonomy table compatible with an OTU count table and Newick tree.

## When NOT to use

- Input is already a processed phyloseq object or feature table (e.g. biom.Rdata or otu.tab.txt) rather than raw reads
- Data are shotgun metagenomic reads, where taxonomic profiling tools like MetaPhlAn or MEGAN apply instead of 16S RDP annotation
- An ASV/DADA2-style amplicon workflow is required, since this skill is tied to 97% open-reference OTU picking against Greengenes

## Inputs

- Quality-filtered raw 16S rRNA V4 amplicon sequence reads (from QIITA study 10508)
- Feature table / representative OTU sequences from 97% open-reference OTU picking against the Greengenes database

## Outputs

- RDP taxonomic annotation table (tax.tab.txt)
- Chimera-filtered sequences suitable for FastTree phylogeny construction (tree.tre)

## How to apply

First quality-filter the raw reads in QIIME 2.0, retaining only reads with >75% of the original length and removing reads with more than three consecutive low-quality bases (Phred score <20), then build the feature table by 97% open-reference OTU picking against the Greengenes database. Assign taxonomy to the OTU sequences with the RDP classifier, removing chimeric sequences using ChimeraSlayer in the same step. The resulting annotations are exported as a taxonomy table (tax.tab.txt) alongside the OTU count table (otu.tab.txt) and the FastTree Newick tree (tree.tre). Rationale: MiSurv's example data (QIITA study 10508, Zhang et al. T1D mouse gut microbiome) was processed exactly this way so that the RDP-derived taxonomy table can be combined with the OTU table and tree into the phyloseq object biom.Rdata. Validate by checking that the reproduced tax.tab.txt is structurally consistent with the deposited file (same sample/OTU dimensions and format).

## Related tools

- **QIIME2** (Pipeline in which the RDP classifier and ChimeraSlayer chimera removal are run to obtain taxonomic annotations for the feature table)
- **RDP classifier** (Assigns taxonomy to representative OTU sequences)
- **ChimeraSlayer** (Removes chimeric sequences during taxonomic annotation)
- **Greengenes database** (Reference database underlying the 97% open-reference OTU picking whose representative sequences receive RDP taxonomy)
- **MiSurv** (Consumes the resulting tax.tab.txt (with otu.tab.txt, sam.dat.txt, tree.tre) as example processed data in the Data folder) — https://github.com/wg99526/MiSurvGit

## Evaluation signals

- A taxonomy table (tax.tab.txt) is produced with dimensions matching the OTU count table (otu.tab.txt) rows
- Reproduced tax.tab.txt is structurally consistent with the deposited file in the MiSurvGit Data folder (same sample/OTU dimensions and format)
- Chimeric sequences are removed via ChimeraSlayer before downstream tree building
- All exported files (otu.tab.txt, tax.tab.txt, tree.tre) exist and can be combined into the phyloseq object biom.Rdata

## Limitations

- No version numbers are given for the R packages used in the surrounding pipeline (e.g. phyloseq), and the DOI/license details are not stated in the repository document
- Taxonomy is only as current as the Greengenes-based 97% OTU picking workflow; the article does not describe retraining or updating the RDP classifier
- The workflow is specific to 16S rRNA (V4) amplicon data from QIITA study 10508 and its quality thresholds (>75% length, Phred <20 consecutive low-quality removal)

## Evidence

- [discussion] the taxonomic annotations using the RDP classifier while removing chimeras using ChimeraSlayer (64): "the taxonomic annotations using the RDP classifier while removing chimeras using ChimeraSlayer (64)"
- [discussion] We used QIIME 2.0 [16] to obtain the feature table, taxonomic annotations, and phylogenetic tree: "We used QIIME 2.0 [16] to obtain the feature table, taxonomic annotations, and phylogenetic tree"
- [discussion] We retained only the reads with >75% of the original length while removing the reads with more than three consecutive low-quality bases (Phred score <20): [evidence span withheld — anchor status: not_found]
- [readme] The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508: "The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508"
- [readme] four individual files (see the files named 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', and 'tree.tre'): "four individual files (see the files named 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', and 'tree.tre')"
