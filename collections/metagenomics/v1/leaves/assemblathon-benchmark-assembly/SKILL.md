---
name: assemblathon-benchmark-assembly
description: Use when evaluating or benchmarking a short-read de novo assembler against
  published baselines on the Assemblathon1 dataset, or when reproducing reported N50
  and error-rate reductions (e.g. 90.12% structural error reduction).
when_to_use_negative:
- Input is single-cell or metagenomic data — the README directs such data to MEGAHIT,
  which 'generates longer contigs and consumes less memory', with SOAPdenovo-fusion
  for scaffolding
- The goal is scaffolding contigs from another assembler rather than whole-pipeline
  assembly comparison — use SOAPdenovo-fusion (map/scaff) instead of the full benchmark
  workflow
- The genome/experiment requires k-mers above 63 (or below 13) — SOAPdenovo2's -K
  option accepts kmers only between 13 and 63/127
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: SOAPdenovo2
  role: Primary assembler under test; runs error correction, de Bruijn graph construction,
    contig assembly, PE reads mapping, scaffold construction, and gap closure
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1
  role: Baseline assembler (initial release) for the comparison of N50 and error rates
- name: SOAPdenovo1.05
  role: Baseline assembler — the SOAPdenovo v1.05 version used in Assemblathon1
- name: GapCloser
  role: SOAPdenovo2 module that fills scaffold gaps using context and PE reads information;
    improved in Supplementary Method 5
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: ALLPATHS-LG
  role: External reference assembler whose Assemblathon1 contig/scaffold N50 values
    (Table 1) anchor the comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:b1c45702fb554068b2bf590a017575a5c810d6d222db3630cccb1e284faf790f
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# assemblathon-benchmark-assembly

## Summary

A benchmarking workflow that reproduces the SOAPdenovo2 vs SOAPdenovo1/1.05 (and ALLPATHS-LG) comparison on the Assemblathon1 simulated-read dataset, quantifying scaffold N50 continuity and structural, substitution, and copy-number error rates.

## When to use

Use when evaluating or benchmarking a short-read de novo assembler against published baselines on the Assemblathon1 dataset, or when reproducing reported N50 and error-rate reductions (e.g. 90.12% structural error reduction).

## When NOT to use

- Input is single-cell or metagenomic data — the README directs such data to MEGAHIT, which 'generates longer contigs and consumes less memory', with SOAPdenovo-fusion for scaffolding
- The goal is scaffolding contigs from another assembler rather than whole-pipeline assembly comparison — use SOAPdenovo-fusion (map/scaff) instead of the full benchmark workflow
- The genome/experiment requires k-mers above 63 (or below 13) — SOAPdenovo2's -K option accepts kmers only between 13 and 63/127

## Inputs

- Assemblathon1 benchmark dataset simulated reads (public deposit, reference [11])
- SOAPdenovo2 configuration file (config_file with [LIB] sections: avg_ins, reverse_seq, asm_flags, rd_len_cutoff, rank, pair_num_cutoff, map_len; FASTA/FASTQ/BAM read files via f=/q=/f1=/f2=/q1=/q2=/p=/b= entries)
- SOAPdenovo1 (initial release) and SOAPdenovo v1.05 source builds for baseline assemblies

## Outputs

- *.contig files (contig sequences without mate-pair information)
- *.scafSeq files (final scaffold sequences; contigs extractable by breaking scaffolds at gap regions)
- Assembly evaluation metrics table: contig N50, scaffold N50, structural error, substitution error, copy number error (cf. paper Table 1, Figure 1)

## How to apply

