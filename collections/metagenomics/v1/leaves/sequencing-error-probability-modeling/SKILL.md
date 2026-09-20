---
name: sequencing-error-probability-modeling
description: Use when assembling overlapping Illumina paired-end reads (e.g. 16S rRNA
  amplicons in FASTQ/SAM) where naive perfect-match overlap discards ~40% of reads
  due to uncalled or miscalled bases.
when_to_use_negative:
- Reads are non-overlapping paired-end (no shared amplicon region), so no overlap
  likelihood can be computed
- Input is already a merged contig or a feature/OTU table — assembly modeling no longer
  applies
- Sequencing technology has a high probability of gap-inclusion, since PANDAseq exploits
  Illumina-specific properties including the low probability of gap-inclusion
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PANDAseq
  role: Implements the probabilistic overlap scoring and error-correcting assembly
    of paired-end reads
  repo: https://github.com/neufeld/pandaseq
- name: pandaseq-sam
  role: Extends PANDAseq to read SAM/BAM formatted paired-end input
  repo: https://github.com/neufeld/pandaseq-sam
- name: Illumina
  role: Sequencing platform producing the paired-end reads whose quality values feed
    the error model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:881e9cc517a8bf95cf8673acdd630d4987ba3965597a16ef2a7b727ced673ebe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  - pandaseq-sam
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sequencing-error-probability-modeling

## Summary

Probabilistic error modeling of paired-end Illumina reads that converts ASCII quality values into base-error probabilities and computes the likelihood of each candidate overlap, so the overlap maximizing Pr[F,R|c] corrects uncalled (N) and miscalled bases during assembly (PANDAseq approach).

## When to use

Use when assembling overlapping Illumina paired-end reads (e.g. 16S rRNA amplicons in FASTQ/SAM) where naive perfect-match overlap discards ~40% of reads due to uncalled or miscalled bases.

## When NOT to use

- Reads are non-overlapping paired-end (no shared amplicon region), so no overlap likelihood can be computed
- Input is already a merged contig or a feature/OTU table — assembly modeling no longer applies
- Sequencing technology has a high probability of gap-inclusion, since PANDAseq exploits Illumina-specific properties including the low probability of gap-inclusion

## Inputs

- Paired-end Illumina FASTQ reads (or SAM/BAM via pandaseq-sam) with CASAVA-encoded ASCII quality values
- Optional embedded amplification primer sequences to locate before overlap scoring
- Quality-score threshold and length constraints for output validation

## Outputs

- Assembled (joined) overlapping sequence per read pair with corrected bases in the overlap region
- Per-pair overlap probability score and overall assembly quality score
- Discarded read pairs for which no suitable overlap was found

## How to apply

Convert CASAVA ASCII quality values (A0 pre-1.8, A1 for CASAVA 1.8) to error probabilities ε = 10^(-A/10). For each candidate overlap c in [1, min(|F|,|R|)), compute Pr[F,R|c] as the product of per-base probabilities: matches score (1-εX)(1-εY) + εXεY/3, mismatches score (1/3)(1-εX)εY + (1/3)(1-εY)εX + (2/9)εXεY, and uncalled N bases score 1/4. Select the overlap c that maximizes Pr[F,R|c]; discard the read pair if no suitable overlap is found. Reconstruct the joined sequence keeping the higher-quality base when bases disagree, treating N as always matching, then compute an overall assembly quality score and validate against user-specified criteria (e.g. quality >= 0.9, as used for the M. capsulatus benchmark from SRA024100).

## Related tools

- **PANDAseq** (Implements the probabilistic overlap scoring and error-correcting assembly of paired-end reads) — https://github.com/neufeld/pandaseq
- **pandaseq-sam** (Extends PANDAseq to read SAM/BAM formatted paired-end input) — https://github.com/neufeld/pandaseq-sam
- **Illumina** (Sequencing platform producing the paired-end reads whose quality values feed the error model)

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- Selected overlap reproduces the expected joined amplicon sequence for validation read pairs
- Assembled sequences meet the paper's assembly quality criterion (quality score >= 0.9, as achieved for all 1 350 602 simulated and 598 775/673 845 M. capsulatus reads)
- Errors introduced by assembly remain in the 0.02-0.08% range, below the 5-8% of sequences with errors corrected
- Read pairs with no overlap exceeding the threshold are discarded rather than force-merged
- At a lowered 0.6 quality threshold, sequence recovery increases (~9% more, 652 249 sequences) without collapsing to trivial output

## Limitations

- Uncertain error corrections come from reads with many low-quality bases
- Approximately 40% of sequences can be discarded when uncalled or miscalled bases preclude a valid overlap
- PANDAseq relies on Illumina-specific properties, notably the low probability of gap-inclusion, so it is not suited to gap-prone chemistries

## Evidence

- [methods] To score alignments, we calculate the probability that the true nucleotides, and, are the same, given the observed nucleotides, X and Y.: [evidence span withheld — anchor status: not_found]
- [intro] determines the proper amount of overlap and reconstructs the entire sequence by correcting errors in the overlapping region: "determines the proper amount of overlap and reconstructs the entire sequence by correcting errors in the overlapping region"
- [methods] If no suitable overlap is found, then the read pair is discarded.: "If no suitable overlap is found, then the read pair is discarded."
- [results] Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9.: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
- [methods] The constructed sequence can then be validated against user-specified criteria.: "The constructed sequence can then be validated against user-specified criteria."
