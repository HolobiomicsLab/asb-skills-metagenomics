---
name: cross-deposit-output-comparison
description: Use when a pipeline's paper or documentation points to a deposited output
  archive (e.g., figshare doi.org/10.6084/m9.figshare.20424705) and you need to verify
  that your rerun reproduces the reference report structure and figures.
when_to_use_negative:
- No deposited reference output exists for the pipeline version you ran — there is
  nothing to diff against and validation must rely on internal checks instead
- You only need a single analysis result (e.g., one vOTU count) and not a report-structure
  comparison — inspecting the specific output file directly is simpler
- The deposited archive version differs substantially from your pipeline version (e.g.,
  dev branch with updated miners), making panel-level comparison misleading
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: MetaPhage
  role: Pipeline whose regenerated HTML report is compared against the deposited reference
    output
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: Nextflow
  role: Workflow manager used to rerun the pipeline (nextflow run main.nf -c project.conf)
    so all report inputs are produced
- name: MultiQC
  role: Assembles the interactive HTML report from fastp_qc, assembly, mining, taxonomy,
    cd-hit and diversity outputs that is then compared
- name: Kraken2
  role: Produces the taxonomy profiling charts (miniBAV default database) inspected
    in the report comparison
- name: MetaQuast
  role: Produces assembly overview statistics that must match the deposited report's
    metaQUAST panel
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:9628cbbee9ab8914762fa6dc4d6750623c9909b5424eb1895368792bd1bf80f3
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Cross-deposit output comparison

## Summary

Validates a locally regenerated analysis (e.g., a MetaPhage HTML report for the VLP dataset) by structurally and visually comparing it against the authors' deposited full output on figshare, confirming that every expected report panel and key figure reproduces.

## When to use

Use when a pipeline's paper or documentation points to a deposited output archive (e.g., figshare doi.org/10.6084/m9.figshare.20424705) and you need to verify that your rerun reproduces the reference report structure and figures.

## When NOT to use

- No deposited reference output exists for the pipeline version you ran — there is nothing to diff against and validation must rely on internal checks instead
- You only need a single analysis result (e.g., one vOTU count) and not a report-structure comparison — inspecting the specific output file directly is simpler
- The deposited archive version differs substantially from your pipeline version (e.g., dev branch with updated miners), making panel-level comparison misleading

## Inputs

- Locally generated MetaPhage report directory containing report/index.html
- Deposited authors' full output (v.0.3.2) from figshare doi.org/10.6084/m9.figshare.20424705
- VLP metagenome paired-end FASTQ reads and CSV metadata file used to build project.conf

## Outputs

- Panel-by-panel comparison of the regenerated report/index.html against the deposited reference report
- Verification checklist covering Miner Comparison vOTU counts, Kraken2 charts, diversity plots, and metaQUAST statistics
- Confirmation that count table, taxonomy table, PhyloSeq objects, vOTU multifasta, protein multifasta, and GFF annotation are present

## How to apply

First reproduce the full analysis end-to-end so all report inputs exist: run newProject.py to build project.conf over the VLP paired-end reads and metadata CSV, then run the MetaPhage Nextflow pipeline (nextflow run main.nf -c project.conf) with fastp quality trimming (cut_mean_quality 15), phiX removal, Kraken2 profiling with the miniBAV default database, Megahit assembly with MetaQuast, and the four phage miners (VirFinder, VirSorter, Phigaro, VIBRANT) enabled; leave the report step enabled (--skip_report defaults to false) so MultiQC assembles the interactive HTML from fastp_qc, assembly, mining, taxonomy, cd-hit and diversity outputs. Next, download the authors' deposited full output (v.0.3.2) from figshare and inspect both report trees. Compare panel-by-panel: the Miner Comparison section (number of vOTUs per mining tool), Kraken2 taxonomy charts, diversity plots, and metaQUAST assembly statistics, plus the presence of interactive filterable tables (count table, taxonomy table, PhyloSeq objects) and the vOTU/protein multifasta and GFF annotation files. Judge success when report/index.html renders with all expected panels and the structure and key figures match the deposited reference.

## Related tools

- **MetaPhage** (Pipeline whose regenerated HTML report is compared against the deposited reference output) — https://github.com/MattiaPandolfoVR/MetaPhage
- **Nextflow** (Workflow manager used to rerun the pipeline (nextflow run main.nf -c project.conf) so all report inputs are produced)
- **MultiQC** (Assembles the interactive HTML report from fastp_qc, assembly, mining, taxonomy, cd-hit and diversity outputs that is then compared)
- **Kraken2** (Produces the taxonomy profiling charts (miniBAV default database) inspected in the report comparison)
- **MetaQuast** (Produces assembly overview statistics that must match the deposited report's metaQUAST panel)

## Examples

```
nextflow run main.nf -c project.conf && ls report/index.html
```

## Evaluation signals

- report/index.html exists and renders as an interactive HTML report with the expected panels
- The Miner Comparison section is present and shows vOTU counts for VirFinder, VirSorter, Phigaro and VIBRANT
- Report structure and key figures match the deposited figshare full output (v.0.3.2)
- Interactive filterable tables (count table, taxonomy table, PhyloSeq objects) are present in the report directory
- vOTU multifasta, protein multifasta, and GFF annotation files are present alongside the report

## Limitations

- Comparison is against a versioned deposit (v.0.3.2); a different MetaPhage version or the dev branch with updated miners may produce legitimately different panels or vOTU counts
- vConTACT2 network is rebuilt every time new data are added, extending runtime, so reproducing every downstream figure can be time-consuming
- Users are urged to be mindful of biases inherent to virome analyses; small differences in vOTU counts may reflect the recall/precision trade-off of virus prediction rather than an error

## Evidence

- [abstract] reads-to-report pipeline that streamlines the use of multiple phage miners and generates an exhaustive report: "a comprehensive reads-to-report pipeline that streamlines the use of multiple phage miners and generates an exhaustive report"
- [methods] report step enabled by default: "(Boolean) Specify whether to perform the quality trimming of your reads or not. Default is `false`."
- [discussion] quality trimming threshold: "trimming where quality falls below a user-defined threshold set with the –cut_mean_quality option (15 by default)"
- [other] vOTU comparison of miners: [evidence span withheld — anchor status: not_found]
- [other] validation against deposited output: [evidence span withheld — anchor status: not_found]
