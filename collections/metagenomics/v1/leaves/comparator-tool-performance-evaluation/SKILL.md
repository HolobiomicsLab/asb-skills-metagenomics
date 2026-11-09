---
name: comparator-tool-performance-evaluation
description: Use when a new host-depletion or filtering tool must be validated against
  an existing comparator on identical inputs, and retention/precision metrics such
  as microbial reads retained must be quantified and compared.
when_to_use_negative:
- Evaluating host-read removal sensitivity on real human reads (e.g. 1000 Genomes
  Project samples) rather than microbial retention on simulated bacterial reads
- Benchmarking masked-index variants (e.g. human-t2t-hla-argos985) where the expected
  retention differs (≥99.997%) from the unmasked-index comparator values
- Scenarios where input reads are not held fixed across tools, making retention percentages
  non-comparable
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Hostile
  role: Candidate tool under evaluation; run in long-read mode (Minimap2, map-ont
    preset, human-t2t-hla index) to deplete host reads and emit a JSON log of read
    counts
  repo: https://github.com/bede/hostile
- name: sra-human-scrubber (HRRT)
  role: Established comparator tool run on the identical simulated ONT read set to
    provide baseline bacterial read retention (99.98901%)
  repo: https://github.com/ncbi/sra-human-scrubber
- name: Minimap2
  role: Alignment engine underlying Hostile long-read decontamination during the benchmark
- name: PBSIM2
  role: Simulator used to generate the shared ONT benchmark reads from 985 FDA-ARGOS
    assemblies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:22bf7222322f1617b3540f698732879d995d6b35be4707b154cbec927f9c7099
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Hostile
  - Minimap2
  - PBSIM2
  - sra-human-scrubber (HRRT)
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# comparator-tool-performance-evaluation

## Summary

Benchmark a candidate host-read removal tool (e.g. Hostile) against an established comparator (NCBI HRRT/sra-human-scrubber) by running both on the same read set and computing microbial read retention, as done for simulated ONT reads from 985 FDA-ARGOS bacterial genomes.

## When to use

Use when a new host-depletion or filtering tool must be validated against an existing comparator on identical inputs, and retention/precision metrics such as microbial reads retained must be quantified and compared.

## When NOT to use

- Evaluating host-read removal sensitivity on real human reads (e.g. 1000 Genomes Project samples) rather than microbial retention on simulated bacterial reads
- Benchmarking masked-index variants (e.g. human-t2t-hla-argos985) where the expected retention differs (≥99.997%) from the unmasked-index comparator values
- Scenarios where input reads are not held fixed across tools, making retention percentages non-comparable

## Inputs

- Simulated ONT long-read FASTQ from 985 FDA-ARGOS bacterial assemblies (BioProject 231221), generated with PBSIM2
- Candidate tool output: Hostile JSON clean log with reads_in/reads_out/reads_removed
- Comparator tool output: HRRT (sra-human-scrubber) cleaned FASTQ and removed-spot counts

## Outputs

- Per-tool bacterial read retention (reads_out / reads_in), e.g. Hostile 99.98918% vs HRRT 99.98901%
- Absolute read-removal counts per tool (Hostile removed 891 reads)
- Comparator performance report confirming ≥99.989% simulated bacterial read retention

## How to apply

Acquire a fixed benchmark input — here, ONT long reads simulated with PBSIM2 from 985 reference-grade FDA-ARGOS bacterial genomes (BioProject 231221) — so both tools see identical reads. Run `hostile clean` in long-read mode (Minimap2 with the map-ont preset, secondary alignments disabled, default human-t2t-hla index) and parse its JSON log for reads_in/reads_out/reads_removed; run the comparator HRRT (sra-human-scrubber) on the same FASTQ. Compute retention as reads_out / reads_in per tool: Hostile retained 99.98918% (891 reads removed) versus HRRT 99.98901%. Report retention against the paper's threshold of at least 99.989% retention of simulated bacterial reads, and record secondary metrics such as decontamination throughput (22 Mbp/s for bacterial ONT reads with 8 alignment threads).

## Related tools

- **Hostile** (Candidate tool under evaluation; run in long-read mode (Minimap2, map-ont preset, human-t2t-hla index) to deplete host reads and emit a JSON log of read counts) — https://github.com/bede/hostile
- **sra-human-scrubber (HRRT)** (Established comparator tool run on the identical simulated ONT read set to provide baseline bacterial read retention (99.98901%)) — https://github.com/ncbi/sra-human-scrubber
- **Minimap2** (Alignment engine underlying Hostile long-read decontamination during the benchmark)
- **PBSIM2** (Simulator used to generate the shared ONT benchmark reads from 985 FDA-ARGOS assemblies)

## Examples

```
hostile clean --fastq long.ont.fastq.gz --index human-t2t-hla --threads 8
```

## Evaluation signals

- Hostile ONT bacterial retention equals 99.98918% of simulated bacterial reads (reads_out / reads_in)
- Hostile unmasked removed count equals 891 reads, matching the paper's reported value
- Comparator HRRT retention equals 99.98901% on the same read set
- Both retentions meet or exceed the paper's reported floor of at least 99.989% retention of simulated bacterial reads
- Hostile's JSON clean log parses with consistent reads_in, reads_out, and reads_removed fields (reads_out + reads_removed = reads_in)

## Limitations

- Values are for simulated ONT reads from reference-grade genomes; real metagenomes may behave differently
- The unmasked human-t2t-hla index was benchmarked here; masked indexes change the expected retention (≥99.997%)
- Prior Hostile versions had unverified index checksums (issue #20) and could corrupt empty fastq.gz files when no reads remained after decontamination (issue #24), which could affect automated benchmark logging

## Evidence

- [results] For simulated bacterial ONT reads from 985 FDA-ARGOS assemblies, Hostile retained 99.98918% of reads while HRRT retained 99.98901%.: [evidence span withheld — anchor status: not_found]
- [abstract] our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads: "our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [methods] bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads: "bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads"
- [readme] Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin.: "Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin."
- [discussion] Prevents corruption of empty `fastq.gz` files when no reads remain after decontamination.: "Prevents corruption of empty `fastq.gz` files when no reads remain after decontamination."
