---
name: sliding-window-quality-filtering-of-fastq-reads
description: Use when raw 454 or Illumina amplicon reads (e.g. 16S rRNA gene FASTQ)
  show deteriorating 3'-end quality and must be trimmed and filtered before OTU clustering
  in a pipeline such as MICCA.
when_to_use_negative:
- Input is already a quality-filtered FASTQ or an OTU table — re-filtering would further
  shrink the dataset
- The analysis requires UPARSE-style stringent truncation at the first base below
  a quality threshold (e.g. reproducing UPARSE benchmark behaviour), since this skill
  deliberately keeps more reads
- Reads are untrimmed raw amplicon data still containing primer sequences — run the
  Cutadapt-based trim step first
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: sickle
  role: Sliding-window adaptive quality trimming tool for FASTQ files; performs the
    windowed 5'/3'-end trimming and minimum-length discarding
  repo: https://github.com/najoshi/sickle
- name: MICCA
  role: Pipeline whose 'filter' command wraps sickle and VSEARCH to combine sliding-window
    trimming, N-trimming, and length filtering, applying a maximum expected-error
    (EE) criterion
  repo: https://github.com/compmetagen/micca
- name: VSEARCH
  role: Wrapped by MICCA's filter command as part of the one-pass quality-filtering
    step
- name: Cutadapt
  role: Used upstream in MICCA's trim command to remove primer sequences with IUPAC
    degenerate motifs before quality filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:a5e080827f28be792effdfed08259639284f81697c81baca7177ded4c9e7ce15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/najoshi/sickle
  tools:
  - Cutadapt
  - MICCA
  - VSEARCH
  - sickle
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# sliding-window-quality-filtering-of-fastq-reads

## Summary

Adaptive quality trimming and filtering of FASTQ amplicon reads using a sliding window over per-base quality scores, as implemented by sickle and wrapped in MICCA's filter command, discarding reads whose expected error exceeds the allowed maximum or whose trimmed length falls below a threshold.

## When to use

Use when raw 454 or Illumina amplicon reads (e.g. 16S rRNA gene FASTQ) show deteriorating 3'-end quality and must be trimmed and filtered before OTU clustering in a pipeline such as MICCA.

## When NOT to use

- Input is already a quality-filtered FASTQ or an OTU table — re-filtering would further shrink the dataset
- The analysis requires UPARSE-style stringent truncation at the first base below a quality threshold (e.g. reproducing UPARSE benchmark behaviour), since this skill deliberately keeps more reads
- Reads are untrimmed raw amplicon data still containing primer sequences — run the Cutadapt-based trim step first

## Inputs

- Primer-trimmed single-end FASTQ file of 16S amplicon reads (e.g. 454 reads with a realistic error model)
- Quality-score encoding specification (Sanger, Solexa, or Illumina CASAVA 1.3-1.7)
- Quality threshold, minimum-length threshold, and maximum expected error (EE) setting

## Outputs

- filtered_reads.fastq — a single FASTQ file containing only reads that passed sliding-window trimming, length and expected-error filters
- Optionally a discarded-reads record or 'N' record per truncated read (sickle -n option)

## How to apply

After primer removal with MICCA trim (Cutadapt), run MICCA filter, which wraps VSEARCH and sickle, on the primer-trimmed FASTQ to apply sliding-window adaptive quality trimming, N-trimming, and minimum/maximum length filtering in one pass. Sickle slides a window whose length is 0.1 times the read length along the quality values, trimming the 5'-end where average window quality rises above the threshold and the 3'-end where it drops below it, e.g. 'sickle se -f input.fastq -t sanger -o trimmed.fastq -q 20 -l 40'. Reads shorter than the minimum length after trimming are discarded, and MICCA additionally filters low-quality sequences according to the maximum allowed expected error (EE), so reads whose expected error exceeds the maximum are dropped. Write surviving reads to a single output FASTQ (e.g. filtered_reads.fastq). Rationale: this flexible filtering strategy uses sequenced reads more efficiently than UPARSE's truncation at the first low-quality base, which loses a large fraction of sequence data.

## Related tools

- **sickle** (Sliding-window adaptive quality trimming tool for FASTQ files; performs the windowed 5'/3'-end trimming and minimum-length discarding) — https://github.com/najoshi/sickle
- **MICCA** (Pipeline whose 'filter' command wraps sickle and VSEARCH to combine sliding-window trimming, N-trimming, and length filtering, applying a maximum expected-error (EE) criterion) — https://github.com/compmetagen/micca
- **VSEARCH** (Wrapped by MICCA's filter command as part of the one-pass quality-filtering step)
- **Cutadapt** (Used upstream in MICCA's trim command to remove primer sequences with IUPAC degenerate motifs before quality filtering)

## Examples

```
sickle se -f input_file.fastq -t sanger -o trimmed_output_file.fastq -q 20 -l 40
```

## Evaluation signals

- Output FASTQ contains only reads whose average windowed quality passed the threshold and whose length is ≥ the minimum-length setting
- Every surviving read has expected error (EE) at or below the allowed maximum; reads exceeding it are absent from filtered_reads.fastq
- Read count in the output is comparable to QIIME's discard rate ('the number of reads discarded by MICCA and QIIME are very similar') and substantially higher retention than UPARSE's 42.3% on the 16S-10 dataset
- No residual primer sequence and no internal 'N' bases in output reads (N-trimming / truncation-at-first-N option was applied)
- Quality encoding declared to sickle (Sanger, Solexa, or Illumina) matches the input FASTQ's Phred offset so trimming positions are computed correctly

## Limitations

- Sickle's Solexa quality setting is an approximation because the actual conversion is a non-linear transformation; only Sanger, Solexa, and Illumina CASAVA 1.3-1.7 encodings are supported
- Discarding behaviour depends on thresholds: reads below the minimum length after trimming are discarded entirely, so overly strict thresholds lose a large fraction of sequence data
- Compared with MICCA's flexible strategy, UPARSE-style filtering (truncate at first base below quality threshold plus minimum length) retains far fewer reads, so results are not interchangeable between pipelines

## Evidence

- [readme] sickle README: windowed trimming algorithm: [evidence span withheld — anchor status: external_doc]
- [readme] sickle README: discard on length: [evidence span withheld — anchor status: external_doc]
- [other] MICCA README: expected-error filtering: "filters low quality sequences according to the maximum allowed **expected
  error (EE) rate** %"
- [results] Article results: UPARSE contrast: "the stringent quality filtering approach used by UPARSE4, which is based on truncating the reads at the first base position which has a quality score less than a specified threshold"
- [discussion] Article discussion: filtering flexibility: "MICCA, implementing a more flexible filtering strategy, tends to use more efficiently the sequenced reads"
- [results] Article results: comparison with QIIME discard rates: "While the number of reads discarded by MICCA and QIIME are very similar"
