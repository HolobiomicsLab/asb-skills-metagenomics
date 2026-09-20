---
name: quality-metric-aggregation-and-tabulation
description: Use when you have per-contig quality assessments (e.g., CheckV complete/high/medium/low/partial/provirus
  calls) for vOTU sets from two or more datasets and need aggregated tier counts to
  explain differences in predicted viral richness.
when_to_use_negative:
- Input is raw contigs that have not been dereplicated into vOTUs or assessed by CheckV
  — there are no per-contig quality tiers to aggregate yet
- The analysis goal is de novo viral identification or assembly rather than summarizing
  and comparing existing quality assessments
- Only a single dataset with no comparison target exists, so cross-dataset tabulation
  of tier counts provides no added value
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0797
tools:
- name: CheckV
  role: Produces the per-contig genome-quality assessment (complete, high/medium/low
    quality, partial, provirus) whose summary is parsed and aggregated into the tabular
    comparison
- name: MetaPhage
  role: Upstream pipeline that generates the dereplicated vOTU multifasta inputs via
    CD-HIT-EST (0.95 identity, 0.85 alignment coverage) that are then quality-assessed
    and tabulated
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:842055e423ac07382ccfe5a090ba157bcf85990fe0a28c2b4930a3b467a1b6de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MattiaPandolfoVR/MetaPhage
  tools:
  - CheckV
  - MetaPhage
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# quality-metric-aggregation-and-tabulation

## Summary

Aggregate per-contig genome-quality calls (e.g., CheckV quality_summary tiers) into dataset-level counts and write them to a tabular summary (checkv_quality_summary.tsv), enabling head-to-head comparison of viral prediction quality across datasets.

## When to use

Use when you have per-contig quality assessments (e.g., CheckV complete/high/medium/low/partial/provirus calls) for vOTU sets from two or more datasets and need aggregated tier counts to explain differences in predicted viral richness.

## When NOT to use

- Input is raw contigs that have not been dereplicated into vOTUs or assessed by CheckV — there are no per-contig quality tiers to aggregate yet
- The analysis goal is de novo viral identification or assembly rather than summarizing and comparing existing quality assessments
- Only a single dataset with no comparison target exists, so cross-dataset tabulation of tier counts provides no added value

## Inputs

- Dereplicated vOTU multifasta sets from MetaPhage CD-HIT-EST dereplication (0.95 identity, word size 9, 0.85 coverage), one per dataset (VLP virome and shotgun metagenome assemblies)
- CheckV per-contig quality_summary output from end_to_end mode runs against the CheckV reference database

## Outputs

- checkv_quality_summary.tsv with aggregated quality-tier counts per dataset and per-vOTU quality calls
- VLP-vs-shotgun comparison table (e.g., high-quality/complete: 569 vs 397; partial/low-quality plus provirus: 113 vs 757)

## How to apply

Run CheckV in end_to_end mode on each dereplicated vOTU multifasta (produced by MetaPhage's CD-HIT-EST dereplication at 0.95 identity, word size 9, 0.85 alignment coverage) so gene prediction, AAI-based completeness, and HMM-based completeness modules all execute against the CheckV reference database. Parse the per-contig quality_summary output and classify each vOTU as complete, high-quality, medium-quality, low-quality, partial, or provirus. Aggregate the tier counts per dataset, group partial/low-quality with provirus calls, and tabulate the cross-dataset comparison into checkv_quality_summary.tsv. In the MetaPhage validation on Liang et al. infant VLP and shotgun metagenomes, this aggregation revealed that MetaPhage's excess vOTU counts (5,284 vs 1,029 originally reported) came from more partial and low-quality genome predictions.

## Related tools

- **CheckV** (Produces the per-contig genome-quality assessment (complete, high/medium/low quality, partial, provirus) whose summary is parsed and aggregated into the tabular comparison)
- **MetaPhage** (Upstream pipeline that generates the dereplicated vOTU multifasta inputs via CD-HIT-EST (0.95 identity, 0.85 alignment coverage) that are then quality-assessed and tabulated) — https://github.com/MattiaPandolfoVR/MetaPhage

## Examples

```
checkv end_to_end votus_vlp.fna checkv_out_vlp && python parse_quality_summary.py checkv_out_vlp/quality_summary.tsv --group partial,low-quality,provirus -o checkv_quality_summary.tsv
```

## Evaluation signals

- Aggregated counts reproduce the reported validation figures: 569 high-quality/complete vOTUs in the VLP dataset and 397 in the shotgun dataset
- Partial/low-quality/provirus group totals match: 757 in the shotgun dataset versus 113 in the VLP dataset
- Every vOTU in the input multifasta appears exactly once in the per-vOTU quality calls (no dropped or duplicated contigs)
- checkv_quality_summary.tsv contains a quality-tier assignment for each vOTU drawn from the complete/high-quality/medium-quality/low-quality/partial/provirus vocabulary
- Tier counts per dataset sum to the total number of dereplicated vOTUs in that dataset

## Limitations

- The excess vOTU counts reflect predictions of more partial and low-quality genomes, so raw vOTU totals are not comparable without quality stratification — users are urged to be mindful of biases inherent to virome analyses
- Interpretation depends on the CheckV reference database version and the completeness of AAI/HMM signal; contigs lacking reference relatives may be mis-tiered

## Evidence

- [discussion] Quality assessment with CheckV (29) showed that the large difference in number of vOTUs predicted was due to predictions of more partial and low-quality genomes: "Quality assessment with CheckV (29) showed that the large difference in number of vOTUs predicted was due to predictions of more partial and low-quality genomes"
- [discussion] CD-HIT-EST dereplication parameters: "with a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85"
- [intro] clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity: "clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity"
