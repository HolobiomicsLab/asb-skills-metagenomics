---
name: wastewater-ngs-read-alignment
description: Use when raw paired-end FASTQ reads from SARS-CoV-2 wastewater sequencing
  (e.g., ENA study PRJEB44141) must be mapped to the SARS-CoV-2 Wuhan reference NC_045512
  before low-frequency variant calling.
when_to_use_negative:
- Input is already a VCF from variant calling — this skill operates upstream on FASTQ
  reads
- Reads are single-end or contain unmatched/multiple mappings that cannot be filtered
  to paired-end pairs
- The target organism or reference is not the SARS-CoV-2 Wuhan genome NC_045512 (e.g.,
  host or metagenomic assembly workflows)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Minimap2
  role: Maps cleaned paired-end reads to the SARS-CoV-2 reference genome NC_045512
    with a minimal chaining score of 40
- name: lineagespot
  role: Downstream R framework that consumes the alignment-derived variant calls;
    its GitHub repository documents the raw-data (read processing and mapping) steps
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:d0c3ee451c72ff7b5e2055580a25d3c171cacd71293fb4a53ed360b1582c7bb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BiodataAnalysisGroup/lineagespot
  tools:
  - GATK Mutect2
  - Minimap2
  - R
  - freebayes
  - lineagespot
  - mpileup
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# wastewater-ngs-read-alignment

## Summary

Aligns cleaned paired-end NGS reads from SARS-CoV-2 wastewater samples to the Wuhan reference genome NC_045512 using Minimap2 with a minimal chaining score of 40, as a variation of the Illumina ARTIC pipeline, to prepare BAM alignments for downstream variant calling with freebayes, mpileup, or GATK Mutect2.

## When to use

Use when raw paired-end FASTQ reads from SARS-CoV-2 wastewater sequencing (e.g., ENA study PRJEB44141) must be mapped to the SARS-CoV-2 Wuhan reference NC_045512 before low-frequency variant calling.

## When NOT to use

- Input is already a VCF from variant calling — this skill operates upstream on FASTQ reads
- Reads are single-end or contain unmatched/multiple mappings that cannot be filtered to paired-end pairs
- The target organism or reference is not the SARS-CoV-2 Wuhan genome NC_045512 (e.g., host or metagenomic assembly workflows)

## Inputs

- Paired-end FASTQ reads from wastewater wastewater NGS sequencing (e.g., ENA study PRJEB44141 / ERP128154)
- Quality-trimmed FASTQ reads with sliding window quality threshold 15 (width 4) and minimum read length 200 nt
- SARS-CoV-2 Wuhan reference genome NC_045512 in FASTA format

## Outputs

- Alignment file (SAM/BAM) of cleaned paired-end reads mapped to NC_045512
- Aligned reads ready for variant calling with freebayes, mpileup, or GATK Mutect2

## How to apply

First retrieve the paired-end FASTQ reads (e.g., from the ENA study PRJEB44141/ERP128154) and clean them by trimming with a sliding-window quality threshold of 15 (window width 4), retaining only reads with a minimum length of 200 nt after trimming. Retain only paired-end sequences and remove unmatched and multiple mappings. Map the cleaned reads to the SARS-CoV-2 Wuhan reference genome NC_045512 using Minimap2 with a minimal chaining score (matching bases minus log gap penalty) of 40, following the variation of the Illumina ARTIC pipeline used by the paper. The resulting alignments feed directly into the three variant callers (freebayes at low mutation frequency 0.01, mpileup reporting every position, GATK Mutect2 cancer-only mode) so alignment parameters directly affect downstream lineage detection via lineagespot. Note that the paper reports an inconsistency to avoid: primer trimming and remapping were omitted for four samples (19 March–15 April 2021), so apply trimming and remapping consistently across samples.

## Related tools

- **Minimap2** (Maps cleaned paired-end reads to the SARS-CoV-2 reference genome NC_045512 with a minimal chaining score of 40)
- **lineagespot** (Downstream R framework that consumes the alignment-derived variant calls; its GitHub repository documents the raw-data (read processing and mapping) steps) — https://github.com/BiodataAnalysisGroup/lineagespot

## Examples

```
minimap2 -x sr -a NC_045512.fa sample_R1.trimmed.fq.gz sample_R2.trimmed.fq.gz | samtools sort -o sample.mapped.bam
```

## Evaluation signals

- All retained alignments derive from paired-end reads; unmatched and multiple mappings are removed
- Trimmed reads meet the minimum length of 200 nt after sliding-window quality trimming at threshold 15
- Mapped reads align to the correct reference (SARS-CoV-2 Wuhan variant, NC_045512)
- Downstream variant calling on the alignments reproduces the paper's conclusion that freebayes at 0.01 mutation frequency is the most productive and informative caller for lineages B.1.177, B.1.1.7, and B.1.351
- Primer trimming and remapping are applied consistently to all samples (unlike the paper's four samples from 19 March–15 April 2021, where they were omitted)

## Limitations

- Inconsistent preprocessing across samples in the original work: primer trimming and remapping to the reference genome were omitted for the four samples between 19 March 2021 and 15 April 2021
- No software version reported for Minimap2, which may hinder exact reproduction of alignments
- Mutation identification downstream can be hindered by viral adsorption and decay in wastewater, independent of alignment quality

## Evidence

- [discussion] Minimap2 chaining score: "mapped to the SARS-CoV-2 reference genome (Wuhan variant, NC_045512), using Minimap2 tool19 with a minimal chaining score"
- [discussion] Paired-end filtering: "only the paired-end sequences were retained, while any other (unmatched, multiple mappings, etc.) were removed"
- [discussion] Trimming thresholds: "a minimum of 200 length in nucleotides for a read to be retained after trimming, and a minimum threshold for sliding window of 15 quality"
- [results] Data source: "retrieved from the ENA study ID: PRJEB44141 (ERP128154)"
- [readme] Raw data analysis documentation: [evidence span withheld — anchor status: external_doc]
