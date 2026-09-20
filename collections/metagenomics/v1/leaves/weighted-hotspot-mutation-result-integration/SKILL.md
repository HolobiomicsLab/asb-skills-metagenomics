---
name: weighted-hotspot-mutation-result-integration
description: Use when Nano2NGS-Muta has produced per-set mutation detection results
  from N extracted short-read sets and final target hotspot mutation calls must be
  integrated and checked against a complete check-hotspot.list.
when_to_use_negative:
- Input is already conventional short-read NGS data with no nanopore long-read conversion
  step — no extraction sets exist to integrate.
- The goal is unbiased genome-wide variant discovery rather than detection of mutations
  in a predefined hotspot panel; Nano2NGS-Muta is designed for hotspot panel detection
  (e.g. clinical diagnosis).
- Reads need base-level error correction before calling — the framework only converts
  long reads into a set of short reads with no error correction for reads and/or bases.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3512
tools:
- name: Nano2NGS-Muta
  role: Extracts the N NGS-liked short-read sets from nanopore reads whose per-set
    results are integrated; invoked via nano2ngs-muta.pl with -fq, -read_len, -position_step,
    -configure, -process
  repo: https://github.com/langjidong/Nano2NGS
- name: GATK Best Practice pipeline
  role: NGS mutation-analysis pipeline applied to each extracted short-read set to
    produce the per-set hotspot mutation results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac033
    title: 'Nano2NGS-Muta: a framework for converting nanopore sequencing data to
      NGS-liked sequencing data for hotspot mutation detection'
  source_content_hash: sha256:f41104234662073ee5f2894389631e3a66492963dc89ea4a985553858d97757c
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
  - GATK Best Practice pipeline
  - Nano2NGS-Muta
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# weighted-hotspot-mutation-result-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates hotspot mutation calls from N NGS-liked short-read sets extracted from nanopore reads using a weighted-sum equation with position-based weights that sum to 1, filtering results not detected across the extracted datasets to produce final hotspot mutation calls.

## When to use

Use when Nano2NGS-Muta has produced per-set mutation detection results from N extracted short-read sets and final target hotspot mutation calls must be integrated and checked against a complete check-hotspot.list.

## When NOT to use

- Input is already conventional short-read NGS data with no nanopore long-read conversion step — no extraction sets exist to integrate.
- The goal is unbiased genome-wide variant discovery rather than detection of mutations in a predefined hotspot panel; Nano2NGS-Muta is designed for hotspot panel detection (e.g. clinical diagnosis).
- Reads need base-level error correction before calling — the framework only converts long reads into a set of short reads with no error correction for reads and/or bases.

## Inputs

- nanopore FASTQ file (.fq/.fastq, possibly gzipped)
- hotspot BED configure file (chr, start, end, gene symbol, mutation, optional transcription ID; minimal position kept per probe)
- check-hotspot.list file containing ALL mutation positions
- N per-set mutation detection results from NGS analysis of the extracted short-read sets

## Outputs

- hotspot_mutation_calls.txt with final target hotspot mutation calls
- integrated weighted-sum result across the N short-read sets

## How to apply

After nano2ngs-muta.pl has cut L-length paired seed windows from the nanopore FASTQ into N NGS-liked short-read sets (e.g. -read_len 101, -position_step 10), run an NGS mutation-analysis pipeline (e.g. the GATK Best Practice pipeline) on each of the N sets. Because one probe may span multiple mutations and the hotspot BED configure file keeps only the minimal position, read the check-hotspot.list file, which must contain ALL mutation positions, so mutations lost from the configure file are still evaluated. Integrate the N per-set detection results with the weighted-sum equation using position-based weights that sum to 1, leveraging the higher base quality expected in the middle of the reads relative to the ends. Filter out results that were not detected across the extracted datasets, then write the final target hotspot mutation calls to hotspot_mutation_calls.txt.

## Related tools

- **Nano2NGS-Muta** (Extracts the N NGS-liked short-read sets from nanopore reads whose per-set results are integrated; invoked via nano2ngs-muta.pl with -fq, -read_len, -position_step, -configure, -process) — https://github.com/langjidong/Nano2NGS
- **GATK Best Practice pipeline** (NGS mutation-analysis pipeline applied to each extracted short-read set to produce the per-set hotspot mutation results)

## Examples

```
perl nano2ngs-muta.pl -fq nanopore.fq -read_len 101 -position_step 10 -configure Hotspot.bed -process 8
```

## Evaluation signals

- Every final target hotspot mutation is called against the complete check-hotspot.list (all mutation positions present).
- The integration weights over the N short-read sets sum to exactly 1.
- Results not detected across the extracted datasets are filtered out of hotspot_mutation_calls.txt.
- Per-probe configure file entries retain only the minimal position; mutations lost to minimal-position truncation are recovered via the check-hotspot.list evaluation.
- Validation against the GIAB NA12878 project.NIST.hc.snps.indels.vcf truth set when using the NA12878 Nanopore benchmark data.

## Limitations

- Limit of detection (LoD) not yet established; more samples needed for evaluation.
- Converted short reads may contain lower or similar error rates to the long reads since the framework performs no error correction for reads and/or bases.
- Selection of read length, position of hotspots, and efficiency of software remain open problems in Nano2NGS-Muta.
- Different NGS analysis algorithms can produce different result features, so actual data are required for LoD analysis and background noise evaluation.

## Evidence

- [other] Read the check-hotspot.list file, which must contain ALL mutation positions: [evidence span withheld — anchor status: not_found]
- [other] Integrate the N per-set detection results using the weighted-sum equation: [evidence span withheld — anchor status: not_found]
- [intro] there is higher base quality in the middle than in the ends of the reads: [evidence span withheld — anchor status: not_found]
