---
name: vlp-vs-shotgun-dataset-comparison
description: Use when comparing phage predictions from paired VLP-enriched and shotgun
  metagenome assemblies and you need to explain differences in vOTU counts via genome-quality
  stratification rather than raw counts.
when_to_use_negative:
- Input is raw reads or unassembled contigs that have not been through MetaPhage phage
  detection and CD-HIT-EST dereplication — CheckV quality tiers assume candidate viral
  contigs
- The analysis goal is taxonomic classification of vOTUs (e.g., vConTACT2 network
  placement) rather than genome completeness/quality assessment
- Single-dataset analysis with no VLP-vs-shotgun contrast to compute
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0780
tools:
- name: CheckV
  role: Runs end_to_end quality assessment (gene prediction, AAI-based and HMM-based
    completeness) to stratify vOTUs into quality tiers
- name: MetaPhage
  role: Upstream pipeline producing the dereplicated vOTU sets (CD-HIT-EST at 0.95
    identity, word size 9, coverage 0.85) for both VLP and shotgun datasets
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:2a81f392a15aeb8aab338d9ac0d63fe8f4a7c93e825e579e8c901a7d54cc98ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MattiaPandolfoVR/MetaPhage
  tools:
  - CheckV
  - MetaPhage
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# vlp-vs-shotgun-dataset-comparison

## Summary

Reproduces the MetaPhage paper's comparison of VLP-enriched versus shotgun infant metagenomes by classifying dereplicated vOTUs with CheckV quality tiers, showing that MetaPhage's excess vOTU counts (5,284 vs 1,029 reported phage contigs) are driven by partial and low-quality genome predictions.

## When to use

Use when comparing phage predictions from paired VLP-enriched and shotgun metagenome assemblies and you need to explain differences in vOTU counts via genome-quality stratification rather than raw counts.

## When NOT to use

- Input is raw reads or unassembled contigs that have not been through MetaPhage phage detection and CD-HIT-EST dereplication — CheckV quality tiers assume candidate viral contigs
- The analysis goal is taxonomic classification of vOTUs (e.g., vConTACT2 network placement) rather than genome completeness/quality assessment
- Single-dataset analysis with no VLP-vs-shotgun contrast to compute

## Inputs

- Dereplicated vOTU multifasta files (VLP virome assembly) from MetaPhage CD-HIT-EST dereplication at 0.95 identity / 0.85 coverage
- Dereplicated vOTU multifasta files (shotgun metagenome assembly) from MetaPhage CD-HIT-EST dereplication
- CheckV reference database

## Outputs

- checkv_quality_summary.tsv with aggregated quality-tier counts per dataset (VLP vs shotgun)
- Per-vOTU CheckV quality calls (complete, high-quality, medium-quality, low-quality, partial, provirus)

## How to apply

Take the dereplicated vOTU multifasta sets produced by MetaPhage's CD-HIT-EST dereplication step (sequence identity threshold 0.95, word size 9, alignment coverage 0.85) for both the VLP virome and the shotgun metagenome assemblies. Run CheckV in end_to_end mode on each set so gene prediction, AAI-based completeness, and HMM-based completeness modules execute against the CheckV reference database. Parse the per-contig quality_summary output, classifying each vOTU into complete, high-quality, medium-quality, low-quality, partial, or provirus tiers for both datasets. Aggregate tier counts per dataset and compute the VLP-vs-shotgun comparison (high-quality/complete: 569 vs 397; partial/low-quality plus provirus: 113 vs 757) to explain the excess vOTU predictions relative to the 1,029 phage contigs originally reported by Liang et al.

## Related tools

- **CheckV** (Runs end_to_end quality assessment (gene prediction, AAI-based and HMM-based completeness) to stratify vOTUs into quality tiers)
- **MetaPhage** (Upstream pipeline producing the dereplicated vOTU sets (CD-HIT-EST at 0.95 identity, word size 9, coverage 0.85) for both VLP and shotgun datasets) — https://github.com/MattiaPandolfoVR/MetaPhage

## Examples

```
checkv end_to_end vlp_votus.fasta checkv_vlp/ && checkv end_to_end shotgun_votus.fasta checkv_shotgun/
```

## Evaluation signals

- Aggregated counts reproduce the reported figures: 569 high-quality/complete vOTUs in the VLP dataset and 397 in the shotgun dataset
- Shotgun dataset shows 757 partial/low-quality/provirus predictions versus 113 in the VLP dataset
- Every vOTU in both input multifasta files receives exactly one quality-tier call in the per-contig quality_summary output
- The VLP-to-shotgun ratio of low-quality/partial/provirus predictions explains the excess vOTU counts (5,284 MetaPhage vOTUs vs 1,029 phage contigs reported by Liang et al.)
- checkv_quality_summary.tsv contains tier counts for both datasets that sum to the number of dereplicated vOTUs in each input

## Limitations

- Users should be mindful of the biases inherent to virome analyses; VLP enrichment and shotgun sequencing produce systematically different genome-completeness profiles
- Excess vOTU counts from MetaPhage reflect more partial and low-quality genome predictions, so raw vOTU counts overestimate phage diversity relative to curated studies
- All-versus-all comparisons and network-based classification against large reference databases are time-consuming; CheckV quality assessment is an alternative lens but does not classify vOTUs taxonomically

## Evidence

- [discussion] CheckV quality assessment of vOTU tiers: "Quality assessment with CheckV (29) showed that the large difference in number of vOTUs predicted was due to predictions of more partial and low-quality genomes"
- [discussion] VLP and shotgun validation datasets: "We used previously published data sets of shotgun and VLP metagenomes of infants after birth (19) for the testing and validation of MetaPhage"
- [discussion] Dereplication parameters: "with a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85"
- [intro] vOTU clustering rationale: "clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity"
