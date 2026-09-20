---
name: marker-gene-based-genome-recovery-assessment
description: Use when you have binned contigs (e.g. MetaBAT2 bins from a HiFi metagenome
  assembly) and need to grade MAG recovery into quality categories from marker-gene
  completeness/contamination estimates.
when_to_use_negative:
- You need assembly-level contiguity/read-mapping metrics rather than per-bin marker-gene
  quality — use Mapler's read_contig_mapping evaluation instead
- Bins have not been produced yet (no FASTA bins) — run binning (e.g. MetaBAT2) first
- You need to classify the whole assembly quality, not the individual MAGs it contains
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0622
tools:
- name: CheckM2
  role: Predicts completeness and contamination of each bin via universally trained
    machine-learning models; its quality_report.tsv drives the categorization
  repo: https://github.com/chklovski/CheckM2
- name: Mapler
  role: Pipeline that runs bin quality classification and emits checkm_report.txt
    and checkm-plot.pdf within its outputs tree
  repo: https://github.com/Nimauric/Mapler
- name: GTDB-Tk
  role: Provides taxonomic assignment of bins, used by Mapler alongside marker-gene
    content for quality categorization
  repo: https://github.com/Ecogenomics/GTDBTk
- name: MetaBAT2
  role: Produces the binned contigs whose quality this assessment grades
  repo: https://bitbucket.org/berkeleylab/metabat/src/master/
- name: Python
  role: Implementation language for parsing quality_report.tsv with pandas and rendering
    the plot with matplotlib
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:48d5e38d57b6083e4982358ae8700f81c641a66041a7c0326fbe0b8d0812967f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chklovski/CheckM2
  tools:
  - CheckM2
  - GTDB-Tk
  - Mapler
  - MetaBAT2
  - Python
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# marker-gene-based-genome-recovery-assessment

## Summary

Assesses the quality of metagenome-assembled genomes (MAGs) by predicting completeness and contamination with CheckM2 and classifying each bin into GSC-style quality categories (near complete, high, medium, low) as implemented in the Mapler pipeline for HiFi metagenome assemblies.

## When to use

Use when you have binned contigs (e.g. MetaBAT2 bins from a HiFi metagenome assembly) and need to grade MAG recovery into quality categories from marker-gene completeness/contamination estimates.

## When NOT to use

- You need assembly-level contiguity/read-mapping metrics rather than per-bin marker-gene quality — use Mapler's read_contig_mapping evaluation instead
- Bins have not been produced yet (no FASTA bins) — run binning (e.g. MetaBAT2) first
- You need to classify the whole assembly quality, not the individual MAGs it contains

## Inputs

- Binned contigs from a metagenome assembly (e.g. MetaBAT2 bins, .fa files)
- CheckM2 quality_report.tsv with per-bin completeness and contamination estimates
- Assembly/binning provenance (sample name, assembler, binner) to locate bins in the Mapler output tree

## Outputs

- checkm_report.txt: count of near complete, high quality, medium quality and low quality bins, followed by details on each bin
- checkm-plot.pdf: 2D scatter plot of bin completeness and contamination
- Per-bin table of completeness, contamination and assigned quality category

## How to apply

Run CheckM2 on the binned contigs to obtain its machine-learning predictions of completeness and contamination, returned as quality_report.tsv. Load this table with pandas and parse the completeness and contamination columns for every bin. Apply the GSC MAG criteria in order: near complete for single-contig bins with completeness >=99% and contamination <=1%; high quality for completeness >=90% and contamination <=5%; medium quality for completeness >=50% and contamination <=10%; and low quality for all remaining bins. Aggregate bin counts per category, compile per-bin details, and write a text report (checkm_report.txt) with an optional matplotlib 2D completeness-vs-contamination scatter plot (checkm-plot.pdf), mirroring Mapler's checkm output directory.

## Related tools

- **CheckM2** (Predicts completeness and contamination of each bin via universally trained machine-learning models; its quality_report.tsv drives the categorization) — https://github.com/chklovski/CheckM2
- **Mapler** (Pipeline that runs bin quality classification and emits checkm_report.txt and checkm-plot.pdf within its outputs tree) — https://github.com/Nimauric/Mapler
- **GTDB-Tk** (Provides taxonomic assignment of bins, used by Mapler alongside marker-gene content for quality categorization) — https://github.com/Ecogenomics/GTDBTk
- **MetaBAT2** (Produces the binned contigs whose quality this assessment grades) — https://bitbucket.org/berkeleylab/metabat/src/master/
- **Python** (Implementation language for parsing quality_report.tsv with pandas and rendering the plot with matplotlib)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('checkm/quality_report.tsv',sep='\t'); print(df[['Name','Completeness','Contamination']])"
```

## Evaluation signals

- Every bin in quality_report.tsv receives exactly one of the four quality labels
- Per-category counts (near complete, high, medium, low) sum to the total bin count in the report
- checkm_report.txt exists under outputs/<sample>/<assembler>/<binner>_bins_reads_alignement/checkm/ and contains both category counts and per-bin details
- checkm-plot.pdf renders a 2D completeness-vs-contamination scatter whose points match the tabulated estimates
- Categorization is reproducible: re-running on the same quality_report.tsv yields identical category assignments

## Limitations

- CheckM2's completeness/contamination are machine-learning predictions and can be unreliable for lineages with few high-quality genomic representatives in the training set
- The 'near complete' category additionally requires a single contig, so a multi-contig bin at >=99% completeness does not qualify
- In Mapler's test configuration, only part of the analysis is run on the test dataset, so checkm outputs may require the full configuration to appear

## Evidence

- [other] near complete (single contig, ≥99% comp., ≤1% cont.), high quality (≥90% comp., ≤5% cont.), medium quality (≥50% comp., ≤10% cont.): [evidence span withheld — anchor status: not_found]
- [readme] In addition to classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment: "In addition to classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment, Mapler analyzes the alignment of reads on contigs."
- [readme] checkm-plot.pdf # A 2D plot of bin completeness and contamination: "checkm-plot.pdf # A 2D plot of bin completeness and contamination
	    │	    └── checkm_report.txt # A count of near complete, high quality, medium quality and low quality bins"
- [methods] Binning is performed with MetaBAT2: "Binning is performed with MetaBAT2"
- [readme] Unlike CheckM1, CheckM2 has universally trained machine learning models: [evidence span withheld — anchor status: external_doc]
