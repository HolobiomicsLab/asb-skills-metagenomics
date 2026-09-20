---
name: paired-end-overlap-scoring
description: Use when merging overlapping paired-end Illumina FASTQ reads (e.g. 16S
  rRNA amplicons) where naive perfect-match overlap fails due to uncalled (N) or miscalled
  bases, and error-correcting probabilistic assembly is needed.
when_to_use_negative:
- Reads are not expected to overlap (e.g. long-insert libraries without adapter/overlap)
  — SeqPrep's README notes the overlapping procedure should be saved for libraries
  where a significant portion of reads overlap
- Sequencing platform with non-negligible gap probability — PANDAseq relies on Illumina-specific
  properties including the low probability of gap-inclusion
- Input is already assembled contigs or a feature/OTU table rather than raw paired-end
  reads
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: PANDAseq
  role: Primary probabilistic overlap-scoring paired-end assembler that identifies
    the optimal overlap and reconstructs the complete sequence with error correction
  repo: https://github.com/neufeld/pandaseq
- name: PANDAseq-sam
  role: PANDAseq extension that applies the same overlap scoring to SAM/BAM formatted
    input
  repo: https://github.com/neufeld/pandaseq-sam
- name: SeqPrep
  role: Alternative merger using alignment-based heuristic overlap with mismatch/match
    fraction cutoffs, contrasted with PANDAseq's probabilistic scoring
  repo: https://github.com/jstjohn/SeqPrep/
- name: stitch
  role: Alternative paired-end overlap assembler (no longer maintained; README directs
    users to PANDAseq)
  repo: https://github.com/audy/stitch
- name: Illumina
  role: Sequencing platform whose paired-end chemistry and quality encodings the probabilistic
    model is designed for
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:c09f99aca2d3bcced7f44a6623e2d9c891bcf48ee9d42192c415856f326d81e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  - PANDAseq-sam
  - SeqPrep
  - stitch
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# paired-end-overlap-scoring

## Summary

Probabilistically scores every candidate overlap between forward and reverse Illumina paired-end reads to select the overlap amount that maximizes Pr[F,R|c], then reconstructs the joined sequence while correcting uncalled and miscalled bases in the overlap region, as implemented in PANDAseq.

## When to use

Use when merging overlapping paired-end Illumina FASTQ reads (e.g. 16S rRNA amplicons) where naive perfect-match overlap fails due to uncalled (N) or miscalled bases, and error-correcting probabilistic assembly is needed.

## When NOT to use

- Reads are not expected to overlap (e.g. long-insert libraries without adapter/overlap) — SeqPrep's README notes the overlapping procedure should be saved for libraries where a significant portion of reads overlap
- Sequencing platform with non-negligible gap probability — PANDAseq relies on Illumina-specific properties including the low probability of gap-inclusion
- Input is already assembled contigs or a feature/OTU table rather than raw paired-end reads

## Inputs

- Forward and reverse paired-end Illumina reads in FASTQ (or qseq/SAM/BAM via pandaseq-sam) with CASAVA ASCII quality values (pre-1.8 'A0' or CASAVA 1.8 'A1' encoding)
- Optional PCR primer sequences embedded in reads (PANDAseq first determines the locations of the amplification primers if specified)
- Quality score threshold (e.g. 0.9), minimum/maximum length constraints, and uncalled-base handling settings

## Outputs

- Reconstructed (assembled) joined sequences for read pairs with an acceptable overlap
- Per-pair assembly quality scores used for filtering (thresholded, e.g. >= 0.9)
- Discarded read pairs where no suitable overlap was found

## How to apply

Retrieve paired-end reads (e.g. from NCBI SRA accession SRA024100) in FASTQ format and convert CASAVA ASCII quality values to base-error probabilities ε = 10^(-A/10). For each candidate overlap c in [1, min(|F|,|R|)), compute Pr[F,R|c] as the product of base-match probabilities: for observed bases X and Y, Pr[X=Y'|X=Y] = (1-εX)(1-εY) + εXεY/3 for matches, the mismatch formula (1/3)(1-εX)εY + (1/3)(1-εY)εX + (2/9)εXεY, and 1/4 when a base is an uncalled N. Select the overlap c that maximizes Pr[F,R|c]; if no suitable overlap is found, discard the read pair. Reconstruct the joined sequence for the selected overlap, keeping the higher-quality base when bases disagree and treating N bases as always matching, then compute the overall assembly quality score and validate against user-specified criteria such as a quality threshold (0.9 in the paper's benchmarks; 0.6 recovers ~9% more sequences). The rationale is that PANDAseq exploits Illumina-specific properties, including the low probability of gap-inclusion, so only overlap length needs to be searched rather than full alignments.

## Related tools

- **PANDAseq** (Primary probabilistic overlap-scoring paired-end assembler that identifies the optimal overlap and reconstructs the complete sequence with error correction) — https://github.com/neufeld/pandaseq
- **PANDAseq-sam** (PANDAseq extension that applies the same overlap scoring to SAM/BAM formatted input) — https://github.com/neufeld/pandaseq-sam
- **SeqPrep** (Alternative merger using alignment-based heuristic overlap with mismatch/match fraction cutoffs, contrasted with PANDAseq's probabilistic scoring) — https://github.com/jstjohn/SeqPrep/
- **stitch** (Alternative paired-end overlap assembler (no longer maintained; README directs users to PANDAseq)) — https://github.com/audy/stitch
- **Illumina** (Sequencing platform whose paired-end chemistry and quality encodings the probabilistic model is designed for)

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq -T 0.9
```

## Evaluation signals

- Selected overlap reproduces the expected joined/amplicon sequence for validation read pairs
- Assembled sequences meet the paper's quality criterion: assembly quality score >= 0.9 (as reported for all 1,350,602 simulated pairs and 598,775 of 673,845 M. capsulatus reads)
- Errors introduced by assembly remain rare: ~0.02-0.08% of output reads, versus 5-8% of sequences with corrected errors
- Read pairs with no overlap exceeding the threshold are discarded rather than force-merged
- Match/mismatch probability calculations handle uncalled bases as Pr = 1/4 (N always matches)

## Limitations

- Uncertain error corrections come from reads with many low-quality bases identified by upstream processing
- Incorrect assemblies with mismatches in quality-masked regions of both reads have low quality scores and are discarded, so heavily degraded overlaps are unrecoverable
- No software version number or changelog is provided in the article for reproducibility
- Naive perfect-match overlap assembly discards ~40% of sequences due to uncalled or miscalled bases, motivating but also bounding the improvement over simpler approaches

## Evidence

- [intro] PANDAseq uses paired-end Illumina reads to determine the proper amount of overlap and reconstructs the entire sequence by correcting errors in the overlapping region.: "determines the proper amount of overlap and reconstructs the entire sequence by correcting errors in the overlapping region"
- [methods] Scoring is based on the probability that true nucleotides match given observed bases and quality-derived error probabilities.: [evidence span withheld — anchor status: not_found]
- [results] Quality threshold benchmark for assembled sequences.: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
- [methods] Read pairs with no acceptable overlap are discarded rather than merged.: "If no suitable overlap is found, then the read pair is discarded."
- [intro] PANDAseq exploits Illumina-specific properties such as low gap probability, so only the overlap amount must be searched.: "PANDAseq makes use of Illumina-specific properties, including the low probability of gap-inclusion."
