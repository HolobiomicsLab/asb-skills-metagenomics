---
name: interactive-html-report-structure-validation
description: Use when a MetaPhage Nextflow run has completed and you must confirm
  the interactive HTML report was generated with all expected sections (miner comparison,
  taxonomy, diversity) before comparing against the deposited reference.
when_to_use_negative:
- The report step was skipped (--skip_report true) or only intermediate step outputs
  (e.g., mining or taxonomy alone) exist, so no assembled HTML report is available
  to validate
- The goal is to assess biological results such as vOTU counts or richness patterns
  rather than the report's structure and panel completeness
- Input is not a MetaPhage run output (e.g., a generic MultiQC report from another
  pipeline), since the expected panels and deposited reference are MetaPhage-specific
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: MetaPhage
  role: Pipeline whose final step assembles the interactive HTML report to be validated
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: Nextflow
  role: Workflow manager executing the MetaPhage pipeline that produces the report
- name: MultiQC
  role: Assembles the interactive HTML report from fastp_qc, assembly, mining, taxonomy,
    cd-hit and diversity outputs
- name: Kraken2
  role: Read profiling step (miniBAV MiniKraken2_V1 default database) whose charts
    appear in the report
- name: MetaQuast
  role: Assembly overview statistics (BLASTN against SILVA 16S rRNA database) displayed
    in the report
- name: newProject.py
  role: Creates the project configuration file (project.conf) pointing at VLP reads
    and CSV metadata before the run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:e6f53c4bbca14b5fd02a756b59e7e2a5f6ea44f5654460e5324f518d3813e46d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MattiaPandolfoVR/MetaPhage
  tools:
  - Kraken2
  - MetaPhage
  - MetaQuast
  - MultiQC
  - Nextflow
  - newProject.py
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Interactive HTML report structure validation

## Summary

Validate that a MetaPhage run produces its final interactive MultiQC-assembled HTML report (report/index.html) with the expected panels — Miner Comparison, Kraken2 charts, metaQUAST assembly statistics, and diversity plots — and that its structure matches the authors' deposited figshare reference report.

## When to use

Use when a MetaPhage Nextflow run has completed and you must confirm the interactive HTML report was generated with all expected sections (miner comparison, taxonomy, diversity) before comparing against the deposited reference.

## When NOT to use

- The report step was skipped (--skip_report true) or only intermediate step outputs (e.g., mining or taxonomy alone) exist, so no assembled HTML report is available to validate
- The goal is to assess biological results such as vOTU counts or richness patterns rather than the report's structure and panel completeness
- Input is not a MetaPhage run output (e.g., a generic MultiQC report from another pipeline), since the expected panels and deposited reference are MetaPhage-specific

## Inputs

- MetaPhage report directory containing report/index.html from a completed Nextflow run on paired-end VLP metagenome reads
- Pipeline output subdirectories: fastp_qc, assembly, mining, taxonomy, cd-hit, diversity
- Deposited reference report (v.0.3.2) from figshare doi.org/10.6084/m9.figshare.20424705

## Outputs

- Validated interactive HTML report (report/index.html) with Miner Comparison, Kraken2, metaQUAST, and diversity panels
- Confirmed presence of count table, taxonomy table, PhyloSeq objects, vOTU multifasta, protein multifasta, and GFF annotation

## How to apply

Run the MetaPhage pipeline with the report step left enabled (--skip_report defaults to false) so MultiQC assembles the interactive HTML report from the fastp_qc, assembly, mining, taxonomy, cd-hit and diversity outputs. The pipeline must have produced all report inputs: fastp quality trimming (cut_mean_quality 15), phiX removal, Kraken2 profiling (miniBAV MiniKraken2_V1 default database), Megahit assembly with MetaQuast overview, and the four phage miners (VirFinder, VirSorter, Phigaro, VIBRANT). Inspect the report directory for index.html and confirm the summary visualizations and interactive filterable tables are present (count table, taxonomy table, PhyloSeq objects, vOTU multifasta, protein multifasta, GFF annotation). Then download the authors' deposited full output (v.0.3.2) from figshare (doi.org/10.6084/m9.figshare.20424705) and compare the generated report's structure and key figures against it, since MetaPhage is explicitly a reads-to-report pipeline whose final deliverable is an exhaustive report.

## Related tools

- **MetaPhage** (Pipeline whose final step assembles the interactive HTML report to be validated) — https://github.com/MattiaPandolfoVR/MetaPhage
- **Nextflow** (Workflow manager executing the MetaPhage pipeline that produces the report)
- **MultiQC** (Assembles the interactive HTML report from fastp_qc, assembly, mining, taxonomy, cd-hit and diversity outputs)
- **Kraken2** (Read profiling step (miniBAV MiniKraken2_V1 default database) whose charts appear in the report)
- **MetaQuast** (Assembly overview statistics (BLASTN against SILVA 16S rRNA database) displayed in the report)
- **newProject.py** (Creates the project configuration file (project.conf) pointing at VLP reads and CSV metadata before the run)

## Examples

```
nextflow run main.nf -c project.conf
```

## Evaluation signals

- report/index.html exists and renders as an interactive HTML report
- Expected panels are present: Miner Comparison showing vOTUs per phage mining tool, Kraken2 taxonomy charts, metaQUAST assembly statistics, and diversity plots
- Interactive filterable tables and summary artifacts are present: count table, taxonomy table, PhyloSeq objects, vOTU multifasta, protein multifasta, GFF annotation
- Report structure and key figures match the deposited figshare reference report (doi.org/10.6084/m9.figshare.20424705, v.0.3.2)
- All upstream report inputs were produced: fastp_qc, assembly, mining, taxonomy, cd-hit and diversity outputs from the enabled pipeline steps

## Limitations

- The deposited reference report is tied to MetaPhage v.0.3.2; a newer or dev-branch version with updated Miners may differ in report structure
- Comparison against the deposited report assumes the same Liang et al. 2020 neonatal VLP demo dataset (20 samples); other datasets will produce different figures
- vConTACT2-based classification steps are time-consuming and the network is rebuilt every time new data are added, which can delay producing the complete report

## Evidence

- [abstract] reads-to-report pipeline generating exhaustive report: "a comprehensive reads-to-report pipeline that streamlines the use of multiple phage miners and generates an exhaustive report"
- [abstract] Report summarizes and visualizes key findings: "The report both summarizes and visualizes the key findings and enables"
- [discussion] Single-run deliverables spanning QC to quantification: "In a single run, MetaPhage provides quality control, read profiling, assembly, phage detection, phage classification, and quantification."
- [other] MultiQC assembles the interactive HTML report from all step outputs: [evidence span withheld — anchor status: not_found]
- [other] Report directory contents to confirm: [evidence span withheld — anchor status: not_found]
- [discussion] metaQUAST assembly statistics shown in the report: "metaQuast uses BLASTN to identify the metagenome content aligning the contigs to the SILVA (34) 16s rRNA database."
- [readme] MetaPhage is a Nextflow phage discovery pipeline: "This is MetaPhage, a nextflow pipeline for automatic phage discovery."
