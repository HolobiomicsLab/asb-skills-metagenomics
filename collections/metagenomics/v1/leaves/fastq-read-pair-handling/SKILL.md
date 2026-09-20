---
name: fastq-read-pair-handling
description: 'Use when reconstructing StrainXpress''s read-overlap scoring: you have
  an interleaved FASTQ of Illumina-type short reads and Minimap2 all-vs-all alignments
  with CIGAR strings, and need per-pair scores D with >=30 bp / >=0.9 identity filtering.'
when_to_use_negative:
- Input reads are third-generation long reads — the article notes the high error rate
  of such reads introduces issues that still need to be resolved
- Input is separate R1/R2 FASTQ files rather than the required single interleaved
  FASTQ
- Input is already assembled contigs — this skill operates on raw quality-controlled
  reads to seed the overlap graph
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Minimap2
  role: Performs the all-vs-all suffix-prefix overlap alignment of reads, outputting
    CIGAR strings from which identity and overlap length are parsed
  repo: https://github.com/lh3/minimap2
- name: StrainXpress
  role: Defines the overlap scoring formula (equation 1) and the 30 bp / 0.9 identity
    filtering thresholds applied to read pairs
  repo: https://github.com/HaploKit/StrainXpress
- name: Pandas
  role: Handles the tabular overlap_scores.tsv output during the scoring workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:0fd59e6cc4a0a6c5501ddde4e9e4d1e16e4972a46933fd391d7c8b3461e4ff12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Minimap2
  - Pandas
  - Python
  - StrainXpress
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# fastq-read-pair-handling

## Summary

Computes a weighted overlap score for each Minimap2-identified suffix-prefix overlap between quality-controlled NGS read pairs, combining alignment identity (weight 0.9) with relative overlap length (weight 0.1). This scoring feeds the overlap graph that StrainXpress builds for strain-aware metagenome assembly.

## When to use

Use when reconstructing StrainXpress's read-overlap scoring: you have an interleaved FASTQ of Illumina-type short reads and Minimap2 all-vs-all alignments with CIGAR strings, and need per-pair scores D with >=30 bp / >=0.9 identity filtering.

## When NOT to use

- Input reads are third-generation long reads — the article notes the high error rate of such reads introduces issues that still need to be resolved
- Input is separate R1/R2 FASTQ files rather than the required single interleaved FASTQ
- Input is already assembled contigs — this skill operates on raw quality-controlled reads to seed the overlap graph

## Inputs

- Interleaved FASTQ file of quality-controlled NGS short reads (e.g. all_reads.fq, with mate pairs @read/1 and @read/2 adjacent)
- Minimap2 all-vs-all overlap alignments in PAF format with CIGAR strings (cg tag) for each read pair

## Outputs

- overlap_scores.tsv: well-formed TSV with one row per surviving read pair, columns for the two read identifiers, overlap length, identity, and overlap score D

## How to apply

Load the interleaved FASTQ read sample (pairs share a header like @S0R0/1 and @S0R0/2) and run Minimap2 in all-vs-all overlap mode so every read is aligned against every other read, retaining the CIGAR string for each alignment. Parse each CIGAR to extract the alignment identity (i) and the overlap length (o) between the two reads. Score each overlap with the StrainXpress formula D = 0.9*i + 0.1*o/((r1+r2)/2), where r1 and r2 are the lengths of the two overlapping reads; this weighted combination (equation 1) avoids assessing overlap quality across the full overlap length. Filter out any overlap shorter than 30 bp or with identity below 0.9. Write the surviving read pairs to a TSV (overlap_scores.tsv) with columns for the two read identifiers, overlap length, identity, and overlap score D.

## Related tools

- **Minimap2** (Performs the all-vs-all suffix-prefix overlap alignment of reads, outputting CIGAR strings from which identity and overlap length are parsed) — https://github.com/lh3/minimap2
- **StrainXpress** (Defines the overlap scoring formula (equation 1) and the 30 bp / 0.9 identity filtering thresholds applied to read pairs) — https://github.com/HaploKit/StrainXpress
- **Pandas** (Handles the tabular overlap_scores.tsv output during the scoring workflow)

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- overlap_scores.tsv exists and is well-formed TSV in which every row contains a read pair, an overlap length, an identity, and a score D
- Every surviving row has overlap length >= 30 bp and identity >= 0.9
- Every overlap score D lies in the range [0, 1] and matches recomputation via D = 0.9*i + 0.1*o/((r1+r2)/2) from the parsed CIGAR identity and overlap length
- The input FASTQ is interleaved: mate pairs (e.g. @S0R0/1 and @S0R0/2) appear adjacently in a single file
- Every output read pair corresponds to a Minimap2 alignment that carried a CIGAR string

## Limitations

- The input file must be a single interleaved FASTQ; separate paired-end files are not accepted
- The weighted score is computed directly from the CIGAR string, avoiding full-length overlap quality assessment, so it is an approximation of per-base overlap quality
- Scoring is designed for short (Illumina-type) reads; the short read length constrains downstream contig lengths and cannot produce strain-specific contigs spanning whole genomes

## Evidence

- [methods] StrainXpress computes an overlap score for each Minimap2-identified suffix-prefix read overlap directly from the CIGAR string output by Minimap2: [evidence span withheld — anchor status: not_found]
- [readme] The input file must be interleaved FASTQ and format like below:: "The input file must be interleaved FASTQ and format like below:"
- [readme] You can ask minimap2 to generate CIGAR at the `cg` tag of PAF with:: [evidence span withheld — anchor status: external_doc]
- [methods] Score each overlap using the StrainXpress formula D = 0.9*i + 0.1*o/((r1+r2)/2), where r1 and r2 are the lengths of the two overlapping reads: [evidence span withheld — anchor status: not_found]
