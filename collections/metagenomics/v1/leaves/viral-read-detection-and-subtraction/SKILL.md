---
name: viral-read-detection-and-subtraction
description: Use when retained reads from host read removal show anomalously high
  retention (e.g. NA12878 0.641% vs 0.406% after correction) and viral contamination
  such as EBV is suspected in 1000 Genomes or similar Illumina data.
when_to_use_negative:
- Input is raw host-mixed FASTQ that has not yet passed hostile clean against a human
  index — viral subtraction is a second-pass step on retained reads
- The analysis goal is host read removal itself rather than quantifying/subtracting
  viral contamination inflating retention figures
- The retained reads come from data where EBV or other viral contamination is expected
  biological signal (e.g. tumor virome studies) rather than a contaminant to remove
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Hostile
  role: Subtracts viral reads by running hostile clean with --index pointing at a
    custom EBV index; JSON logs supply reads_in/reads_out/reads_removed_proportion
    for recomputing retention
  repo: https://github.com/bede/hostile
- name: MEGAHIT
  role: De novo assembly of the Hostile-retained reads per genome to generate contigs
    screened for EBV representation
- name: Minimap2
  role: Aligns retained reads and assemblies against the EBV reference genomes NC_007605.1
    and NC_009334.1; also used by Hostile for long-read/custom-index cleaning
- name: Bowtie2
  role: Short-read alignment backend used for screening and removing reads aligning
    to the viral index
- name: Bedtools
  role: Used with Minimap2 to mask aligned regions with N when building custom masked
    indexes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:1fa74ba15d191ef5842a5e1c3c79d391e58c513262e317f327d4c525e0f66426
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bedtools
  - Bowtie2
  - Hostile
  - MEGAHIT
  - Minimap2
  - Samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Viral read detection and subtraction

## Summary

Detects viral contamination (e.g. Epstein-Barr Virus, NC_007605.1/NC_009334.1) in host-depleted FASTQs via de novo assembly and alignment, then subtracts viral reads with a custom Hostile index so host read retention metrics are not inflated by viral contaminant reads.

## When to use

Use when retained reads from host read removal show anomalously high retention (e.g. NA12878 0.641% vs 0.406% after correction) and viral contamination such as EBV is suspected in 1000 Genomes or similar Illumina data.

## When NOT to use

- Input is raw host-mixed FASTQ that has not yet passed hostile clean against a human index — viral subtraction is a second-pass step on retained reads
- The analysis goal is host read removal itself rather than quantifying/subtracting viral contamination inflating retention figures
- The retained reads come from data where EBV or other viral contamination is expected biological signal (e.g. tumor virome studies) rather than a contaminant to remove

## Inputs

- Hostile-cleaned retained paired FASTQ files (*.clean_1.fastq.gz / *.clean_2.fastq.gz) from 27 Byrska-Bishop et al. Illumina 2x150bp genomes
- Viral reference sequences (EBV accessions NC_007605.1 and NC_009334.1)
- Hostile JSON logs with reads_in/reads_out/reads_removed_proportion fields from the upstream clean runs

## Outputs

- MEGAHIT contigs per sample for viral screening
- Custom viral Hostile index built from NC_007605.1/NC_009334.1
- Viral-subtracted retained FASTQs
- Metrics table of per-sample retention before and after viral subtraction (overall 0.594% -> 0.393%; NA12878 0.641% -> 0.406%)
- Final adjusted retention figure and flag of ERR3242202 at ~1.3% EBV read proportion

## How to apply

Start from the Hostile-cleaned retained paired FASTQs (*.clean_1.fastq.gz / *.clean_2.fastq.gz) from the upstream hostile clean runs. De novo assemble each sample's retained reads with MEGAHIT, then screen assemblies and reads for viral representation by aligning to the EBV references NC_007605.1 and NC_009334.1 with Minimap2/Bowtie2. For samples with viral signal (e.g. ERR3242202 at ~1.3% EBV), build a custom index from those accessions and run hostile clean with --index pointed at the custom genome file to subtract viral reads. Recompute per-sample retention from the reads_in/reads_out/reads_removed_proportion fields of the Hostile JSON logs, before and after subtraction. The rationale is that viral reads that escape the human index inflate apparent host retention, so subtracting them (from 0.594% to 0.393% overall across the 27 Byrska-Bishop et al. 2x150bp genomes) yields the true human retention figure.

## Related tools

- **Hostile** (Subtracts viral reads by running hostile clean with --index pointing at a custom EBV index; JSON logs supply reads_in/reads_out/reads_removed_proportion for recomputing retention) — https://github.com/bede/hostile
- **MEGAHIT** (De novo assembly of the Hostile-retained reads per genome to generate contigs screened for EBV representation)
- **Minimap2** (Aligns retained reads and assemblies against the EBV reference genomes NC_007605.1 and NC_009334.1; also used by Hostile for long-read/custom-index cleaning)
- **Bowtie2** (Short-read alignment backend used for screening and removing reads aligning to the viral index)
- **Bedtools** (Used with Minimap2 to mask aligned regions with N when building custom masked indexes)

## Examples

```
hostile clean --fastq1 ERR3242202.clean_1.fastq.gz --fastq2 ERR3242202.clean_2.fastq.gz --index custom-ebv-NC_007605.1-NC_009334.1 --aligner minimap2
```

## Evaluation signals

- After EBV subtraction, overall human retention for the 27 2x150bp genomes must drop from 0.594% to 0.393%
- NA12878 retention must drop from 0.641% to 0.406% after EBV reads are removed
- ERR3242202 must be flagged with an EBV read proportion of approximately 1.3%
- Per-sample retention values recomputed from the Hostile JSON logs (reads_in/reads_out/reads_removed_proportion) must match the reported metrics table
- MEGAHIT contigs must align to NC_007605.1/NC_009334.1 in every one of the 27 genomes for the EBV-representation claim to hold

## Limitations

- The default human-t2t-hla index does not remove EBV reads, so viral contamination silently inflates reported host retention unless a custom viral subtraction pass is added
- EBV contamination is sample-specific (e.g. ~1.3% in ERR3242202 vs lower levels elsewhere), so per-sample screening is required rather than a single global correction
- Earlier Hostile versions could corrupt empty fastq.gz files when no reads remain after decontamination, and did not verify downloaded index checksums (issue #20)

## Evidence

- [intro] Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin.: "Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin."
- [readme] For best possible retention of microbial reads, optionally use an existing index masked against bacterial and/or viral genomes, or make your own using the built-in masking utility.: "For best possible retention of microbial reads, optionally use an existing index masked against bacterial and/or viral genomes, or make your own using the built-in masking utility."
- [methods] Masking uses Minimap2 to align 150mers of the supplied target genomes with the reference genome, and bedtools to mask all aligned regions with N.: "Masking uses Minimap2 to align 150mers of the supplied target genomes with the reference genome, and bedtools to mask all aligned regions with N."
- [methods] Remove reads aligning to an index from fastq[.gz] input files or stdin.: "Remove reads aligning to an index from fastq[.gz] input files or stdin."
