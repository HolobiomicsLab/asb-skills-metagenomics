---
name: web-service-metatranscriptome-annotation
description: Use when a metatranscriptome analysis pipeline depends on an external
  web annotation service (e.g., MG-RAST as in SAMSA 1.0), or when a web-service runtime/accuracy
  baseline is needed to compare against a standalone pipeline.
when_to_use_negative:
- Input files contain millions of sequences per sample and a fast, fully controllable
  local pipeline is required — web services may become oversubscribed and slow
- Custom reference databases must be mapped against — MG-RAST and COMAN do not support
  custom reference databases
- Maximum reproducibility is required — the web service's software and database are
  not under user control
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3174
tools:
- name: MG-RAST
  role: Web-based annotation service used by SAMSA 1.0 to annotate metatranscriptome
    reads remotely; serves as the runtime baseline in the benchmark
- name: SAMSA
  role: First open-source metatranscriptomic pipeline whose version 1.0 depended on
    the MG-RAST web service for annotation
- name: SAMSA2
  role: Standalone successor pipeline used as the faster comparison point, replacing
    the web service with local DIAMOND annotation
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:4b9a201ff2d9aa2258e07cf12f9829050d44d310d2e8cb421e648daafb3923dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - MG-RAST
  - PEAR
  - SAMSA
  - SAMSA2
  - SortMeRNA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# web-service-metatranscriptome-annotation

## Summary

Annotates metatranscriptome sequence files by submitting them to a web-based service such as MG-RAST, which performs the annotation remotely. Used here as the SAMSA 1.0 baseline against which standalone pipelines like SAMSA2 are benchmarked for runtime and accuracy.

## When to use

Use when a metatranscriptome analysis pipeline depends on an external web annotation service (e.g., MG-RAST as in SAMSA 1.0), or when a web-service runtime/accuracy baseline is needed to compare against a standalone pipeline.

## When NOT to use

- Input files contain millions of sequences per sample and a fast, fully controllable local pipeline is required — web services may become oversubscribed and slow
- Custom reference databases must be mapped against — MG-RAST and COMAN do not support custom reference databases
- Maximum reproducibility is required — the web service's software and database are not under user control

## Inputs

- Raw metatranscriptome sequencing files in FASTQ format (e.g., five 50,000-sequence sample metatranscriptomes)
- MG-RAST web service account and data-publicization permission settings

## Outputs

- Web-service annotation results (organism and functional annotations per read)
- Recorded per-sample wall-clock runtimes for benchmarking
- Genus-level annotation accuracy figures for pipeline comparison

## How to apply

Submit the raw FASTQ metatranscriptome files (e.g., the packaged SAMSA2 sample files of 50,000 sequences each) to the MG-RAST web service and record per-sample wall-clock times for the annotation job. Note that reported runtimes are longer when permission to make the data public immediately is not granted, so grant or withhold that permission deliberately and document it. Compare the resulting totals (SAMSA 1.0 averaged 1 day 19 h 45 min on five 50,000-sequence metatranscriptomes) against a standalone pipeline such as SAMSA2 (7 h 41 min average) to quantify the speed-up. Also record annotation accuracy (SAMSA 1.0 reached 91% at genus level vs 95% for SAMSA2) and note that MG-RAST's software and database are not under user control, limiting parameter control and reproducibility.

## Related tools

- **MG-RAST** (Web-based annotation service used by SAMSA 1.0 to annotate metatranscriptome reads remotely; serves as the runtime baseline in the benchmark)
- **SAMSA** (First open-source metatranscriptomic pipeline whose version 1.0 depended on the MG-RAST web service for annotation)
- **SAMSA2** (Standalone successor pipeline used as the faster comparison point, replacing the web service with local DIAMOND annotation) — https://github.com/transcript/samsa2

## Evaluation signals

- Total runtime on five 50,000-sequence metatranscriptomes should match the reported SAMSA 1.0 baseline of 1 day 19 h 45 min under comparable permission settings
- Genus-level annotation accuracy should be approximately 91% for the web-service (SAMSA 1.0) route vs 95% for SAMSA2
- Runtimes must be documented with the data-publicization permission state, since SAMSA 1.0 runtimes are longer if permission is not given to MG-RAST to make the data public immediately
- Benchmark table should tabulate per-component runtimes (pre-processing; RefSeq; SEED Subsystems; CAZy) side by side for both pipelines and compute the SAMSA2 speed-up

## Limitations

- MG-RAST and COMAN are dependent upon a service that may become oversubscribed and slow, and they do not support mapping to custom reference databases
- SAMSA 1.0 runtimes are longer if permission is not given to MG-RAST to make the data public immediately
- One downside of MG-RAST is that the software and database are not under control of the user, limiting parameter control and reproducibility

## Evidence

- [intro] Web-based tools for metatranscriptome analysis are limited in analysis level, may become oversubscribed and slow: "both MG-RAST and COMAN are dependent upon a service that may become oversubscribed and slow, and they do not support mapping to custom reference databases"
- [intro] SAMSA 1.0 dependency on MG-RAST limits reproducibility: "One downside of MG-RAST is that the software and database are not under control of the user. For maximum reproducibility, one must have complete control over all parameters"
- [discussion] Runtime caveat for publicization permission: "SAMSA 1.0 runtimes are longer if permission is not given to MG-RAST to make the data public immediately"
- [discussion] Accuracy comparison web service vs standalone: [evidence span withheld — anchor status: not_found]
