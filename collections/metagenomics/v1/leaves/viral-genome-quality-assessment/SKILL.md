---
name: viral-genome-quality-assessment
description: Use when a virome/metagenome pipeline (e.g. MetaPhage) yields unexpectedly
  many vOTUs and you must classify each contig as complete, high/medium/low-quality,
  partial, or provirus to validate the excess predictions.
when_to_use_negative:
- Input contigs have not been dereplicated and length-filtered by the upstream MetaPhage
  postprocessing (minlen 1000 bp default) — quality tiers would be dominated by redundant
  or trivially short scaffolds
- The goal is taxonomic classification or network analysis of vOTUs (use vConTACT2/graphanalyzer
  instead), not completeness estimation
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: CheckV
  role: Executed in end_to_end mode to compute AAI-based and HMM-based completeness
    estimates and assign quality tiers to each vOTU
- name: MetaPhage
  role: Upstream pipeline producing the dereplicated vOTU multifasta sets (CD-HIT-EST
    at 0.95 identity, 0.85 coverage) that serve as CheckV input
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:7f0182acd471ee91f99f9ad79cd850ecc51f323664e27971b5b4ea06a55fdc87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MattiaPandolfoVR/MetaPhage
  tools:
  - CheckV
  - MetaPhage
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# viral-genome-quality-assessment

## Summary

Assesses the quality/completeness of predicted viral contigs (vOTUs) with CheckV to determine whether a phage-mining pipeline's predictions are complete, partial, or low-quality genomes. In the MetaPhage validation it explained why 5,284 predicted vOTUs far exceeded the 1,029 phage contigs reported by Liang et al.

## When to use

Use when a virome/metagenome pipeline (e.g. MetaPhage) yields unexpectedly many vOTUs and you must classify each contig as complete, high/medium/low-quality, partial, or provirus to validate the excess predictions.

## When NOT to use

- Input contigs have not been dereplicated and length-filtered by the upstream MetaPhage postprocessing (minlen 1000 bp default) — quality tiers would be dominated by redundant or trivially short scaffolds
- The goal is taxonomic classification or network analysis of vOTUs (use vConTACT2/graphanalyzer instead), not completeness estimation

## Inputs

- Dereplicated vOTU multifasta files (output of MetaPhage CD-HIT-EST dereplication at 0.95 identity / 0.85 coverage)
- CheckV reference database

## Outputs

- Per-contig CheckV quality_summary with tier calls (complete, high-quality, medium-quality, low-quality, partial, provirus)
- Aggregated per-dataset quality-tier counts written to checkv_quality_summary.tsv

## How to apply

Take the dereplicated vOTU multifasta sets produced by MetaPhage's CD-HIT-EST dereplication (identity 0.95, word size 9, alignment coverage 0.85) for each dataset (e.g. VLP virome and shotgun metagenome assemblies). Run CheckV in end_to_end mode so gene prediction, AAI-based completeness, and HMM-based completeness modules execute against the CheckV reference database. Parse the per-contig quality_summary output and assign each vOTU to complete, high-quality, medium-quality, low-quality, partial, or provirus tiers. Aggregate tier counts per dataset and compare across datasets; for the Liang et al. infant data this reproduces 569 vs 397 high-quality/complete vOTUs (VLP vs shotgun) and 113 vs 757 partial/low-quality/provirus calls, revealing that the extra MetaPhage predictions are largely partial and low-quality genomes.

## Related tools

- **CheckV** (Executed in end_to_end mode to compute AAI-based and HMM-based completeness estimates and assign quality tiers to each vOTU)
- **MetaPhage** (Upstream pipeline producing the dereplicated vOTU multifasta sets (CD-HIT-EST at 0.95 identity, 0.85 coverage) that serve as CheckV input) — https://github.com/MattiaPandolfoVR/MetaPhage

## Examples

```
checkv end_to_end vOTUs_vlp.fna checkv_out_vlp -t 16 && checkv end_to_end vOTUs_shotgun.fna checkv_out_shotgun -t 16
```

## Evaluation signals

- Aggregated VLP dataset counts reproduce 569 high-quality/complete vOTUs
- Aggregated shotgun dataset counts reproduce 397 high-quality/complete vOTUs
- Partial/low-quality plus provirus counts equal 113 (VLP) and 757 (shotgun), showing the excess of MetaPhage vOTUs over Liang et al.'s 1,029 phage contigs is attributable to partial and low-quality genome predictions
- Every vOTU in the multifasta receives exactly one quality-tier call in the parsed quality_summary output
- Output table checkv_quality_summary.tsv contains both per-vOTU calls and per-dataset aggregated counts

## Limitations

- CheckV completeness estimates depend on its reference database; vOTUs without close references may be mis-tiered
- MetaPhage's phage miners are sensitive, so many predictions are partial/low-quality — vOTU counts alone should not be interpreted as complete genome counts
- Users are urged to be mindful of biases inherent to virome analyses affecting recall/precision of virus prediction

## Evidence

- [discussion] CheckV quality assessment explained the excess vOTUs: "Quality assessment with CheckV (29) showed that the large difference in number of vOTUs predicted was due to predictions of more partial and low-quality genomes"
- [discussion] vOTU dereplication parameters feeding CheckV: "with a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85"
- [intro] vOTU clustering rationale: "clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity"
- [methods] Upstream scaffold length filter: "Specify the minimal length in bp for a viral _consensus_ scaffold. Default is `1000`."
