---
name: read-mapping-coverage-calculation
description: Use when viral scaffolds (e.g. from VIBRANT or VirSorter prediction)
  from one or more metagenomes need coverage abundance features before binning scaffolds
  into vMAGs with vRhyme.
when_to_use_negative:
- Input is already a pre-computed coverage table compatible with vRhyme's -c flag
  — no read alignment is needed
- You intend to bin microbial (non-viral) genomes; vRhyme is not meant to bin microbes
  and performance on whole metagenomes is not fully evaluated
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: vRhyme
  role: Automatically calculates and refines read coverage (end masking, outlier removal)
    from alignments or raw reads
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Bowtie2
  role: Short-read aligner for mapping metagenomic reads to viral scaffolds (--no-unal
    --no-discordant)
  repo: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- name: BWA
  role: Alternative aligner (BWA mem) for computing per-sample scaffold coverage
  repo: https://github.com/lh3/bwa
- name: Samtools
  role: Required dependency for coverage calculation when coverage input other than
    -c is used
  repo: http://www.htslib.org/
- name: PySam
  role: Python wrapper of samtools C-API used by vRhyme to read SAM/BAM alignments
    during coverage computation
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:ff0158b45baeed99f75f460cb42c9e1c60de6089de1cb39aa238a6056eaccfe0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - BWA
  - Bowtie2
  - Mmseqs2
  - Prodigal
  - PySam
  - Python
  - Samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# read-mapping-coverage-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-sample read coverage profiles for viral scaffolds by mapping metagenomic reads with Bowtie2 or BWA and refining them in vRhyme, providing the coverage variance features that drive vMAG binning.

## When to use

Use when viral scaffolds (e.g. from VIBRANT or VirSorter prediction) from one or more metagenomes need coverage abundance features before binning scaffolds into vMAGs with vRhyme.

## When NOT to use

- Input is already a pre-computed coverage table compatible with vRhyme's -c flag — no read alignment is needed
- You intend to bin microbial (non-viral) genomes; vRhyme is not meant to bin microbes and performance on whole metagenomes is not fully evaluated

## Inputs

- Viral scaffold FASTA (length-filtered, minimum 2000bp) from viral prediction
- Paired/interleaved or unpaired metagenomic reads (FASTQ) from each sample in the cohort

## Outputs

- Per-sample scaffold coverage table used by vRhyme for coverage effect size comparisons
- Refined coverage values with scaffold-end masking and outlier removal

## How to apply

Align metagenomic reads to the length-filtered viral scaffold FASTA using Bowtie2 (with --no-unal --no-discordant for paired/interleaved reads) or BWA mem, feeding the reads directly to vRhyme via -r/-v/-u so coverage is computed automatically with Samtools/pysam. Mask 150bp at scaffold ends and remove coverage outliers more than 4.0 standard deviations above the mean (--outliers 4.0) to prevent spurious coverage signal. Supply per-sample coverage from the full cohort (the human skin run used 270 samples) because vRhyme relies on single- or multi-sample coverage effect size comparisons between scaffolds, and it performs optimally with multiple input samples for more robust coverage variance comparisons. The resulting coverage table becomes a core feature, alongside supervised machine learning classification of sequence features, for building the weighted networks that are refined into bins.

## Related tools

- **vRhyme** (Automatically calculates and refines read coverage (end masking, outlier removal) from alignments or raw reads) — https://github.com/AnantharamanLab/vRhyme
- **Bowtie2** (Short-read aligner for mapping metagenomic reads to viral scaffolds (--no-unal --no-discordant)) — http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- **BWA** (Alternative aligner (BWA mem) for computing per-sample scaffold coverage) — https://github.com/lh3/bwa
- **Samtools** (Required dependency for coverage calculation when coverage input other than -c is used) — http://www.htslib.org/
- **PySam** (Python wrapper of samtools C-API used by vRhyme to read SAM/BAM alignments during coverage computation) — https://github.com/pysam-developers/pysam

## Examples

```
vRhyme -i viral_scaffolds.fasta -r sample1_1.fq -v sample1_2.fq --outliers 4.0 --min_len 2000 -t 15 -o vRhyme_out
```

## Evaluation signals

- Coverage table has one column per input sample (e.g. 270 human skin samples) and one row per scaffold
- Scaffold-end masking of 150bp and --outliers 4.0 removal applied — no coverage values exceed mean + 4 SD from unmasked regions
- test_vRhyme.py reports Samtools as Success when coverage inputs other than -c are used
- Downstream vRhyme run reconstructs the expected bins (e.g. 849 vMAGs from 10,601 human skin scaffolds; 22-scaffold Herelleviridae vMAG at ~100% CheckV completeness), indicating coverage features were informative
- Alignment used --no-unal --no-discordant for paired/interleaved reads so unmapped and discordant pairs do not inflate coverage

## Limitations

- vRhyme performs optimally with multiple input samples; coverage variance comparisons from a single sample are less robust, and the optimal number depends on how the dataset was constructed
- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome, which coverage signal alone cannot resolve
- If reads are not provided (-r/-v/-u), Bowtie2 and BWA are optional but Samtools becomes required for any coverage input besides -c

## Evidence
