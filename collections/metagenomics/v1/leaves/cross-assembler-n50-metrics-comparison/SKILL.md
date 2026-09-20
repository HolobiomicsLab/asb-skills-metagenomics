---
name: cross-assembler-n50-metrics-comparison
description: Use when a task requires comparing de novo assembler performance on identical
  short-read input using continuity (N50) and accuracy (structural/substitution/copy
  number error) metrics against published baseline numbers.
when_to_use_negative:
- The goal is metagenomic or single-cell assembly where MEGAHIT (not SOAPdenovo) is
  the appropriate assembler per the README
- The input is a single assembler's output that already has quality metrics and no
  cross-assembler baseline exists for comparison
- The task involves single-molecule long reads rather than Illumina GA short reads,
  which SOAPdenovo is specifically designed to assemble
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: SOAPdenovo2
  role: Primary assembler under evaluation; runs the six-module pipeline (pregraph/contig/map/scaff
    incl. GapCloser) on the Assemblathon1 reads
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1
  role: Initial-release baseline assembler run on the same reads to quantify SOAPdenovo2
    improvements
- name: SOAPdenovo1.05
  role: The Assemblathon1-era version of SOAPdenovo, used as a second baseline assembly
- name: ALLPATHS-LG
  role: External assembler whose published Table 1 N50 values anchor the competitiveness
    comparison
- name: GapCloser
  role: SOAPdenovo2 module that fills scaffold gaps using context and PE reads information
    as the final pipeline step
  repo: https://github.com/aquaskyline/SOAPdenovo2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:4f42ba44fe4bf5171ba53b8ad53e59545f3e108bfdf8d0102c1e8e60ab09d28f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/SOAPdenovo2
  tools:
  - ALLPATHS-LG
  - GapCloser
  - SOAPdenovo
  - SOAPdenovo1
  - SOAPdenovo1.05
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cross-assembler-n50-metrics-comparison

## Summary

Reproduces a benchmark comparison of genome assemblers (SOAPdenovo2 vs SOAPdenovo1/1.05 vs ALLPATHS-LG) on a shared read dataset, evaluating contig/scaffold N50 and structural, substitution, and copy number error rates to quantify improvement or competitiveness.

## When to use

Use when a task requires comparing de novo assembler performance on identical short-read input using continuity (N50) and accuracy (structural/substitution/copy number error) metrics against published baseline numbers.

## When NOT to use

- The goal is metagenomic or single-cell assembly where MEGAHIT (not SOAPdenovo) is the appropriate assembler per the README
- The input is a single assembler's output that already has quality metrics and no cross-assembler baseline exists for comparison
- The task involves single-molecule long reads rather than Illumina GA short reads, which SOAPdenovo is specifically designed to assemble

## Inputs

- Assemblathon1 benchmark dataset (simulated reads, reference [11]) in FASTA/FASTQ read format
- Source code of SOAPdenovo2, SOAPdenovo1, and SOAPdenovo1.05
- Published ALLPATHS-LG comparison numbers (paper Table 1)

## Outputs

- SOAPdenovo2, SOAPdenovo1, and SOAPdenovo1.05 assemblies (*.contig and *.scafSeq files)
- Comparison table of contig N50, scaffold N50, structural error, substitution error, and copy number error across assemblers
- Reported deltas: error reductions of 90.12%/92.13%/69.47% vs SOAPdenovo1 and N50 ratios ~1.53x/1.84x vs ALLPATHS-LG

## How to apply

Obtain the shared benchmark read set (here, the simulated Assemblathon1 dataset, reference [11]) and build each assembler from source (compile SOAPdenovo2 with 'make' at the root of the unpacked folder; use 'make debug=1' only for a debug build). Run the full SOAPdenovo2 pipeline — read error correction, de Bruijn graph construction, contig assembly, paired-end reads mapping, scaffold construction, and gap closure with the GapCloser module — then run SOAPdenovo1 and SOAPdenovo v1.05 on the same reads to generate baseline assemblies. Evaluate every assembly for contig N50, scaffold N50, structural error, substitution error, and copy number error, anchoring results against ALLPATHS-LG numbers from the paper's Table 1. Validation anchors: scaffold N50 nearly an order of magnitude longer than SOAPdenovo1, error reductions of 90.12% (structural), 92.13% (substitution), and 69.47% (copy number) versus SOAPdenovo1, and contig/scaffold N50 approximately 1.53x/1.84x those of ALLPATHS-LG (Table 1, Figure 1).

## Related tools

- **SOAPdenovo2** (Primary assembler under evaluation; runs the six-module pipeline (pregraph/contig/map/scaff incl. GapCloser) on the Assemblathon1 reads) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1** (Initial-release baseline assembler run on the same reads to quantify SOAPdenovo2 improvements)
- **SOAPdenovo1.05** (The Assemblathon1-era version of SOAPdenovo, used as a second baseline assembly)
- **ALLPATHS-LG** (External assembler whose published Table 1 N50 values anchor the competitiveness comparison)
- **GapCloser** (SOAPdenovo2 module that fills scaffold gaps using context and PE reads information as the final pipeline step) — https://github.com/aquaskyline/SOAPdenovo2

## Examples

```
SOAPdenovo2 all -s config_file -K 63 -R -F -o assemblathon1_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- SOAPdenovo2 scaffold N50 on Assemblathon1 is nearly an order of magnitude longer than SOAPdenovo1's
- Error reductions vs SOAPdenovo1 match the reported values: structural error 90.12%, substitution error 92.13%, copy number error 69.47%
- SOAPdenovo2 contig N50 and scaffold N50 are approximately 1.53x and 1.84x those of ALLPATHS-LG (Table 1, Figure 1)
- Each assembler is run on the identical read set so deltas are attributable to the assembler, not the input
- The SOAPdenovo2 build succeeds with plain 'make' (GNU make) and the pipeline completes all six modules including GapCloser gap closure

## Limitations

- SOAPdenovo targets 64-bit Linux with a minimum of 5G physical memory; big genomes like human require about 150 GB
- The comparison rests on a simulated benchmark dataset (Assemblathon1 reference [11]); absolute metrics may not transfer to real genomes
- Sparse pregraph module typically needs 2-10bp smaller kmer length to match the original pregraph performance, so k-mer choice affects comparability

## Evidence

- [abstract] Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo and is competitive to other assemblers on both assembly length and accuracy.: "Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo and is competitive to other assemblers on both assembly length and accuracy."
- [results] Error reductions vs SOAPdenovo1: "reduction of structural error by 90.12%, substitution error by 92.13%, and copy number error by 69.47% (Table 1, Figure 1)"
- [results] N50 ratios vs ALLPATHS-LG: "SOAPdenovo2 produced contig N50 and scaffold N50 that were approximately 1.53 and 1.84-times longer"
- [readme] Build and run procedure: [evidence span withheld — anchor status: external_doc]
- [readme] Typical full-run invocation: [evidence span withheld — anchor status: external_doc]
