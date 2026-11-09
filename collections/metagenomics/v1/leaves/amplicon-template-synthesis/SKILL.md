---
name: amplicon-template-synthesis
description: Use when you need a simulated paired-end Illumina benchmark with known
  ground-truth sequences but realistic, experimentally derived per-base error probabilities
  for validating an assembler.
when_to_use_negative:
- You need real sequencing data with biological variation — this skill produces a
  single-template synthetic benchmark, not experimental reads
- You lack access to real Illumina quality-score profiles, since substituting synthetic
  quality values would invalidate the error-mask fidelity this method depends on
- The assembler under test does not consume standard FASTQ paired-end input (forward/reverse
  files)
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0632
tools:
- name: PANDAseq
  role: Validates the simulated benchmark by assembling all synthesized pairs at the
    0.9 quality threshold
  repo: https://github.com/neufeld/pandaseq
- name: Illumina
  role: Source of the real sequencing run whose quality-score profiles serve as the
    error masks
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:c2cbe6133b4fbbaa30371de8a9bae0ea02d76b2875e8491be598e05d30a795d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# amplicon-template-synthesis

## Summary

Constructs a simulated benchmark of error-free paired-end FASTQ reads by splicing a known amplicon template (e.g., the 135-base Sinorhizobium meliloti 16S V3 region amplified by 341f/518r) onto real Illumina quality-score profiles from a published run (NCBI SRA024100), enabling error-free ground-truth testing of paired-end assemblers like PANDAseq.

## When to use

Use when you need a simulated paired-end Illumina benchmark with known ground-truth sequences but realistic, experimentally derived per-base error probabilities for validating an assembler.

## When NOT to use

- You need real sequencing data with biological variation — this skill produces a single-template synthetic benchmark, not experimental reads
- You lack access to real Illumina quality-score profiles, since substituting synthetic quality values would invalidate the error-mask fidelity this method depends on
- The assembler under test does not consume standard FASTQ paired-end input (forward/reverse files)

## Inputs

- Published Illumina quality-score profiles / run (NCBI SRA accession SRA024100)
- Reference 16S V3 amplicon template sequence (e.g., 135-base S. meliloti V3 region, primers 341f/518r excluded)
- Target read length (108 bases) and number of read pairs to synthesize (e.g., 1,350,602)

## Outputs

- forward.fastq (error-free template sequences with borrowed Illumina quality strings, PHRED-encoded)
- reverse.fastq (paired by template position, same format)
- Ground-truth mapping of each simulated read pair to its source amplicon template for downstream error-rate evaluation

## How to apply

Obtain real Illumina quality-score profiles from the published run (NCBI SRA accession SRA024100) and reference 16S V3 amplicon template sequences. Replace the original run's sequence with the corresponding V3 amplicon template (e.g., the 135-base S. meliloti V3 region amplified by 341f and 518r, excluding primers) truncated to the original read length (108 bases), pairing reads by template position until the target number of pairs is generated (the paper synthesized 1,350,602 pairs). For each base, convert the borrowed ASCII quality value A to an error probability via ε = 10^(-A/10), respecting the CASAVA convention that end-of-read scores may be masked to the lowest value. Apply the error masks by stochastically miscalling bases with probability ε (errors equiprobable over the other three nucleotides) or emitting N with the lowest quality score, and write per-base quality strings to forward.fastq and reverse.fastq in standard PHRED-encoded FASTQ. Validate by assembling with PANDAseq (pandaseq -f forward.fastq -r reverse.fastq) at the 0.9 quality-score threshold and confirming all pairs assemble successfully.

## Related tools

- **PANDAseq** (Validates the simulated benchmark by assembling all synthesized pairs at the 0.9 quality threshold) — https://github.com/neufeld/pandaseq
- **Illumina** (Source of the real sequencing run whose quality-score profiles serve as the error masks)

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq -t 0.9  # validate all 1,350,602 synthesized pairs assemble >0.9
```

## Evaluation signals

- All 1,350,602 synthesized paired-end sequences assemble with PANDAseq quality scores greater than 0.9 (pandaseq -f forward.fastq -r reverse.fastq)
- Output files are valid FASTQ with per-base PHRED-encoded quality lines matching the borrowed Illumina profiles
- Per-base error probability derived as ε = 10^(-A/10) matches the original run's quality distribution, including CASAVA end-of-read quality masking to the lowest score
- Uncalled bases (N) are emitted only at positions with the lowest quality score
- Each simulated pair traces back to exactly one V3 amplicon template, enabling exact expected-sequence comparison after assembly

## Limitations

- Errors are applied as equiprobable substitutions over the three alternative nucleotides plus N emission — no indels, which is justified by Illumina's low probability of gap-inclusion
- Simulated reads come from a single amplicon template, so the benchmark cannot test assembler behavior on mixed-template or overlapping-homopolymer complexity
- The ϵ model itself was calibrated by counting mismatch rates in known index tags of a defined community data set; masks borrowed from SRA024100 may not generalize to other chemistries or read lengths

## Evidence

- [results] Simulated data were produced using real error masks derived from a previously published run of mixed pure bacterial cultures (NCBI SRA accession SRA024100): "made from mixed pure bacterial cultures (NCBI SRA accession SRA024100)"
- [results] All synthesized pairs assemble above the 0.9 quality threshold, confirming benchmark correctness: "all 1 350 602 synthesized paired-end sequences assembled with quality scores greater than 0.9 (Figure 2)"
- [intro] PANDAseq exploits Illumina-specific error properties that the simulation preserves: "PANDAseq makes use of Illumina-specific properties, including the low probability of gap-inclusion."
- [methods] Quality scores drive per-base error probabilities in the simulation and assembly scoring: [evidence span withheld — anchor status: not_found]
- [readme] PANDAseq CLI used for validation of the synthesized FASTQ pairs: "pandaseq -f forward.fastq -r reverse.fastq"