Download the public Assemblathon1 benchmark dataset (reference [11] simulated reads) from its deposit. Build SOAPdenovo2 from source by running 'make' at the root of the unpacked folder (use 'make debug=1' only for the debug version), then run its six modules in order: read error correction, de Bruijn graph construction, contig assembly, paired-end reads mapping, scaffold construction, and gap closure with the GapCloser module (e.g. 'SOAPdenovo2 all -s config_file -K 63 -R -F -o graph_prefix'). Run SOAPdenovo1 (initial release) and SOAPdenovo v1.05 (the version used in Assemblathon1) on the same reads as baselines. Evaluate every assembly for contig N50, scaffold N50, structural error, substitution error, and copy number error, anchoring the comparison against ALLPATHS-LG numbers from Table 1 and validating against the reported reductions of 90.12% structural, 92.13% substitution, and 69.47% copy number error versus SOAPdenovo1.

## Related tools

- **SOAPdenovo2** (Primary assembler under test; runs error correction, de Bruijn graph construction, contig assembly, PE reads mapping, scaffold construction, and gap closure) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1** (Baseline assembler (initial release) for the comparison of N50 and error rates)
- **SOAPdenovo1.05** (Baseline assembler — the SOAPdenovo v1.05 version used in Assemblathon1)
- **GapCloser** (SOAPdenovo2 module that fills scaffold gaps using context and PE reads information; improved in Supplementary Method 5) — https://github.com/aquaskyline/SOAPdenovo2
- **ALLPATHS-LG** (External reference assembler whose Assemblathon1 contig/scaffold N50 values (Table 1) anchor the comparison)

## Examples

```
SOAPdenovo2 all -s config_file -K 63 -R -F -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- Reproduced error reductions versus SOAPdenovo1 match the paper: structural error reduced by 90.12%, substitution error by 92.13%, copy number error by 69.47% (Table 1, Figure 1)
- SOAPdenovo2 scaffold N50 is nearly an order of magnitude longer than SOAPdenovo1's on Assemblathon1
- SOAPdenovo2 contig N50 and scaffold N50 are approximately 1.53x and 1.84x those of ALLPATHS-LG respectively
- Assembly outputs contain both *.contig and *.scafSeq files, and final contigs can be extracted by breaking scaffolds at gap regions
- The pipeline log shows all six modules ran (pregraph/sparse_pregraph, contig, map, scaff with -F gap filling), e.g. 'SOAPdenovo2 all -s config_file -K 63 -R -o graph_prefix'

## Limitations

- SOAPdenovo2 requires a 64-bit Linux system with a minimum of 5G physical memory; ~150 GB is required for big genomes like human
- Sparse pregraph mode requires an estimated genome size (-z) that should be set slightly larger than the real genome size, and usually needs a 2-10bp smaller kmer than the original pregraph to achieve the same performance
- Results are grounded on the Assemblathon1 simulated-read benchmark and GAGE; the comparison anchors against specific baseline versions (SOAPdenovo1, v1.05) and Table 1 numbers, so metric drift may occur with different versions or datasets
- Assemblers with multiple libraries must set library 'rank' values so pairs in each rank provide adequate physical coverage; misconfigured libraries degrade scaffolding

## Evidence

- [results] scaffold N50 nearly an order of magnitude longer with 90.12% structural error reduction: "the new scaffold N50 was nearly an order of magnitude longer and the accuracy was higher"
- [results] error reduction percentages vs SOAPdenovo1: "reduction of structural error by 90.12%"
- [results] copy number error reduction figure: "copy number error by 69.47% (Table 1, Figure 1)"
- [results] ALLPATHS-LG comparison ratios: "SOAPdenovo2 produced contig N50 and scaffold N50 that were approximately 1.53 and 1.84-times longer"
- [results] Assemblathon1 benchmark used for evaluation: "To test the performance of SOAPdenovo2, we assembled the Assemblathon1 benchmark dataset"
- [readme] build instruction from README: [evidence span withheld — anchor status: external_doc]
- [readme] typical one-shot run command from README: [evidence span withheld — anchor status: external_doc]
- [readme] output files produced by assembly: "b. *.scafSeq
  scaffold sequences (final contig sequences can be extracted by breaking down scaffold sequences at gap regions)."
- [results] pipeline module list from paper: "six modules that handle read error correction"
