---
name: assembly-error-rate-quantification
description: Use when comparing de novo assembler versions or assemblies on a reference-backed
  benchmark and you need continuity (N50) plus per-error-class rates (structural,
  substitution, copy number) to claim improvement.
when_to_use_negative:
- Assembling metagenomic or single-cell data, where MEGAHIT is recommended instead
  of SOAPdenovo
- Small-genome projects on machines with under 5 GB physical memory, or human-size
  assemblies with less than ~150 GB RAM
- Benchmarks with no reference genome available, since structural/substitution/copy
  number errors cannot be quantified without a reference
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: Candidate assembler producing the assemblies whose error rates are quantified;
    run via pregraph/contig/map/scaff modules and GapCloser
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1
  role: Baseline assembler for computing relative error reductions and N50 gains
- name: SOAPdenovo1.05
  role: Version used in Assemblathon1; provides the published benchmark baseline
- name: ALLPATHS-LG
  role: Third-party assembler whose published Table 1 metrics anchor cross-assembler
    comparison
- name: GapCloser
  role: Gap closure module used in the pipeline step before error evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:7be177293574a326afd81a183437b91df3ba8af82950a58199e9c6bc5e9cdaec
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# assembly-error-rate-quantification

## Summary

Quantifies de novo genome assembly quality by measuring contig/scaffold N50 continuity alongside structural, substitution, and copy number error rates, as established for SOAPdenovo2 versus SOAPdenovo1 on the Assemblathon1 benchmark. It lets an agent judge whether an assembler improvement actually reduces assembly errors, not just length.

## When to use

Use when comparing de novo assembler versions or assemblies on a reference-backed benchmark and you need continuity (N50) plus per-error-class rates (structural, substitution, copy number) to claim improvement.

## When NOT to use

- Assembling metagenomic or single-cell data, where MEGAHIT is recommended instead of SOAPdenovo
- Small-genome projects on machines with under 5 GB physical memory, or human-size assemblies with less than ~150 GB RAM
- Benchmarks with no reference genome available, since structural/substitution/copy number errors cannot be quantified without a reference

## Inputs

- Benchmark short-read dataset (e.g. simulated Assemblathon1 reference [11] reads, Illumina GA FASTA/FASTQ, optionally BAM)
- SOAPdenovo2 configuration file with [LIB] sections specifying avg_ins, reverse_seq, asm_flags, rank, pair_num_cutoff, map_len
- Baseline assemblies from SOAPdenovo1 / SOAPdenovo1.05 and published ALLPATHS-LG Table 1 numbers for comparison

## Outputs

- Per-assembly contig N50 and scaffold N50 statistics
- Relative error reductions: structural error %, substitution error %, copy number error % versus baseline
- Assembled *.contig and *.scafSeq sequences plus assembly logs (ass.log/ass.err or per-step logs)

## How to apply

Assemble the benchmark reads with the candidate assembler and baseline(s) — e.g. SOAPdenovo2 built by 'make' from github.com/aquaskyline/SOAPdenovo2 and run through its six modules (read error correction, de Bruijn graph construction, contig assembly, paired-end reads mapping, scaffold construction, gap closure via GapCloser), against SOAPdenovo1 and SOAPdenovo v1.05 baselines. Evaluate each assembly for contig N50, scaffold N50, structural error, substitution error, and copy number error, anchoring comparisons against published Table 1 / Figure 1 numbers and competing assemblers such as ALLPATHS-LG. Report improvements as relative reductions — on Assemblathon1, SOAPdenovo2 achieved a scaffold N50 nearly an order of magnitude longer than SOAPdenovo1 with structural error reduced by 90.12%, substitution error by 92.13%, and copy number error by 69.47%. Correctness is confirmed when reproduced metrics match the reported ratios (contig/scaffold N50 approximately 1.53x/1.84x those of ALLPATHS-LG).

## Related tools

- **SOAPdenovo2** (Candidate assembler producing the assemblies whose error rates are quantified; run via pregraph/contig/map/scaff modules and GapCloser) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1** (Baseline assembler for computing relative error reductions and N50 gains)
- **SOAPdenovo1.05** (Version used in Assemblathon1; provides the published benchmark baseline)
- **ALLPATHS-LG** (Third-party assembler whose published Table 1 metrics anchor cross-assembler comparison)
- **GapCloser** (Gap closure module used in the pipeline step before error evaluation)

## Examples

```
SOAPdenovo2 all -s config_file -K 63 -R -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- Scaffold N50 of SOAPdenovo2 is nearly an order of magnitude longer than SOAPdenovo1's on Assemblathon1
- Error reductions versus SOAPdenovo1 match reported values: structural 90.12%, substitution 92.13%, copy number 69.47%
- Contig N50 and scaffold N50 are approximately 1.53x and 1.84x those of ALLPATHS-LG (Table 1, Figure 1)
- Assembly output contains both *.contig and *.scafSeq files, with final contigs extractable by breaking scaffolds at gap regions
- Compile succeeds with plain 'make'; 'make debug=1' is reserved for the debug version only

## Limitations

- Metrics are benchmark-specific: reported reductions were measured on the simulated Assemblathon1 dataset and may not transfer to real genomes
- SOAPdenovo2 requires a 64-bit Linux system with at least 5 GB memory, and ~150 GB for human-size genomes
- Assembler accepts odd K-mer sizes; k-mer choice (min 13) affects graph complexity and error profiles, and the FAQ in the README constrains parameter selection

## Evidence

- [results] scaffold N50 nearly an order of magnitude longer with error reductions: "the new scaffold N50 was nearly an order of magnitude longer and the accuracy was higher"
- [results] reported error reductions: "reduction of structural error by 90.12%"
- [results] contig/scaffold N50 ratio vs ALLPATHS-LG: "SOAPdenovo2 produced contig N50 and scaffold N50 that were approximately 1.53 and 1.84-times longer"
- [results] pipeline modules evaluated: "six modules that handle read error correction"
- [readme] typical run command from README: [evidence span withheld — anchor status: external_doc]
