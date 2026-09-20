---
name: genome-de-novo-assembly
description: Use when you have Illumina short reads (FASTA/FASTQ/BAM, paired-end or
  mate-pair) from a genome project and need a draft assembly (contigs and scaffold
  sequences) evaluated by N50 and error rates.
when_to_use_negative:
- Input is single-cell or metagenomics data — the README points to MEGAHIT, which
  generates longer contigs with less memory for such data
- Only long reads or a hybrid strategy is available — SOAPdenovo2 is specially designed
  for Illumina short reads
- The machine is 32-bit or has under ~5 GB physical memory for small genomes (150
  GB needed for human-sized projects)
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: 'Primary assembler: runs error correction, de Bruijn/sparse-pregraph construction,
    contig assembly, PE mapping, scaffolding and gap closure'
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: GapCloser
  role: Fills scaffold gaps using read context and paired-end information (invoked
    with -F in scaff step)
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1.05
  role: Predecessor baseline assembler used for comparison on Assemblathon1 and GAGE
    benchmarks
- name: ALLPATHS-LG
  role: External assembler whose Assemblathon1 contig/scaffold N50 served as comparison
    reference
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_006
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:537dd017c9e07e54c4fa3d6b0c494b379e2417cb3f9e1c6863d2c33c60de9216
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
yes_rate: 0.8
n_claims: 15
---

# genome-de-novo-assembly

## Summary

De novo assembly of short-read (Illumina GA) sequencing data into contigs and scaffolds using SOAPdenovo2, a memory-efficient de Bruijn graph assembler with multi-k-mer graph construction, improved scaffolding and GapCloser-based gap closure.

## When to use

Use when you have Illumina short reads (FASTA/FASTQ/BAM, paired-end or mate-pair) from a genome project and need a draft assembly (contigs and scaffold sequences) evaluated by N50 and error rates.

## When NOT to use

- Input is single-cell or metagenomics data — the README points to MEGAHIT, which generates longer contigs with less memory for such data
- Only long reads or a hybrid strategy is available — SOAPdenovo2 is specially designed for Illumina short reads
- The machine is 32-bit or has under ~5 GB physical memory for small genomes (150 GB needed for human-sized projects)

## Inputs

- Illumina GA short reads in FASTA or FASTQ format (single-end, paired-in-two-files via f1=/q1=, paired-in-one-file via p=, or BAM via b=)
- Configuration file describing libraries: max_rd_len, [LIB] sections with avg_ins, reverse_seq, asm_flags, rd_len_cutoff, rank, pair_num_cutoff, map_len
- Estimated genome size (for sparse_pregraph -z)

## Outputs

- *.contig — contig sequences assembled without mate-pair information
- *.scafSeq — scaffold sequences with gaps (final contigs obtainable by breaking scaffolds at gap regions)
- Assembly statistics (contig N50, scaffold N50, genome coverage, error counts)

## How to apply

Compile SOAPdenovo2 from source with GNU make at the root of the unpacked folder ('make debug=1' only for a debug build), or use the pre-compiled binary. Write a configuration file with a global max_rd_len and one [LIB] section per library specifying avg_ins, reverse_seq, asm_flags, rd_len_cutoff, rank, pair_num_cutoff and map_len. Run the full pipeline ('all' mode) covering read error correction, de Bruijn graph (or sparse_pregraph) construction, contig assembly, paired-end reads mapping, scaffold construction, and gap closure with GapCloser (-F), typically with an odd K-mer between 13 and 63 (e.g. -K 63 -R). For large genomes use sparse_pregraph with -z set slightly above the estimated genome size; expect ~150 GB memory for a human-sized genome, with peak consumption reduced ~2/3 versus SOAPdenovo1. Evaluate the resulting .contig and .scafSeq files for contig/scaffold N50, structural error, substitution error and copy number error against a reference (e.g. Assemblathon1 or GAGE benchmarks).

## Related tools

- **SOAPdenovo2** (Primary assembler: runs error correction, de Bruijn/sparse-pregraph construction, contig assembly, PE mapping, scaffolding and gap closure) — https://github.com/aquaskyline/SOAPdenovo2
- **GapCloser** (Fills scaffold gaps using read context and paired-end information (invoked with -F in scaff step)) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1.05** (Predecessor baseline assembler used for comparison on Assemblathon1 and GAGE benchmarks)
- **ALLPATHS-LG** (External assembler whose Assemblathon1 contig/scaffold N50 served as comparison reference)

## Examples

```
${bin} all -s config_file -K 63 -R -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- On Assemblathon1, scaffold N50 should be nearly an order of magnitude longer than SOAPdenovo1's, with structural error reduced by 90.12%, substitution error by 92.13% and copy number error by 69.47%
- Contig and scaffold N50 should be approximately 1.53x and 1.84x those of ALLPATHS-LG on Assemblathon1
- On GAGE datasets, correct assembly length should increase roughly 3–80 fold versus SOAPdenovo1
- Pipeline completes all steps (pregraph/sparse_pregraph, contig, map, scaff) with populated .contig and .scafSeq outputs
- Peak memory consumption should be ~2/3 lower than SOAPdenovo1 at the point of largest consumption

## Limitations

- K-mer size must be an odd number within the supported range; larger K-mers increase uniqueness but require deeper sequencing depth and longer read lengths
- sparse_pregraph typically needs a 2–10 bp smaller K-mer than the original pregraph module to achieve the same performance, and -z should be set slightly larger than the true genome size
- Runs on 64-bit Linux with minimum 5 GB physical memory; human-sized genomes require about 150 GB
- If unsure about config parameters, they can be omitted (defaults apply), but mis-specified avg_ins/reverse_seq/asm_flags affect scaffolding quality

## Evidence

- [abstract] Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo: "Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo"
- [results] SOAPdenovo2 produced contig N50 and scaffold N50 that were approximately 1.53 and 1.84-times longer: "SOAPdenovo2 produced contig N50 and scaffold N50 that were approximately 1.53 and 1.84-times longer"
- [readme] ${bin} all -s config_file -K 63 -R -o graph_prefix 1>ass.log 2>ass.err: [evidence span withheld — anchor status: external_doc]
- [readme] It runs on 64-bit Linux system with a minimum of 5G physical memory. For big genomes like human, about 150 GB memory would be required.: [evidence span withheld — anchor status: external_doc]
