---
name: fastq-read-length-distribution-modeling
description: Use when simulating ONT-like sequencing datasets to benchmark mutation
  detection, requiring FASTQ read-length and base-quality distributions (e.g. 1020–3968
  bp, Q10–Q42) matched to the study's Methods parameters.
when_to_use_negative:
- Input is real nanopore FASTQ data (NA12878 or experimental ONT runs) — no simulation
  is needed; apply the detection pipeline directly
- Simulated ONT-like data are required to match actual experimental and sequencing
  platforms — the article notes the simulated data 'were not generated on the same
  experimental and sequencing platforms'
- The goal is metagenomic taxonomic classification simulation — that is the Nano2NGS-Meta
  module, not hotspot mutation benchmarking
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: Nano2NGS-Muta
  role: Downstream framework the simulated FASTQ datasets benchmark; converts nanopore
    long reads to NGS-liked short reads for hotspot mutation detection
  repo: https://github.com/langjidong/Nano2NGS
- name: Oxford Nanopore Technologies nanopore sequencer
  role: Source platform whose read-length (1020–3968 bp) and base-quality (Q10–Q42)
    characteristics the simulation is parameterized to mimic
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac033
    title: 'Nano2NGS-Muta: a framework for converting nanopore sequencing data to
      NGS-liked sequencing data for hotspot mutation detection'
  source_content_hash: sha256:295d020af7d5ee4698f7ddbba3413c656960d7aa4f9eab780ef362119fe77eda
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# FASTQ read-length distribution modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates simulated nanopore FASTQ reads whose lengths are drawn from a specified normal distribution (as in Nano2NGS-Muta benchmarking: mean 2467 bp, sd 783 bp, range 1020–3968 bp) so that downstream hotspot-mutation callers are evaluated on realistic ONT-like data rather than fixed-length synthetic reads.

## When to use

Use when simulating ONT-like sequencing datasets to benchmark mutation detection, requiring FASTQ read-length and base-quality distributions (e.g. 1020–3968 bp, Q10–Q42) matched to the study's Methods parameters.

## When NOT to use

- Input is real nanopore FASTQ data (NA12878 or experimental ONT runs) — no simulation is needed; apply the detection pipeline directly
- Simulated ONT-like data are required to match actual experimental and sequencing platforms — the article notes the simulated data 'were not generated on the same experimental and sequencing platforms'
- The goal is metagenomic taxonomic classification simulation — that is the Nano2NGS-Meta module, not hotspot mutation benchmarking

## Inputs

- BED-like hotspot configuration file (chr, start, end, gene symbol, mutation) covering a 12-hotspot panel (BRAF-p.V600E, EGFR-p.L858R, EGFR-p.T790M, EGFR-p.G719A, KRAS-p.G12C, KRAS-p.G13D, PIK3CA-p.H1047R, PIK3CA-p.E545K, NRAS-p.Q61R, FGFR3-p.Y375C, EGFR-p.E746A750del, EGFR-p.A767V769dup)
- Reference sequences for hotspot regions
- Simulation parameters: read-length distribution (mean 2467 bp, sd 783 bp, range 1020–3968 bp), base-quality range Q10–Q42, per-hotspot read counts (9000 wild-type, 1000 mutant)
- Target mutant frequency list: 10%, 5%, 1%, 0.5%, 0.1% with three replicates each

## Outputs

- Mixture FASTQ files per hotspot/frequency/replicate (9000 wild-type plus the appropriate mutant fraction)
- Negative-control FASTQ replicates containing only 9000 wild-type reads per hotspot
- Manifest listing hotspot, mutation type, frequency, replicate, and read counts

## How to apply

Define the hotspot panel in BED-like configuration format (chr, start, end, gene symbol, mutation) and for each hotspot draw read lengths from a normal distribution with mean 2467 bp and sd 783 bp, clipping to the 1020–3968 bp range observed in the study, with base qualities spanning Q10–Q42. Generate 9000 wild-type reads per hotspot, then engineer the target SNV or indel (e.g. BRAF-p.V600E, EGFR-p.L858R, KRAS-p.G12C) into 1000 reads. Mix wild-type and mutant reads at theoretical mutant frequencies of 10%, 5%, 1%, 0.5%, and 0.1% with three replicates per frequency, plus three negative-control replicates containing only 9000 wild-type reads. Write mixtures as FASTQ files and compile a manifest recording hotspot, mutation type, frequency, replicate, and read counts. Rationale: simulated ONT-like reads let Nano2NGS-Muta be benchmarked at known ground-truth mutant fractions that experimental data cannot provide at low frequency.

## Related tools

- **Nano2NGS-Muta** (Downstream framework the simulated FASTQ datasets benchmark; converts nanopore long reads to NGS-liked short reads for hotspot mutation detection) — https://github.com/langjidong/Nano2NGS
- **Oxford Nanopore Technologies nanopore sequencer** (Source platform whose read-length (1020–3968 bp) and base-quality (Q10–Q42) characteristics the simulation is parameterized to mimic)

## Examples

```
perl nano2ngs-muta.pl -fq simulated_hotspot_mix_1pct.fq.gz -read_len 101 -position_step 10 -configure Hotspot.bed -process 8
```

## Evaluation signals

- Each replicate FASTQ contains exactly 9000 wild-type reads plus the appropriate mutant read count for the designed mutant fraction (1000 mutant reads drawn from for 10%/5%/1%/0.5%/0.1% mixes)
- Empirical read-length distribution of simulated reads falls within 1020–3968 bp with mean ≈ 2467 bp and sd ≈ 783 bp
- Base-quality scores fall within the Q10–Q42 range
- Three negative-control replicates contain only 9000 wild-type reads per hotspot with zero mutant reads
- Manifest counts reconcile with FASTQ read counts for every hotspot, frequency, and replicate

## Limitations

- Simulated ONT or ONT-like data were not generated on the same experimental and sequencing platforms as real runs
- Read lengths and base qualities are modeled with a normal distribution, which may not capture true ONT length/quality tails
- Simulation supports hotspot SNVs/indels only; the current Nano2NGS framework does not address copy number, structural variation, gene fusion, or expression

## Evidence

- [methods] Simulate ONT-like wild-type reads per hotspot with read lengths drawn from a normal distribution (mean 2467 bp, sd 783 bp, range 1020-3968 bp) and base qualities spanning Q10-Q42: [evidence span withheld — anchor status: not_found]
- [methods] For each hotspot, 9000 wild-type reads and 1000 mutant reads were randomly generated as simulated nanopore reads: [evidence span withheld — anchor status: not_found]
- [methods] Mix wild-type and mutant reads at theoretical mutant frequencies of 10%, 5%, 1%, 0.5%, and 0.1%, with three replicates per frequency per hotspot: [evidence span withheld — anchor status: not_found]
- [methods] Verification that each replicate FASTQ contains the designed read counts and that simulated length/quality distributions fall within specified ranges: [evidence span withheld — anchor status: not_found]
- [discussion] Article discussion caveat on platform mismatch of simulated data: "Although we simulated ONT or ONT-like data, they were not generated on the same experimental and sequencing platforms"
- [readme] README describing input FASTQ handling for the framework that consumes the simulated data: "-fq	<Input File>				Input fq.gz file"
