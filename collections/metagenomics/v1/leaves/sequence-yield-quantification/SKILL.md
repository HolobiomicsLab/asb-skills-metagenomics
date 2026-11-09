---
name: sequence-yield-quantification
description: Use when benchmarking how many sequences an overlapping paired-end assembler
  recovers from FASTQ read pairs relative to a naive perfect-match overlap baseline,
  or when tuning an assembly quality-score threshold to raise recovery.
when_to_use_negative:
- The input is already an assembled or clustered sequence/abundance table rather than
  raw paired-end reads — yield comparison requires re-running the assembly itself.
- Read pairs do not contain overlapping regions (e.g. fragment length far exceeds
  read length), since the yield gain relies on reconstructing the overlap with error
  correction.
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: PANDAseq
  role: Probabilistic error-correcting assembler used to assemble the paired-end reads
    at quality thresholds 0.9 and 0.6; run as `pandaseq -f forward.fastq -r reverse.fastq`
  repo: https://github.com/neufeld/pandaseq
- name: baseline_naive
  role: Naive perfect-overlap assembly baseline that discards pairs whose sequenced
    bases do not match exactly in the overlap; provides the denominator for percent
    yield increase
- name: Illumina
  role: Sequencing platform that produced the paired-end reads of the composite control
    library
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:7818506c503f863c4a1919d428993815d4d6ca53a6de041be251b90b20282fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  - baseline_naive
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sequence-yield-quantification

## Summary

Quantify how many paired-end Illumina reads survive assembly under different assembler settings (e.g. PANDAseq quality thresholds of 0.9 vs 0.6) and compare recovery against a naive perfect-overlap baseline, expressed as relative percent yield increase.

## When to use

Use when benchmarking how many sequences an overlapping paired-end assembler recovers from FASTQ read pairs relative to a naive perfect-match overlap baseline, or when tuning an assembly quality-score threshold to raise recovery.

## When NOT to use

- The input is already an assembled or clustered sequence/abundance table rather than raw paired-end reads — yield comparison requires re-running the assembly itself.
- Read pairs do not contain overlapping regions (e.g. fragment length far exceeds read length), since the yield gain relies on reconstructing the overlap with error correction.

## Inputs

- Paired-end Illumina FASTQ read pairs (forward and reverse files) from a pooled composite control library, e.g. NCBI SRA accession SRA024100
- PANDAseq installed with zlib and libbz2 development packages (build-essential, libtool, automake, zlib1g-dev, libbz2-dev, pkg-config)

## Outputs

- Retained assembled-sequence counts for each condition (naive baseline, PANDAseq at threshold 0.9, PANDAseq at threshold 0.6)
- Relative percent yield increases versus the naive baseline (expected ~3.9% at 0.9 and ~50% overall at 0.6)
- A comparison table such as pooled_yield_comparison.csv with counts and computed percent increases

## How to apply

Download paired-end Illumina reads (e.g. NCBI SRA accession SRA024100, a composite control library built from mixed pure bacterial cultures) and assemble them with PANDAseq at an assembly quality score threshold of 0.9, counting retained sequences. Repeat the assembly with the threshold lowered to 0.6, since relaxed thresholds correct errors probabilistically in the overlap rather than discarding reads with uncalled or miscalled bases (which caused ~40% loss under naive assembly). Assemble the same read pairs with the naive perfect-overlap baseline, which discards any pair whose sequenced bases do not match exactly in the overlap. Compute relative yield as (PANDAseq sequences minus naive sequences) divided by naive sequences for each threshold, and write counts and percent increases to a CSV such as pooled_yield_comparison.csv. Expect roughly 3.9% increase at threshold 0.9 and about 50% overall (85% within the most abundant clusters) at threshold 0.6.

## Related tools

- **PANDAseq** (Probabilistic error-correcting assembler used to assemble the paired-end reads at quality thresholds 0.9 and 0.6; run as `pandaseq -f forward.fastq -r reverse.fastq`) — https://github.com/neufeld/pandaseq
- **baseline_naive** (Naive perfect-overlap assembly baseline that discards pairs whose sequenced bases do not match exactly in the overlap; provides the denominator for percent yield increase)
- **Illumina** (Sequencing platform that produced the paired-end reads of the composite control library)

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq -t 0.6 -w assembled.fasta; pandaseq -f forward.fastq -r reverse.fastq -t 0.9 -w assembled_t09.fasta
```

## Evaluation signals

- The relative yield increase at threshold 0.9 versus the naive baseline is approximately 3.9%.
- The relative yield increase at threshold 0.6 versus the naive baseline is roughly 50% overall, and about 85% within the most abundant clusters, consistent with the reported 4-50% range.
- Counts for all three assemblies (naive, PANDAseq 0.9, PANDAseq 0.6) are present in pooled_yield_comparison.csv and percent increases equal (PANDAseq − naive) / naive.
- Sanity-check counts against published figures: 598,775 of 673,845 M. capsulatus pairs assembled at threshold ≥ 0.9, rising to 652,249 (+9%) at threshold 0.6.
- PANDAseq-assembled outputs contain very few introduced errors (~0.02-0.08% of output reads), so higher yield should not come with visibly degraded sequence quality.

## Limitations

- Lowering the quality threshold to 0.6 increases recovery but permits more uncertain error corrections; uncertain corrections come from reads with many low-quality bases.
- Yield gains depend on the library: the control library showed 4-50% over naive assembly, whereas a single-template library showed only a 9% gain at threshold 0.6.
- If no suitable overlap is found for a read pair, the pair is discarded regardless of threshold; PANDAseq also rejects sequences with too-short/too-long lengths or uncalled bases when those filters are enabled.

## Evidence

- [abstract] Assembly of control libraries showed a 4-50% increase in the number of assembled sequences over naïve assembly: "Assembly of control libraries showed a 4-50% increase in the number of assembled sequences over naïve assembly"
- [results] We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences.: "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences."
- [results] Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9.: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
- [results] made from mixed pure bacterial cultures (NCBI SRA accession SRA024100): "made from mixed pure bacterial cultures (NCBI SRA accession SRA024100)"
- [readme] The short version is:

	pandaseq -f forward.fastq -r reverse.fastq: "The short version is:

	pandaseq -f forward.fastq -r reverse.fastq"
