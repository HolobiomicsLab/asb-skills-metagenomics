---
name: bin-quality-scatter-visualization
description: Use when a CheckM2 quality_report.tsv of metagenomic bins (completeness/contamination
  columns) is available and a visual, per-bin completeness-vs-contamination overview
  is needed alongside the quality category counts.
when_to_use_negative:
- Input is raw reads or an unassembled FASTA with no CheckM2 quality estimates — there
  are no completeness/contamination values to plot
- The goal is assembly-level contig evaluation (e.g. MetaQUAST or read-contig mapping
  ratios) rather than per-bin genome quality
- Only aggregate quality category counts are needed with no visual deliverable — the
  text report alone suffices
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: CheckM2
  role: Produces the completeness and contamination estimates per bin that populate
    the scatter plot axes
  repo: https://github.com/chklovski/CheckM2
- name: Python
  role: Implementation language for parsing quality_report.tsv and rendering the plot;
    Mapler source is implemented in Python as a Snakemake pipeline
  repo: https://github.com/Nimauric/Mapler
- name: pandas
  role: Loads and parses the CheckM2 quality_report.tsv table
- name: matplotlib
  role: Renders the 2D completeness/contamination scatter as checkm-plot.pdf
- name: Mapler
  role: Pipeline that orchestrates binning (MetaBAT2), CheckM2 assessment, quality
    categorization, and generation of the scatter plot and report
  repo: https://github.com/Nimauric/Mapler
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:42685a2c4438d6f89b04cc1a7f359227b9947ff9b4ee87943f540ecfe90e1375
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chklovski/CheckM2
  tools:
  - CheckM2
  - Mapler
  - Python
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# bin-quality-scatter-visualization

## Summary

Renders a 2D scatter plot of metagenomic bin completeness versus contamination from CheckM2 quality estimates, as produced by the Mapler pipeline for HiFi long-read assemblies, giving a visual complement to the per-bin quality categorization report.

## When to use

Use when a CheckM2 quality_report.tsv of metagenomic bins (completeness/contamination columns) is available and a visual, per-bin completeness-vs-contamination overview is needed alongside the quality category counts.

## When NOT to use

- Input is raw reads or an unassembled FASTA with no CheckM2 quality estimates — there are no completeness/contamination values to plot
- The goal is assembly-level contig evaluation (e.g. MetaQUAST or read-contig mapping ratios) rather than per-bin genome quality
- Only aggregate quality category counts are needed with no visual deliverable — the text report alone suffices

## Inputs

- CheckM2 quality_report.tsv with per-bin Completeness and Contamination columns
- FASTA bin files (bins/*.fa) whose names match rows in the quality report

## Outputs

- checkm-plot.pdf — a 2D scatter plot of bin completeness vs contamination
- checkm_report.txt — counts of near complete / high / medium / low quality bins plus per-bin details (completeness, contamination, category)

## How to apply

Load the CheckM2 output table (quality_report.tsv) for the binned assembly (e.g. Mapler's metabat2_bins_reads_alignement output) with pandas in Python, parsing the completeness and contamination estimates per bin. Categorize each bin using the GSC MAG criteria applied by Mapler: near complete (single contig, completeness >=99%, contamination <=1%), high quality (>=90% comp., <=5% cont.), medium quality (>=50% comp., <=10% cont.), and low quality otherwise. Render the completeness/contamination 2D scatter plot with matplotlib, optionally coloring or annotating points by quality category, and write it as checkm-plot.pdf next to the text report (checkm_report.txt). The scatter plot is the visual counterpart to the per-bin detail section of checkm_report.txt, so the points plotted must correspond one-to-one with bins listed in that report.

## Related tools

- **CheckM2** (Produces the completeness and contamination estimates per bin that populate the scatter plot axes) — https://github.com/chklovski/CheckM2
- **Python** (Implementation language for parsing quality_report.tsv and rendering the plot; Mapler source is implemented in Python as a Snakemake pipeline) — https://github.com/Nimauric/Mapler
- **pandas** (Loads and parses the CheckM2 quality_report.tsv table)
- **matplotlib** (Renders the 2D completeness/contamination scatter as checkm-plot.pdf)
- **Mapler** (Pipeline that orchestrates binning (MetaBAT2), CheckM2 assessment, quality categorization, and generation of the scatter plot and report) — https://github.com/Nimauric/Mapler

## Examples

```
df = pandas.read_csv('quality_report.tsv', sep='\t'); matplotlib.pyplot.scatter(df['Completeness'], df['Contamination']); matplotlib.pyplot.savefig('checkm-plot.pdf')
```

## Evaluation signals

- Every bin in quality_report.tsv receives exactly one of the four quality labels (near complete, high, medium, low) and per-category counts sum to the total bin count
- checkm-plot.pdf contains one point per bin, matching the per-bin detail section of checkm_report.txt
- Category assignment follows GSC thresholds: >=99% comp./<=1% cont. near complete (single contig), >=90%/<=5% high, >=50%/<=10% medium, otherwise low
- checkm_report.txt begins with category counts followed by per-bin completeness/contamination details, as in the Mapler output layout
- Plot file is a valid PDF named checkm-plot.pdf located alongside checkm_report.txt in the binning output directory

## Limitations

- Quality labels derive from CheckM2 marker-gene-based predictions; bins with poor machine-learning predictions for their lineage may be misclassified
- Near-complete labeling additionally depends on the bin being a single contig, so multi-contig bins with >=99% completeness do not qualify
- The visualization reflects only completeness/contamination; it does not capture the taxonomic assignment or uncaptured-diversity metrics Mapler also computes

## Evidence

- [other] Mapler assesses bins with CheckM2 and categorizes them by completeness and contamination: near complete (single contig, ≥99% comp., ≤1% cont.), high quality (≥90% comp., ≤5% cont.), medium quality (≥50% comp., ≤10% cont.), and low quality for remaining bins.: [evidence span withheld — anchor status: not_found]
- [readme] README output tree shows the scatter plot and report files produced by Mapler for a binned assembly.: "checkm-plot.pdf # A 2D plot of bin completeness and contamination"
- [readme] README describes checkm_report.txt content: category counts followed by per-bin details.: "checkm_report.txt # A count of near complete, high quality, medium quality and low quality bins, followed by details on each bins"
- [methods] Article methods state that Mapler classifies bins in quality categories by marker gene content and taxonomic assignment.: "classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment"
