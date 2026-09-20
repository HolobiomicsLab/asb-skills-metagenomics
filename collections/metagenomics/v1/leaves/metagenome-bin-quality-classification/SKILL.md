---
name: metagenome-bin-quality-classification
description: Use when a set of binned metagenome contigs (e.g. MetaBAT2 bins from
  a HiFi co-assembly) needs to be assigned quality labels from CheckM2 completeness/contamination
  values for assembly comparison or reporting.
when_to_use_negative:
- Input is raw reads or an unbinned assembly with no MAG bins — quality categories
  are undefined without bins
- No completeness/contamination estimates are available (e.g. CheckM2 has not been
  run or the quality_report.tsv is missing)
- The goal is taxonomic placement of bins rather than quality tiering — use GTDB-Tk
  classification instead
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0196
tools:
- name: CheckM2
  role: Predicts per-bin completeness and contamination with machine-learning models,
    providing the numeric criteria the classification thresholds are applied to
  repo: https://github.com/chklovski/CheckM2
- name: Mapler
  role: Pipeline that wraps binning, CheckM2 assessment and the quality-category classification,
    emitting checkm_report.txt and checkm-plot.pdf
  repo: https://github.com/Nimauric/Mapler
- name: MetaBAT2
  role: Produces the assembly bins that are subsequently classified into quality categories
  repo: https://bitbucket.org/berkeleylab/metabat
- name: matplotlib
  role: Renders the completeness/contamination 2D scatter plot (checkm-plot.pdf)
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:be041d85d6364dec68d27805655c1e46ae33100e97d78f039c9394cba11f429d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chklovski/CheckM2
  tools:
  - CheckM2
  - Conda
  - Mapler
  - MetaBAT2
  - Python
  - hifiasm-meta
  - matplotlib
  - metaMDBG
  - metaflye
  - minimap2
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metagenome-bin-quality-classification

## Summary

Classifies metagenome assembly bins into the GSC/MIMAG-style quality categories used by Mapler — near complete, high, medium and low quality — by combining CheckM2 completeness and contamination estimates with single-contig criteria, and reporting per-bin and per-category summaries.

## When to use

Use when a set of binned metagenome contigs (e.g. MetaBAT2 bins from a HiFi co-assembly) needs to be assigned quality labels from CheckM2 completeness/contamination values for assembly comparison or reporting.

## When NOT to use

- Input is raw reads or an unbinned assembly with no MAG bins — quality categories are undefined without bins
- No completeness/contamination estimates are available (e.g. CheckM2 has not been run or the quality_report.tsv is missing)
- The goal is taxonomic placement of bins rather than quality tiering — use GTDB-Tk classification instead

## Inputs

- CheckM2 quality_report.tsv with completeness and contamination columns per bin
- Metagenome bins in FASTA format (e.g. MetaBAT2 outputs such as outputs/<sample>/<assembler>/metabat2_bins_reads_alignement/bins/bin.1.fa)
- Assembly FASTA (to test the single-contig criterion for 'near complete')

## Outputs

- checkm_report.txt: counts of near complete, high quality, medium quality and low quality bins followed by per-bin details
- checkm-plot.pdf: 2D scatter of bin completeness vs contamination
- Per-bin quality label table (bin id, completeness, contamination, category) usable for downstream read-mapping breakdowns by bin quality

## How to apply

Load the CheckM2 output table (quality_report.tsv) for the bins of interest (e.g. under outputs/<sample>/<assembler>/metabat2_bins_reads_alignement/checkm/) with pandas. Parse each bin's completeness and contamination estimates, then apply the GSC MAG criteria: near complete (single contig, completeness >=99%, contamination <=1%), high quality (completeness >=90%, contamination <=5%), medium quality (completeness >=50%, contamination <=10%), and low quality for everything else — each bin receives exactly one label. Aggregate counts per category and write a text report (checkm_report.txt) listing category counts followed by per-bin details; optionally render the completeness/contamination 2D scatter plot (checkm-plot.pdf) with matplotlib. The rationale is that marker-gene-based quality classification (as implemented by CheckM2's machine-learning models) lets Mapler compare assemblers by how much diversity they recover at medium quality or better.

## Related tools

- **CheckM2** (Predicts per-bin completeness and contamination with machine-learning models, providing the numeric criteria the classification thresholds are applied to) — https://github.com/chklovski/CheckM2
- **Mapler** (Pipeline that wraps binning, CheckM2 assessment and the quality-category classification, emitting checkm_report.txt and checkm-plot.pdf) — https://github.com/Nimauric/Mapler
- **MetaBAT2** (Produces the assembly bins that are subsequently classified into quality categories) — https://bitbucket.org/berkeleylab/metabat
- **matplotlib** (Renders the completeness/contamination 2D scatter plot (checkm-plot.pdf))

## Examples

```
import pandas as pd; df = pd.read_csv('checkm/quality_report.tsv', sep='\t')
```

## Evaluation signals

- Every bin in quality_report.tsv receives exactly one of the four labels (near complete, high, medium, low) — no bin unclassified, none double-labeled
- Per-category counts sum to the total number of bins in the input quality_report.tsv
- Boundary cases respect the GSC thresholds: e.g. completeness exactly 90% with contamination 5% is high quality, not medium
- 'Near complete' is only assigned to single-contig bins meeting completeness >=99% and contamination <=1%
- checkm_report.txt mirrors the README schema: category counts followed by details on each bin

## Limitations

- CheckM2 predictions are model-based completeness/contamination estimates, so borderline bins near the 90%/50% or 5%/10% cutoffs may shift category with model or binning changes
- The single-contig requirement for 'near complete' means highly contiguous assemblies are advantaged, complicating cross-assembler comparison (e.g. metaMDBG vs hifiasm-meta)
- The README notes only part of the analysis is run on the bundled test dataset, so full classification behavior must be validated on real data

## Evidence

- [methods] GSC quality thresholds: [evidence span withheld — anchor status: not_found]
- [readme] classification purpose: "In addition to classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment, Mapler analyzes the alignment of reads on contigs."
- [readme] output schema: "checkm_report.txt # A count of near complete, high quality, medium quality and low quality bins, followed by details on each bins"
- [readme] CheckM2 basis: [evidence span withheld — anchor status: external_doc]
