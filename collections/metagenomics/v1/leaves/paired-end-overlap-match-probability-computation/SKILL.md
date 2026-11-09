---
name: paired-end-overlap-match-probability-computation
description: Use when scoring overlapping paired-end FASTQ reads (e.g. 108-base Illumina
  reads) to decide which assembled sequences pass a quality threshold such as p =
  0.9 or p = 0.6.
when_to_use_negative:
- Read pairs have no suitable overlap at all — the pair should simply be discarded
  rather than scored ('If no suitable overlap is found, then the read pair is discarded.')
- Input is SAM/BAM formatted and the goal is assembly with alignment output rather
  than overlap probability scoring — pandaseq-sam extends PANDAseq for that purpose
- Sequences are already assembled and a feature/OTU table exists — scoring overlap
  match probabilities is only relevant before or during paired-end assembly
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PANDAseq
  role: Implements the overlap match-probability scoring, overall quality score computation,
    and threshold-based rejection of assembled paired-end reads
  repo: https://github.com/neufeld/pandaseq
- name: Illumina
  role: Sequencing platform whose CASAVA quality encoding and low gap-inclusion probability
    supply the ε error probabilities used in the match probability formulas
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:f354b550623817238fe815dc1eafc23189292fc789896b809bce4dc4b4fcd596
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# paired-end-overlap-match-probability-computation

## Summary

Computes, for each position in an Illumina paired-end read overlap, the probability that the two observed nucleotides are derived from the same true base, using per-base CASAVA PHRED error probabilities and quality-mask handling; these probabilities are geometrically averaged into the overall assembly quality score used for threshold acceptance/discard decisions.

## When to use

Use when scoring overlapping paired-end FASTQ reads (e.g. 108-base Illumina reads) to decide which assembled sequences pass a quality threshold such as p = 0.9 or p = 0.6.

## When NOT to use

- Read pairs have no suitable overlap at all — the pair should simply be discarded rather than scored ('If no suitable overlap is found, then the read pair is discarded.')
- Input is SAM/BAM formatted and the goal is assembly with alignment output rather than overlap probability scoring — pandaseq-sam extends PANDAseq for that purpose
- Sequences are already assembled and a feature/OTU table exists — scoring overlap match probabilities is only relevant before or during paired-end assembly

## Inputs

- Illumina paired-end FASTQ read pairs with CASAVA-encoded PHRED quality strings
- Quality-mask region boundaries (CASAVA-masked tail regions per read)
- Quality threshold value(s), e.g. p = 0.9 and p = 0.6

## Outputs

- Per-pair overall assembly quality score in (0, 1]
- Two-column table (sequence_id, quality_score) with acceptance flags at p = 0.9 and p = 0.6 (quality_scores.tsv)

## How to apply

For each base in the overlap region, convert the CASAVA-encoded PHRED quality character to an error probability ε = 10^(-A/10), where A is the ASCII quality value. When the observed bases agree, compute Pr[X̂=Ŷ|X=Y] = (1-εX)(1-εY) + εXεY/3; when they disagree, compute Pr[X̂=Ŷ|X≠Y] = (1/3)(1-εX)εY + (1/3)(1-εY)εX + (2/9)εXεY; uncalled bases use Pr[X̂=Ŷ|Y=N] = 1/4. Apply the quality-mask handling: if one base lies in the CASAVA-masked tail region, use the other base's probability on a match and 1/4 on a mismatch, and 1/4 when both are masked — this is why mismatches in doubly-masked regions of both reads drive the score down and cause the assembly to be discarded. Combine per-base scores over the full assembled sequence via the geometric mean (compensating for variable lengths) to obtain the overall assembly quality score, and flag the sequence as accepted when the score meets the threshold (e.g. 0.9 as in the paper's 598 775/673 845 accepted M. capsulatus reads, or 0.6 to recover 9% more sequences).

## Related tools

- **PANDAseq** (Implements the overlap match-probability scoring, overall quality score computation, and threshold-based rejection of assembled paired-end reads) — https://github.com/neufeld/pandaseq
- **Illumina** (Sequencing platform whose CASAVA quality encoding and low gap-inclusion probability supply the ε error probabilities used in the match probability formulas)

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- quality_scores.tsv exists with one row per input pair and fields sequence_id, quality_score, accept_0.9, accept_0.6
- Every quality_score value lies in (0, 1]
- accept_0.9 and accept_0.6 flags exactly match the threshold comparisons score >= 0.9 and score >= 0.6
- Sanity check against the paper: all 1 350 602 simulated paired-end sequences scored above 0.9, and 598 775 of 673 845 M. capsulatus reads passed the 0.9 threshold (598,775/673,845 ≈ 88.9%)
- Assemblies whose mismatches fall in doubly quality-masked regions should yield low scores and be flagged for discard

## Limitations

- Mismatches in the quality-masked regions of both reads produce two low-quality, arbitrary base calls per mismatch, driving the overall score down so otherwise usable assemblies are discarded below the acceptance threshold
- Uncertain error corrections come from reads with many low-quality bases identified by upstream processing
- The epsilon error rate itself was calibrated empirically by counting mismatch rates in index tags of a defined community data set, so scores depend on that calibration
- PANDAseq can also introduce errors: about 0.02-0.08% of output reads contained errors introduced by the assembly process

## Evidence

- [methods] To score alignments, we calculate the probability that the true nucleotides, and, are the same, given the observed nucleotides, X and Y.: [evidence span withheld — anchor status: not_found]
- [methods] Overall score and construction: "Once the overlap is selected, the output sequence is constructed and an overall quality score is calculated."
- [discussion] Low-score rejection rationale: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [results] Simulated sequences all above 0.9: "all 1 350 602 synthesized paired-end sequences assembled with quality scores greater than 0.9"
- [readme] PANDAseq purpose from README: "PANDASEQ is a program to align Illumina reads, optionally with PCR primers embedded in the sequence, and reconstruct an overlapping sequence."
