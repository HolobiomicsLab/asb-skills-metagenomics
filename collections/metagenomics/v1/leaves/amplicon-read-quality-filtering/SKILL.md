---
name: amplicon-read-quality-filtering
description: Use when raw 16S rRNA amplicon sequencing reads (e.g., FASTQ deposited
  in QIITA) must be quality-filtered in QIIME 2.0 before downstream feature table,
  taxonomy, and phylogeny generation for MiSurv-style survival analysis.
when_to_use_negative:
- Input is already processed data — the phyloseq object biom.Rdata or the four files
  (otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre) already in the MiSurv Data folder;
  go straight to the Data Processing module instead
- Data are shotgun metagenomic rather than 16S rRNA amplicon reads (the paper maps
  those to MEGAN/MetaPhlAn, not to this 16S QC step)
- Only post-read-level quality control is needed, e.g., the library-size/taxonomic-name
  QC or rarefy_even_depth rarefaction inside the MiSurv Data Processing module
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0102
tools:
- name: QIIME2
  role: Performs the read quality filtering (length and consecutive low-quality base
    cutoffs) and the subsequent feature table construction
- name: Greengenes database
  role: Reference database used for the 97% open-reference OTU picking that follows
    this filtering step
- name: ChimeraSlayer
  role: Removes chimeric sequences during taxonomic annotation after filtered reads
    are clustered
- name: MiSurv
  role: 'Downstream consumer: validates filtered-and-processed outputs against the
    deposited otu.tab.txt, tax.tab.txt, and tree.tre'
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:e760d48bea52399fe97b4b30d9901285da14738ad355c688ce3e67e09374d1c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - ChimeraSlayer
  - Greengenes database
  - MiSurv
  - QIIME2
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# amplicon-read-quality-filtering

## Summary

Reconstructs the read-level quality control that the MiSurv authors applied to raw 16S rRNA reads from QIITA study 10508: QIIME 2.0 filtering that retains reads with >75% of the original length and removes reads with more than three consecutive low-quality bases (Phred score <20), as the first stage before OTU picking, RDP taxonomy, and FastTree tree building.

## When to use

Use when raw 16S rRNA amplicon sequencing reads (e.g., FASTQ deposited in QIITA) must be quality-filtered in QIIME 2.0 before downstream feature table, taxonomy, and phylogeny generation for MiSurv-style survival analysis.

## When NOT to use

- Input is already processed data — the phyloseq object biom.Rdata or the four files (otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre) already in the MiSurv Data folder; go straight to the Data Processing module instead
- Data are shotgun metagenomic rather than 16S rRNA amplicon reads (the paper maps those to MEGAN/MetaPhlAn, not to this 16S QC step)
- Only post-read-level quality control is needed, e.g., the library-size/taxonomic-name QC or rarefy_even_depth rarefaction inside the MiSurv Data Processing module

## Inputs

- Raw 16S rRNA amplicon sequence reads from QIITA study 10508 (https://qiita.ucsd.edu/study/description/10508)

## Outputs

- Quality-filtered read set entering 97% open-reference OTU picking against Greengenes
- Downstream otu.tab.txt (OTU count table), tax.tab.txt (taxonomy table), and tree.tre (Newick tree) matching the MiSurv GitHub Data folder deposits

## How to apply

Download the raw 16S rRNA sequence reads from QIITA study 10508 and load them into QIIME 2.0. Apply the paper's two-part read filter: retain only reads with >75% of the original read length, and remove reads containing more than three consecutive low-quality bases defined as Phred score <20. Only after this filtering, build the feature table by 97% open-reference OTU picking against the Greengenes database, assign taxonomy with the RDP classifier while removing chimeras with ChimeraSlayer, and build the phylogenetic tree of representative sequences with FastTree. The rationale is that these thresholds are exactly the ones the authors used to produce the processed data MiSurv ships (otu.tab.txt, tax.tab.txt, tree.tre, biom.Rdata), so reproducing them keeps downstream survival analysis inputs consistent with the deposited example data.

## Related tools

- **QIIME2** (Performs the read quality filtering (length and consecutive low-quality base cutoffs) and the subsequent feature table construction)
- **Greengenes database** (Reference database used for the 97% open-reference OTU picking that follows this filtering step)
- **ChimeraSlayer** (Removes chimeric sequences during taxonomic annotation after filtered reads are clustered)
- **MiSurv** (Downstream consumer: validates filtered-and-processed outputs against the deposited otu.tab.txt, tax.tab.txt, and tree.tre) — https://github.com/wg99526/MiSurvGit

## Evaluation signals

- Only reads retaining >75% of their original length pass the filter; reads shorter than this cutoff are dropped
- Reads containing more than three consecutive bases with Phred score <20 are removed
- Downstream OTU clustering proceeds at 97% sequence similarity via open-reference picking against Greengenes
- Reproduced otu.tab.txt, tax.tab.txt, and tree.tre have the same sample/OTU dimensions and Newick format as the deposited files in the MiSurv GitHub Data folder
- Chimeric sequences are removed via ChimeraSlayer before taxonomic annotations are finalized

## Limitations

- The thresholds are specific to the Zhang et al. T1D mouse fecal 16S V4 data (QIITA study 10508); other datasets may need different cutoffs
- The paper provides thresholds but no executable QIIME 2.0 command lines or QIIME2 version pinned beyond 'QIIME 2.0', so exact reimplementation requires judgment
- No changelog or version numbers for the processing pipeline are provided in the repository

## Evidence

- [discussion] We retained only the reads with >75% of the original length while removing the reads with more than three consecutive low-quality bases (Phred score <20): [evidence span withheld — anchor status: not_found]
- [discussion] We used QIIME 2.0 [16] to obtain the feature table, taxonomic annotations, and phylogenetic tree: "We used QIIME 2.0 [16] to obtain the feature table, taxonomic annotations, and phylogenetic tree"
- [discussion] the feature table using the 97% sequence similarity through open reference piking based on the Greengenes database: "the feature table using the 97% sequence similarity through open reference piking based on the Greengenes database"
- [discussion] the taxonomic annotations using the RDP classifier while removing chimeras using ChimeraSlayer (64): "the taxonomic annotations using the RDP classifier while removing chimeras using ChimeraSlayer (64)"
- [readme] The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508 (https://qiita.ucsd.edu/study/description/10508).: "The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508 (https://qiita.ucsd.edu/study/description/10508)."
