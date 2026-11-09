---
name: fastq-record-length-and-phred-thresholding
description: Use when raw pooled/subsampled ONT long-read FASTQs (e.g. .fastq.gz from
  a MinION run) must be quality-trimmed before plasmid assembly, and every retained
  record must meet a length and Q-score cutoff.
when_to_use_negative:
- Input is Illumina short-read FASTQ — the Plassembler pipeline handles short reads
  with fastp (>=0.24.2), not chopper.
- Reads are already error-corrected or subsampled assemblies/contigs, not raw long-read
  FASTQ records.
- A different length/quality regime is required by the upstream assembler (e.g. canu
  -correct subsampling to 100x depth) — thresholds must be re-derived rather than
  reusing the 500 bp/Q9 defaults.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: chopper
  role: Long-read FASTQ filter applied with minimum length 500 bp (-l 500) and minimum
    average Q-score 9 (-q 9); reads gzipped FASTQ from stdin and writes gzip-compressed
    output
  repo: https://github.com/wdecoster/chopper
- name: Plassembler
  role: Upstream pipeline whose quality-control step invokes chopper on long-read
    FASTQs with these defaults
  repo: https://github.com/gbouras13/plassembler
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:a25d344be75b282c06e7db2ab399eb43fd7d243a71bb0f80c600808a4277cb7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wdecoster/chopper
  tools:
  - Plassembler
  - chopper
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# fastq-record-length-and-phred-thresholding

## Summary

Filters a long-read FASTQ by dropping records below a minimum length and minimum average Phred score, using chopper as Plassembler's quality-control step (defaults of 500 bp and Q9). It prevents low-quality, very short ONT reads from entering assembly or mapping.

## When to use

Use when raw pooled/subsampled ONT long-read FASTQs (e.g. .fastq.gz from a MinION run) must be quality-trimmed before plasmid assembly, and every retained record must meet a length and Q-score cutoff.

## When NOT to use

- Input is Illumina short-read FASTQ — the Plassembler pipeline handles short reads with fastp (>=0.24.2), not chopper.
- Reads are already error-corrected or subsampled assemblies/contigs, not raw long-read FASTQ records.
- A different length/quality regime is required by the upstream assembler (e.g. canu -correct subsampling to 100x depth) — thresholds must be re-derived rather than reusing the 500 bp/Q9 defaults.

## Inputs

- Gzipped Oxford Nanopore long-read FASTQ (.fastq.gz), optionally pooled and subsampled (e.g. 60x ONT read set from zenodo.org/record/7996690)

## Outputs

- Gzip-compressed filtered FASTQ (filtered_longreads.fastq.gz) containing only records >=500 bp with average Q-score >=9

## How to apply

Stream the gzipped long-read FASTQ (e.g. a pooled subsampled 60x ONT file such as Klebsiella_variicola_subsampled60x.fastq.gz) from stdin into chopper (>=0.11.0), which is a Rust implementation of NanoFilt+NanoLyse that filters based on average read quality and minimum read length. Apply the Plassembler QC defaults: minimum read length of 500 bp and minimum average quality of Q-score 9, reading gzipped FASTQ from stdin and emitting gzip-compressed FASTQ to stdout. Rationale: Plassembler replaced nanofilt with chopper for this read-filtering step, so these thresholds reproduce Plassembler's documented QC defaults on the 60x ONT read sets used in its benchmarking. Decision point: raise -l/-q for very high-quality R10.4 data or lower them for marginal runs, but 500/Q9 is the Plassembler default pairing.

## Related tools

- **chopper** (Long-read FASTQ filter applied with minimum length 500 bp (-l 500) and minimum average Q-score 9 (-q 9); reads gzipped FASTQ from stdin and writes gzip-compressed output) — https://github.com/wdecoster/chopper
- **Plassembler** (Upstream pipeline whose quality-control step invokes chopper on long-read FASTQs with these defaults) — https://github.com/gbouras13/plassembler

## Examples

```
gunzip -c Klebsiella_variicola_subsampled60x.fastq.gz | chopper -l 500 -q 9 | gzip > filtered_longreads.fastq.gz
```

## Evaluation signals

- Every retained record in filtered_longreads.fastq.gz is >=500 bp and has an average Phred score >=9 (spot-check with seqkit or awk on the 4-line FASTQ records).
- Output is a valid gzip stream of well-formed 4-line FASTQ records with no truncation after piping through chopper.
- Record count (and total bases) of the output is strictly less than or equal to the input, and no record shorter than 500 bp survives.
- Reproducibility: running the same input through chopper -l 500 -q 9 yields an identical output FASTQ, matching Plassembler's documented QC behavior.

## Limitations

- chopper's filtering is based on average read quality, min/max length, and GC content — per-base or per-window quality issues within a retained read are not addressed.
- The 500 bp/Q9 values are Plassembler's defaults; other workflows (or versions replacing nanofilt) may legitimately use different thresholds, so the defaults should not be silently applied outside the Plassembler QC context.
- Very aggressive filtering of short reads can remove genuine small-plasmid signal — Plassembler specifically recovered small plasmids (e.g. 1934 bp in K. variicola INF345), so the length cutoff should not be raised far above 500 bp.

## Evidence

- [methods] Plassembler filters long reads with chopper using a minimum read length of 500 bp and a minimum quality of Q-score 9 by default.: [evidence span withheld — anchor status: not_found]
- [discussion] Read filtering with chopper replaced nanofilt in Plassembler.: "* Replaces nanofilt with chopper"
- [readme] Chopper filters and trims long-read FASTQ files based on average quality, length, and GC content.: [evidence span withheld — anchor status: external_doc]
- [readme] Chopper is a Rust implementation of NanoFilt and NanoLyse intended for long-read sequencing.: [evidence span withheld — anchor status: external_doc]
