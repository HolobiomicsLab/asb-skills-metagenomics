---
name: chimera-detection-removal
description: Use when processing raw 16S rRNA amplicon reads (e.g., V4 regions from
  QIITA studies) into taxonomic annotations, and chimeric PCR artifacts must be filtered
  out before building the taxonomy table.
when_to_use_negative:
- Input is already a processed feature table / phyloseq object (e.g., biom.Rdata or
  otu.tab.txt from the MiSurv Data folder) — chimera removal was already applied upstream.
- Data are shotgun metagenomic profiles (e.g., processed via MEGAN or MetaPhlAn) rather
  than 16S rRNA amplicon sequences, where ChimeraSlayer-based 16S chimera checking
  does not apply.
- Taxonomic annotations come from a pipeline that performs its own chimera filtering
  internally and does not expose pre-filter sequences.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: ChimeraSlayer
  role: Detects and removes chimeric 16S sequences during the taxonomic annotation
    step of the MiSurv example data processing pipeline
- name: RDP classifier
  role: Assigns taxonomy to the reads in the same stage in which chimeras are removed
    via ChimeraSlayer
- name: QIIME2
  role: Hosts the upstream quality filtering and 97% open-reference OTU picking against
    Greengenes that precede chimera removal
- name: MiSurv
  role: Consumes the chimera-free taxonomy table (tax.tab.txt / biom.Rdata) in its
    data processing and four analytic modules
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:6f32cdef03f2d2fb153d3272f16c482fe486504593aebc2b79a610dcf8e6f4a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - ChimeraSlayer
  - MiSurv
  - QIIME2
  - RDP classifier
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Chimera detection and removal

## Summary

Removes chimeric 16S rRNA sequences during taxonomic annotation so that downstream microbiome-survival analyses in MiSurv are not distorted by artificial hybrid amplicons. In the MiSurv example pipeline (QIITA study 10508), chimeras are removed with ChimeraSlayer while taxonomy is assigned with the RDP classifier.

## When to use

Use when processing raw 16S rRNA amplicon reads (e.g., V4 regions from QIITA studies) into taxonomic annotations, and chimeric PCR artifacts must be filtered out before building the taxonomy table.

## When NOT to use

- Input is already a processed feature table / phyloseq object (e.g., biom.Rdata or otu.tab.txt from the MiSurv Data folder) — chimera removal was already applied upstream.
- Data are shotgun metagenomic profiles (e.g., processed via MEGAN or MetaPhlAn) rather than 16S rRNA amplicon sequences, where ChimeraSlayer-based 16S chimera checking does not apply.
- Taxonomic annotations come from a pipeline that performs its own chimera filtering internally and does not expose pre-filter sequences.

## Inputs

- Quality-filtered 16S rRNA amplicon reads (V4 region) from QIITA study 10508
- Feature table from 97% open-reference OTU picking against the Greengenes database
- Reference sequence context for chimera detection (ChimeraSlayer)

## Outputs

- Taxonomy table (tax.tab.txt) with RDP annotations and chimeras removed
- Chimera-checked OTU representative sequences feeding the FastTree phylogeny (tree.tre)
-  phyloseq-format object biom.Rdata deposited in the MiSurv Git Data folder

## How to apply

Run chimera checking alongside taxonomic assignment rather than as an entirely separate stage: assign taxonomy to the quality-filtered reads with the RDP classifier while removing chimeras using ChimeraSlayer. This step follows quality filtering in QIIME 2.0 (reads retained only if >75% of original length and lacking more than three consecutive low-quality bases with Phred score <20) and open-reference OTU picking at 97% sequence similarity against Greengenes. The rationale is that chimeras would otherwise contaminate the taxonomic table (tax.tab.txt) and propagate into MiSurv's downstream modules (composition comparison, association testing, survival prediction). The resulting clean taxonomy table is exported together with otu.tab.txt and tree.tre and packaged as a phyloseq object (biom.Rdata) in the MiSurv Data folder.

## Related tools

- **ChimeraSlayer** (Detects and removes chimeric 16S sequences during the taxonomic annotation step of the MiSurv example data processing pipeline)
- **RDP classifier** (Assigns taxonomy to the reads in the same stage in which chimeras are removed via ChimeraSlayer)
- **QIIME2** (Hosts the upstream quality filtering and 97% open-reference OTU picking against Greengenes that precede chimera removal)
- **MiSurv** (Consumes the chimera-free taxonomy table (tax.tab.txt / biom.Rdata) in its data processing and four analytic modules) — https://github.com/wg99526/MiSurvGit

## Evaluation signals

- Chimera removal co-occurs with RDP taxonomy assignment: the output tax.tab.txt should contain RDP-style taxonomic strings and no ChimeraSlayer-flagged sequences.
- Structural consistency of the reproduced taxonomy table with the deposited reference file tax.tab.txt (same OTU/sample dimensions and taxonomy format).
- The same OTU IDs appear in otu.tab.txt, tax.tab.txt, and tree.tre — chimera filtering must not orphan OTUs in one output only.
- Final processed data round-trips into a phyloseq object (biom.Rdata) loadable in R, matching the four deposited files (otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre).

## Limitations

- The article reports chimera removal only for the 16S rRNA V4 example (QIITA study 10508); behavior on other amplicon regions or shotgun data is not described.
- No software versions are given for ChimeraSlayer or the RDP classifier, so exact reproducibility of the chimera filtering is not guaranteed.
- Validation in the source workflow is structural (dimensions and format comparison against deposited files), not sequence-level chimera counts, so the true chimera removal rate cannot be verified from the deposits.

## Evidence

- [discussion] Chimera removal using ChimeraSlayer alongside RDP classification: "the taxonomic annotations using the RDP classifier while removing chimeras using ChimeraSlayer (64)"
- [discussion] Feature table and annotations obtained with QIIME 2.0 for the MiSurv example: "We used QIIME 2.0 [16] to obtain the feature table, taxonomic annotations, and phylogenetic tree"
- [discussion] Greengenes-based 97% open-reference picking precedes taxonomy assignment: "the feature table using the 97% sequence similarity through open reference piking based on the Greengenes database (63)"
- [readme] Deposited processed files that chimera-filtered annotations feed into: "four individual files (see the files named 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', and 'tree.tre')"
- [discussion] Upstream quality filtering thresholds applied before chimera removal: [evidence span withheld — anchor status: not_found]
