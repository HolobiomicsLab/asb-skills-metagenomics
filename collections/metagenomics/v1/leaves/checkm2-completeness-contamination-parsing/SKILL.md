---
name: checkm2-completeness-contamination-parsing
description: Use when you have CheckM2 quality_report.tsv output for binned HiFi metagenome
  assemblies (e.g. Deadwood or Mock community bins) and need to reproduce Mapler's
  near complete / high / medium / low quality counts.
when_to_use_negative:
- Input is raw reads or assembly contigs rather than binned MAGs that have already
  been assessed by CheckM2 (no quality_report.tsv exists yet — run CheckM2 first)
- The goal is taxonomic classification of bins or reads (use GTDB-Tk for bins or Kraken
  2 for reads, as Mapler does), not completeness/contamination quality categories
- The analysis requires assembly-level metrics such as mapped read/base ratios (MetaQUAST
  / read-contig alignment reports) rather than per-bin quality labels
edam_topics: []
tools:
- name: CheckM2
  role: Produces the machine-learning Completeness and Contamination estimates in
    quality_report.tsv that this skill parses and thresholds
  repo: https://github.com/chklovski/CheckM2
- name: Mapler
  role: Pipeline whose checkm report/plots (checkm_report.txt, checkm-plot.pdf) this
    skill reconstructs from the CheckM2 output
  repo: https://github.com/Nimauric/Mapler
- name: pandas
  role: Loads and parses the CheckM2 quality_report.tsv table in Python
- name: matplotlib
  role: Renders the optional 2D completeness/contamination scatter plot (checkm-plot.pdf)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:c1751bcf3f9dd9dac663e0c8cb9274283b843525627d571b2994fe2b71398906
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# checkm2-completeness-contamination-parsing

## Summary

Reproduces Mapler's bin quality categorization step: parse the CheckM2 quality_report.tsv for metagenomic bins and assign each bin to near complete, high quality, medium quality, or low quality using GSC MAG thresholds on completeness and contamination.

## When to use

Use when you have CheckM2 quality_report.tsv output for binned HiFi metagenome assemblies (e.g. Deadwood or Mock community bins) and need to reproduce Mapler's near complete / high / medium / low quality counts.

## When NOT to use

- Input is raw reads or assembly contigs rather than binned MAGs that have already been assessed by CheckM2 (no quality_report.tsv exists yet — run CheckM2 first)
- The goal is taxonomic classification of bins or reads (use GTDB-Tk for bins or Kraken 2 for reads, as Mapler does), not completeness/contamination quality categories
- The analysis requires assembly-level metrics such as mapped read/base ratios (MetaQUAST / read-contig alignment reports) rather than per-bin quality labels

## Inputs

- CheckM2 quality_report.tsv (per-bin Completeness and Contamination estimates)
- Binned FASTA files from the binning step (e.g. outputs/<sample>/<assembler>/metabat2_bins_reads_alignement/bins/bin.*.fa) to verify the single-contig near-complete condition

## Outputs

- checkm_report.txt: counts of near complete, high quality, medium quality and low quality bins followed by per-bin details (completeness, contamination, category)
- checkm-plot.pdf (optional): a 2D plot of bin completeness and contamination
- Per-bin category assignments suitable for downstream split-by-quality reports

## How to apply

Load the CheckM2 output table (quality_report.tsv) for the deposited bins with pandas in Python and parse each bin's Completeness and Contamination estimates. Categorize each bin by marker-gene-based GSC MAG criteria: near complete requires a single contig with completeness >=99% and contamination <=1%; high quality is completeness >=90% and contamination <=5%; medium quality is completeness >=50% and contamination <=10%; all remaining bins are low quality. Aggregate counts of bins per quality category and compile per-bin details (completeness, contamination, category). Write the summary and per-bin details as a text report (checkm_report.txt) and optionally render a 2D completeness/contamination scatter plot (checkm-plot.pdf) with matplotlib. The single-contig condition for near complete must be checked from the bin FASTA files, since it is not a column of the CheckM2 report.

## Related tools

- **CheckM2** (Produces the machine-learning Completeness and Contamination estimates in quality_report.tsv that this skill parses and thresholds) — https://github.com/chklovski/CheckM2
- **Mapler** (Pipeline whose checkm report/plots (checkm_report.txt, checkm-plot.pdf) this skill reconstructs from the CheckM2 output) — https://github.com/Nimauric/Mapler
- **pandas** (Loads and parses the CheckM2 quality_report.tsv table in Python)
- **matplotlib** (Renders the optional 2D completeness/contamination scatter plot (checkm-plot.pdf))

## Examples

```
import pandas as pd; df = pd.read_csv('outputs/sample/metabat2_bins_reads_alignement/checkm/quality_report.tsv', sep='\t'); df['quality'] = df.apply(lambda b: 'near complete' if (b.Completeness>=99 and b.Contamination<=1) else 'high quality' if b.Completeness>=90 and b.Contamination<=5 else 'medium quality' if b.Completeness>=50 and b.Contamination<=10 else 'low quality', axis=1)
```

## Evaluation signals

- Every bin in the input quality_report.tsv receives exactly one of the four labels (near complete, high quality, medium quality, low quality) — no bin is unlabeled or double-labeled
- Per-category counts written to checkm_report.txt sum to the total number of bins in the quality_report.tsv
- Boundary checks: a bin with completeness 99% and contamination 1% on a single contig is 'near complete'; completeness 90%/contamination 5% is 'high quality'; 50%/10% is 'medium quality'; anything below falls to 'low quality'
- checkm_report.txt contains both the per-category counts and the per-bin details (completeness, contamination, category), mirroring the Mapler output structure
- If rendered, checkm-plot.pdf shows each bin as a point in the completeness-vs-contamination plane matching the parsed values

## Limitations

- Near-complete labeling requires the single-contig condition, which must be derived from the bin FASTA files because CheckM2's report only provides completeness/contamination estimates
- Thresholds are GSC MAG criteria applied to CheckM2's machine-learning estimates; CheckM2 notes its models apply 'regardless of taxonomic lineage', so estimates may disagree with lineage-specific CheckM1-style assessments
- Only one binner/assembly combination (e.g. metabat2 under an assembler directory) is reported at a time; multi-binner comparisons require running the categorization per binner directory

## Evidence

- [readme] checkm_report.txt # A count of near complete, high quality, medium quality and low quality bins, followed by details on each bins: "checkm_report.txt # A count of near complete, high quality, medium quality and low quality bins, followed by details on each bins"
- [methods] classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment: "classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment"
- [readme] checkm-plot.pdf # A 2D plot of bin completeness and contamination: "checkm-plot.pdf # A 2D plot of bin completeness and contamination"
