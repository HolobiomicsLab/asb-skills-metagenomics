---
name: open-reference-otu-picking-greengenes
description: Use when raw 16S rRNA amplicon reads (e.g., FASTQ from a QIITA study)
  must be converted into an OTU-level feature table, RDP taxonomy table, and FastTree
  phylogeny that are structurally consistent with a deposited phyloseq/biom dataset.
when_to_use_negative:
- The processed data already exist (otu.tab.txt, tax.tab.txt, tree.tre, or biom.Rdata
  are available in the MiSurvGit Data folder), so only validation against those deposits
  is needed rather than re-clustering.
- Input is shotgun metagenomics data rather than 16S rRNA amplicon reads (the MiSurv
  article lists MetaPhlAn/MEGAN for shotgun metagenomics, not QIIME 2.0 OTU picking).
- The downstream goal is a denoised amplicon sequence variant (ASV) table instead
  of 97% OTU clusters, since this skill reproduces the paper's specific open-reference
  OTU protocol.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: QIIME2
  role: Runs quality filtering, the 97% open-reference OTU picking against Greengenes,
    and generation of the feature table, taxonomic annotations, and phylogenetic tree
- name: Greengenes database
  role: Reference database for 97% sequence similarity open-reference picking used
    to build the feature table
- name: MiSurv
  role: Consumes the resulting otu.tab.txt, sam.dat.txt, tax.tab.txt, and tree.tre
    (or biom.Rdata phyloseq object) as example processed data
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:262ab9ac78991d565fb49f308e81c4f8a8c6160f641d120a110a23609c1c3d8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - Greengenes database
  - MiSurv
  - QIIME2
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# open-reference-otu-picking-greengenes

## Summary

Reconstructs a 16S rRNA amplicon feature table by open-reference OTU picking at 97% sequence similarity against the Greengenes database in QIIME 2.0, as used to build the processed example data (otu.tab.txt, tax.tab.txt, tree.tre, biom.Rdata) for the MiSurv platform from QIITA study 10508 raw reads.

## When to use

Use when raw 16S rRNA amplicon reads (e.g., FASTQ from a QIITA study) must be converted into an OTU-level feature table, RDP taxonomy table, and FastTree phylogeny that are structurally consistent with a deposited phyloseq/biom dataset.

## When NOT to use

- The processed data already exist (otu.tab.txt, tax.tab.txt, tree.tre, or biom.Rdata are available in the MiSurvGit Data folder), so only validation against those deposits is needed rather than re-clustering.
- Input is shotgun metagenomics data rather than 16S rRNA amplicon reads (the MiSurv article lists MetaPhlAn/MEGAN for shotgun metagenomics, not QIIME 2.0 OTU picking).
- The downstream goal is a denoised amplicon sequence variant (ASV) table instead of 97% OTU clusters, since this skill reproduces the paper's specific open-reference OTU protocol.

## Inputs

- raw 16S rRNA sequence reads from QIITA study 10508 (https://qiita.ucsd.edu/study/description/10508)
- Greengenes database (reference for 97% open-reference OTU picking)
- deposited reference files otu.tab.txt, tax.tab.txt, tree.tre, and biom.Rdata from the MiSurvGit Data folder

## Outputs

- OTU count table (otu.tab.txt)
- taxonomy table (tax.tab.txt)
- Newick phylogenetic tree file (tree.tre)
- phyloseq-format object (biom.Rdata)

## How to apply

Download the raw 16S rRNA sequence reads from QIITA study 10508 (https://qiita.ucsd.edu/study/description/10508). Quality-filter the reads in QIIME 2.0, retaining only reads with >75% of the original length and removing reads with more than three consecutive low-quality bases (Phred score <20). Build the feature table by open-reference OTU picking at 97% sequence similarity based on the Greengenes database in QIIME 2.0. Assign taxonomy with the RDP classifier while removing chimeric sequences with ChimeraSlayer, then construct the phylogenetic tree of representative sequences with FastTree. Export the results as an OTU count table (otu.tab.txt), a taxonomy table (tax.tab.txt), and a Newick tree file (tree.tre), and check them against the files deposited in the MiSurvGit Data folder alongside biom.Rdata.

## Related tools

- **QIIME2** (Runs quality filtering, the 97% open-reference OTU picking against Greengenes, and generation of the feature table, taxonomic annotations, and phylogenetic tree)
- **Greengenes database** (Reference database for 97% sequence similarity open-reference picking used to build the feature table)
- **MiSurv** (Consumes the resulting otu.tab.txt, sam.dat.txt, tax.tab.txt, and tree.tre (or biom.Rdata phyloseq object) as example processed data) — https://github.com/wg99526/MiSurvGit

## Evaluation signals

- The three generated files otu.tab.txt, tax.tab.txt, and tree.tre exist and can be compared content-wise against the deposits in the MiSurvGit Data folder.
- The reproduced feature table and taxonomic table have the same sample/OTU dimensions as the deposited otu.tab.txt and tax.tab.txt.
- The tree file is in Newick format and structurally consistent with the deposited tree.tre.
- The 97% similarity threshold with open-reference picking against Greengenes was used (not closed- or de novo-only picking).
- Chimeric sequences were removed with ChimeraSlayer before taxonomy assignment with the RDP classifier.

## Limitations

- The article does not state specific QIIME 2.0 commands, versions of supporting R packages, or per-file checksums, so reproduction is validated by structural consistency (dimensions, format) rather than exact per-cell identity.
- The protocol is tied to the Zhang et al. T1D 16S rRNA V4 data of QIITA study 10508; applying it to other regions or studies is an extrapolation beyond the paper's evidence.
- No changelog or version history was found for the repository, so the deposited processed data may not be traceable to a specific pipeline run.

## Evidence

- [discussion] feature table via 97% open-reference OTU picking against Greengenes: "the feature table using the 97% sequence similarity through open reference piking based on the Greengenes database (63)"
- [discussion] QIIME 2.0 used for feature table, taxonomy, and tree: "We used QIIME 2.0 [16] to obtain the feature table, taxonomic annotations, and phylogenetic tree"
- [discussion] quality filtering thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] chimera removal and RDP classifier: "the taxonomic annotations using the RDP classifier while removing chimeras using ChimeraSlayer (64)"
- [readme] deposited example data files and raw read source: "The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508 (https://qiita.ucsd.edu/study/description/10508)."
