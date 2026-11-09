---
name: mutation-frequency-mixture-design
description: Use when benchmarking a hotspot mutation caller on nanopore data and
  you need simulated FASTQ mixtures with known truth sets across mutant frequencies,
  replicates, and negative controls.
when_to_use_negative:
- Input is real experimental nanopore data without a known truth set — this skill
  designs simulation mixtures with engineered mutant reads, not analysis of real samples
- You need whole-genome or non-hotspot variant calling on long reads — the mixtures
  are restricted to a small BED-defined hotspot panel, not genome-wide discovery
- Training data for deep-learning callers (e.g. iGDA, PEPPER-Margin-DeepVariant) is
  required — the article notes simulated ONT data were 'not generated on the same
  experimental and sequencing platforms' and their own experimental data were insufficient
  as a training dataset
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Nano2NGS-Muta
  role: Target detection framework benchmarked on the simulated hotspot mixtures;
    invoked with FASTQ input, NGS-liked read length 101 bp, position_step 10, and
    hotspot BED file
  repo: https://github.com/langjidong/Nano2NGS
- name: Oxford Nanopore Technologies nanopore sequencer
  role: Reference platform whose read-length and base-quality characteristics (1020–3968
    bp, Q10–Q42) are mimicked when simulating ONT-like reads
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac033
    title: 'Nano2NGS-Muta: a framework for converting nanopore sequencing data to
      NGS-liked sequencing data for hotspot mutation detection'
  source_content_hash: sha256:40c445651d6b6ec87ca6ca83bc27e8471e6b08530eb39f02b45f5ec9a56cced6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/langjidong/Nano2NGS
  repo_url: https://github.com/langjidong/Nano2NGS
  tools:
  - Nano2NGS-Muta
  - Oxford Nanopore Technologies nanopore sequencer
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mutation-frequency-mixture-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design in silico nanopore read mixtures carrying hotspot mutations at graded mutant frequencies (10%, 5%, 1%, 0.5%, 0.1%) with wild-type/mutant read counts of 9000:1000 per hotspot, to benchmark hotspot mutation detection methods such as Nano2NGS-Muta.

## When to use

Use when benchmarking a hotspot mutation caller on nanopore data and you need simulated FASTQ mixtures with known truth sets across mutant frequencies, replicates, and negative controls.

## When NOT to use

- Input is real experimental nanopore data without a known truth set — this skill designs simulation mixtures with engineered mutant reads, not analysis of real samples
- You need whole-genome or non-hotspot variant calling on long reads — the mixtures are restricted to a small BED-defined hotspot panel, not genome-wide discovery
- Training data for deep-learning callers (e.g. iGDA, PEPPER-Margin-DeepVariant) is required — the article notes simulated ONT data were 'not generated on the same experimental and sequencing platforms' and their own experimental data were insufficient as a training dataset

## Inputs

- Hotspot panel configuration (BED-like: chr, start, end, gene symbol, mutation, optional transcription ID)
- Reference genome sequences (e.g. NCBI Genome strain reference sequences)
- Design parameters: read length distribution (mean 2467 bp, sd 783 bp, range 1020–3968 bp), base quality range Q10–Q42, per-hotspot read counts (9000 WT / 1000 mutant)

## Outputs

- Per-replicate FASTQ files mixing wild-type and mutant ONT-like reads at each target mutant frequency
- Negative-control FASTQ replicates containing only 9000 wild-type reads per hotspot
- Manifest listing hotspot, mutation type, frequency, replicate, and read counts (truth set for evaluation)

## How to apply

Define a hotspot panel in BED-like format (chr, start, end, gene symbol, mutation) covering targets such as BRAF-p.V600E, EGFR-p.L858R, EGFR-p.T790M, KRAS-p.G12C, and EGFR-p.E746A750del. For each hotspot, simulate ONT-like wild-type reads with lengths drawn from a normal distribution (mean 2467 bp, sd 783 bp, range 1020–3968 bp) and base qualities spanning Q10–Q42, generating 9000 wild-type reads per hotspot. Engineer the SNV or indel into 1000 reads while keeping 9000 wild-type, then mix at theoretical mutant frequencies of 10%, 5%, 1%, 0.5%, and 0.1% with three replicates per frequency per hotspot, plus three negative-control replicates containing only 9000 wild-type reads. Write all mixtures as FASTQ and compile a manifest recording hotspot, mutation type, frequency, replicate, and read counts. The rationale is that only a simulation with known truth frequencies can measure false positive/negative rates at each mutant frequency, which the paper's discussion notes depends on the mutation-frequency threshold used for detection.

## Related tools

- **Nano2NGS-Muta** (Target detection framework benchmarked on the simulated hotspot mixtures; invoked with FASTQ input, NGS-liked read length 101 bp, position_step 10, and hotspot BED file) — https://github.com/langjidong/Nano2NGS
- **Oxford Nanopore Technologies nanopore sequencer** (Reference platform whose read-length and base-quality characteristics (1020–3968 bp, Q10–Q42) are mimicked when simulating ONT-like reads)

## Examples

```
perl nano2ngs-muta.pl -fq simulated_mix_rep1.fq.gz -read_len 101 -position_step 10 -configure Hotspot.bed -process 8
```

## Evaluation signals

- Each replicate FASTQ contains the designed read counts: 9000 wild-type reads plus the appropriate mutant fraction (e.g. 1000 mutant reads at 10%)
- Simulated read length distribution falls within the specified range (1020–3968 bp, mean ≈ 2467 bp, sd ≈ 783 bp)
- Simulated base qualities fall within Q10–Q42
- Negative-control replicates contain only wild-type reads (no engineered mutations) and should yield zero true positives
- Manifest read counts per hotspot, frequency, and replicate reconcile exactly with the FASTQ contents

## Limitations

- Simulated ONT/ONT-like data were not generated on the same experimental and sequencing platforms, so performance on real data may differ
- The limit of detection (LoD) of the downstream method is not established; the false positive/negative balance at a given mutation-frequency threshold requires more samples
- This design covers a 12-hotspot SNV/indel panel only; copy number variation, structural variation, gene fusion, and expression detection are out of scope (planned future work)

## Evidence

- [results] For each hotspot, 9000 wild-type reads and 1000 mutant reads were randomly generated as simulated nanopore reads: [evidence span withheld — anchor status: not_found]
- [readme] hotspot BED format for Nano2NGS-Muta: [evidence span withheld — anchor status: not_found]
- [discussion] simulated data platform mismatch: "Although we simulated ONT or ONT-like data, they were not generated on the same experimental and sequencing platforms"
- [discussion] LoD threshold consideration: "Another is the LoD to this method, that is, when the threshold of mutation frequency is detected, the false positive rate and false negative rate could be better controlled"
