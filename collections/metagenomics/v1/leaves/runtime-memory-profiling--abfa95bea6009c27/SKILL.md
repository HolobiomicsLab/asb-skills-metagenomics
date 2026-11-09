---
name: runtime-memory-profiling
description: Use when comparing the computational performance of two tools or implementations
  on identical inputs and settings, e.g. to verify a reported speedup or memory reduction
  for alignment-free dissimilarity calculation.
when_to_use_negative:
- The goal is correctness of dissimilarity values rather than computational performance
  — this skill measures runtime and memory only, not output agreement of the d2* values
  themselves
- The two implementations are run with non-identical settings (k-mer length, Markov
  order); ratios are then not comparable
- Inputs are unassembled NGS reads where CAFE's -R reverse-complement option changes
  the computation relative to the original implementation
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: CAFE
  role: Stand-alone executable run with -D D2star and matching -K/-M settings; its
    runtime and peak memory are profiled as the accelerated implementation under test
  repo: https://github.com/younglululu/CAFE
- name: JELLYFISH
  role: Optional k-mer counting accelerator passed to CAFE via -J <jfexe_path> during
    the profiled CAFE runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:4d9bd0924c62fa5a05fff9b1e12af8e6b13af42e3f62281016321aff04baae80
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/younglululu/CAFE
  repo_url: https://github.com/younglululu/CAFE
  tools:
  - CAFE
  - JELLYFISH
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# runtime-memory-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for profiling elapsed runtime and peak resident memory of two alternative implementations of the same computation — here CAFE versus the original Reinert et al. d2* implementation — to reproduce reported performance gains such as the 24.0x speedup and 55.3% peak-memory consumption.

## When to use

Use when comparing the computational performance of two tools or implementations on identical inputs and settings, e.g. to verify a reported speedup or memory reduction for alignment-free dissimilarity calculation.

## When NOT to use

- The goal is correctness of dissimilarity values rather than computational performance — this skill measures runtime and memory only, not output agreement of the d2* values themselves
- The two implementations are run with non-identical settings (k-mer length, Markov order); ratios are then not comparable
- Inputs are unassembled NGS reads where CAFE's -R reverse-complement option changes the computation relative to the original implementation

## Inputs

- Pairs of genome sequence FASTA files (.fasta/.fa/.fna), e.g. the virus genomes from CAFE's 'example/data' folder
- Original d2* reference implementation (Reinert et al.) executable
- CAFE stand-alone executable and a jellyfish executable (e.g. jellyfish-2.2.4/bin/./jellyfish) for accelerated k-mer counting

## Outputs

- Per-pair and aggregate runtime and peak-memory measurements for both implementations
- benchmark_table.csv containing CAFE-vs-original speedup ratios and peak-memory consumption ratios

## How to apply

Load pairs of genome sequence FASTA files (e.g. the virus genomes in CAFE's 'example/data' folder) as benchmark inputs. Run the original d2* reference implementation first, with identical settings (k-mer length; Markov order inferred automatically by BIC via -M -1), profiling elapsed runtime and peak resident memory (e.g. with /usr/bin/time -v). Then run the CAFE stand-alone executable with -D D2star and matching -K and -M settings, optionally accelerating k-mer counting with the -J jellyfish executable path, profiling identically. Compute the speedup ratio (original runtime / CAFE runtime) and memory ratio (CAFE peak memory / original peak memory), assemble per-pair and aggregate measurements into benchmark_table.csv, and check that the ratios are in the vicinity of the reported 24.0x speedup and ~55.3% memory consumption (Figure 4).

## Related tools

- **CAFE** (Stand-alone executable run with -D D2star and matching -K/-M settings; its runtime and peak memory are profiled as the accelerated implementation under test) — https://github.com/younglululu/CAFE
- **JELLYFISH** (Optional k-mer counting accelerator passed to CAFE via -J <jfexe_path> during the profiled CAFE runs)

## Examples

```
/usr/bin/time -v ./cafe -D D2star -I speciesA.fa,speciesB.fa -K 10 -M -1 -J jellyfish-2.2.4/bin/./jellyfish -O output_path
```

## Evaluation signals

- benchmark_table.csv exists and reports both the speedup ratio (original runtime / CAFE runtime) and the peak-memory ratio (CAFE memory / original memory)
- The computed speedup is in the vicinity of the reported 24.0x speedup for d2* (Figure 4)
- CAFE's peak memory consumption is in the vicinity of ~55.3% of the original implementation's peak memory
- Both implementations were profiled on identical sequence pairs with identical settings (same -K k-mer length and -M Markov order with BIC-based inference via -M -1)
- Runtime and peak memory were captured with a consistent profiling mechanism (e.g. /usr/bin/time -v) for every pair

## Limitations

- The reported 24.0x speedup and 55.3% memory figures are specific to random pairs of genome sequences with the d2* measure; other measures, read data, or hardware will yield different ratios
- CAFE provides no stated software version number or license in the README, so exact reproducibility of the benchmark may be limited
- Automatic Markov order inference via -M -1 (BIC) is described as time-consuming, which inflates absolute runtimes if not applied consistently to both implementations

## Evidence

- [results] On random pairs of genome sequences using the d2* measure, CAFE achieves a 24.0x speedup over the original implementation while consuming on average 55.3% of the peak memory.: [evidence span withheld — anchor status: not_found]
