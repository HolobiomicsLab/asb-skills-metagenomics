---
name: quality-profile-error-masking
description: Use when you need a controlled paired-end benchmark with known truth
  sequences but realistic, position-dependent Illumina error structure (e.g. to validate
  an assembler like PANDAseq).
when_to_use_negative:
- Input is already real sequenced paired-end FASTQ data — no truth sequence exists
  to mask, so use quality filtering/assembly instead
- You need error models with base-context-dependent substitution rates; this skill
  assumes errors are equiprobable over the other three nucleotides at a per-position
  ε
- The amplicon template is longer than the source run's read length, since simulated
  reads are truncated to the original read lengths
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0080
tools:
- name: PANDAseq
  role: Downstream assembler used to validate the simulated paired-end benchmark (all
    1,350,602 pairs should assemble at quality-score threshold 0.9)
  repo: https://github.com/neufeld/pandaseq
- name: Illumina
  role: Source platform whose real quality-score profiles (SRA024100) supply the error
    masks
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:aabc022c5c00195f9ac3b36f845e89a5d87e3140ab495dbde828bdb96aaba5a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# quality-profile-error-masking

## Summary

Builds simulated error-free paired-end FASTQ reads that carry real Illumina quality-score information by transplanting an amplicon template (e.g. the 135-base Sinorhizobium meliloti 16S V3 region) into a published Illumina run's quality profiles, then stochastically masking bases using per-position error probabilities derived from the borrowed PHRED scores.

## When to use

Use when you need a controlled paired-end benchmark with known truth sequences but realistic, position-dependent Illumina error structure (e.g. to validate an assembler like PANDAseq).

## When NOT to use

- Input is already real sequenced paired-end FASTQ data — no truth sequence exists to mask, so use quality filtering/assembly instead
- You need error models with base-context-dependent substitution rates; this skill assumes errors are equiprobable over the other three nucleotides at a per-position ε
- The amplicon template is longer than the source run's read length, since simulated reads are truncated to the original read lengths

## Inputs

- Reference 16S V3 amplicon template sequence (135 bases, primers 341f/518r region excluded)
- Real Illumina quality-score profiles from a published run (NCBI SRA accession SRA024100)
- Target read length (108 bases) and target number of paired-end reads (e.g. 1,350,602 pairs)

## Outputs

- forward.fastq (simulated reads with borrowed PHRED quality strings)
- reverse.fastq (simulated reads with borrowed PHRED quality strings)
- Error masks (per-position error probabilities ε = 10^(-A/10)) applied to the error-free templates

## How to apply

Obtain the reference 16S V3 amplicon template (135 bases, primers 341f/518r excluded) and the real Illumina quality-score profiles from a published run (NCBI SRA accession SRA024100). Replace the original run's sequences with the template, truncated to the original read length (108 bases), so error-free reads retain the experimental quality scores; pair reads by template position until the target count (1,350,602 pairs in the paper's benchmark). For each base, convert the borrowed ASCII quality value A to an error probability via ε = 10^(-A/10), respecting the CASAVA convention that end-of-read quality scores may be masked to the lowest score. Stochastically miscall each base with probability ε (errors equiprobable over the other three nucleotides) or emit N with the lowest quality score, writing per-base quality strings from the profiles into forward.fastq and reverse.fastq in standard FASTQ (PHRED-encoded) format. Rationale: keeping sequences error-free while borrowing real quality masks preserves ground truth for validating assembly error correction while reproducing position-dependent Illumina error structure.

## Related tools

- **PANDAseq** (Downstream assembler used to validate the simulated paired-end benchmark (all 1,350,602 pairs should assemble at quality-score threshold 0.9)) — https://github.com/neufeld/pandaseq
- **Illumina** (Source platform whose real quality-score profiles (SRA024100) supply the error masks)

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- All simulated pairs (1,350,602 in the paper) assemble successfully with PANDAseq at quality-score threshold 0.9 (pandaseq -f forward.fastq -r reverse.fastq)
- Per-base error probability equals 10^(-A/10) for the borrowed ASCII quality value A, with end-of-read positions respecting the CASAVA lowest-score masking convention
- Simulated sequences are error-free before masking (template sequence truncated to original 108-base read length)
- Output FASTQs carry PHRED-encoded per-base quality strings copied from the source quality profiles
- Uncalled bases (N), when emitted, carry the lowest quality score

## Limitations

- Errors are equiprobable over the other three nucleotides — real Illumina errors may be biased toward specific substitutions
- CASAVA end-of-read quality masking to the lowest score means the error mask is conservative at read ends
- The ε used in the paper was itself calibrated from a defined community data set; the simulated benchmark inherits that calibration rather than a new one

## Evidence

- [results] The simulated dataset was built from a previously published Illumina sequencing run of V3 hypervariable regions: "made from mixed pure bacterial cultures (NCBI SRA accession SRA024100)"
- [results] All simulated pairs assemble at threshold 0.9: "all 1 350 602 synthesized paired-end sequences assembled with quality scores greater than 0.9 (Figure 2)"
- [readme] Basic PANDAseq invocation on the simulated FASTQ pair: "The short version is:

	pandaseq -f forward.fastq -r reverse.fastq"
- [methods] The ϵ error rate used was calibrated by counting mismatches in index tags: "This value of ϵ was calculated by counting the mismatch rate in known index tags in a defined community data set (described below)."
- [results] Benchmark data source for M. capsulatus reads: "Sequencing produced 673 845 paired-end 108-base reads, available at http://neufeldserver.uwaterloo.ca/~apmasell/pandaseq_sampledata.tar."
