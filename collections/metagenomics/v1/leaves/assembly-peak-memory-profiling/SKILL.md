---
name: assembly-peak-memory-profiling
description: Use when comparing memory consumption of genome assembler graph-construction
  modes on a fixed read set, or when quantifying the assembly (N50) tradeoff introduced
  by a memory-saving algorithm such as the sparse de Bruijn graph.
when_to_use_negative:
- The goal is overall assembly quality rather than isolating graph-construction memory
  cost — full 'all' runs conflate pregraph, contig, map, and scaff memory footprints.
- Input is long reads or metagenomic data better served by MEGAHIT, which the README
  notes generates longer contigs with less memory than SOAPdenovo.
- The genome size is unknown and no reasonable -z estimate can be supplied, since
  sparse_pregraph requires -z (mandatory) to allocate memory.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: SOAPdenovo2
  role: Provides the sparse_pregraph module whose graph-construction peak memory is
    being profiled
  repo: github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo
  role: Predecessor assembler whose dense pregraph build serves as the memory baseline
    on identical reads
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:5dd0225bb8ea571859e34bed78c86656b0029ee681ab01f7bfe54fce2217ffdc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SOAPdenovo
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# assembly-peak-memory-profiling

## Summary

Benchmark peak resident memory of de Bruijn graph construction by running SOAPdenovo2's sparse_pregraph mode against the dense pregraph build of SOAPdenovo on identical short-read input (e.g., GAGE S. aureus), capturing resource usage with /usr/bin/time -v and pairing memory figures with contig N50 to expose the assembly-continuity tradeoff.

## When to use

Use when comparing memory consumption of genome assembler graph-construction modes on a fixed read set, or when quantifying the assembly (N50) tradeoff introduced by a memory-saving algorithm such as the sparse de Bruijn graph.

## When NOT to use

- The goal is overall assembly quality rather than isolating graph-construction memory cost — full 'all' runs conflate pregraph, contig, map, and scaff memory footprints.
- Input is long reads or metagenomic data better served by MEGAHIT, which the README notes generates longer contigs with less memory than SOAPdenovo.
- The genome size is unknown and no reasonable -z estimate can be supplied, since sparse_pregraph requires -z (mandatory) to allocate memory.

## Inputs

- Paired-end short-read FASTQ files (e.g., GAGE Staphylococcus aureus benchmark reads)
- SOAPdenovo2 configuration file with [LIB] section (avg_ins, reverse_seq, asm_flags, rd_len_cutoff, rank, pair_num_cutoff, map_len, q1/q2 paths)
- Estimated genome size for the sparse_pregraph -z parameter
- Compiled SOAPdenovo2 and SOAPdenovo binaries

## Outputs

- memory_benchmark.tsv with columns mode, peak_memory_mb, contig_n50_bp (one row per graph-construction mode)
- Peak resident memory readings from /usr/bin/time -v for sparse_pregraph and dense pregraph runs
- Contig assemblies (*.contig files) from both graph builds used for N50 computation

## How to apply

Compile SOAPdenovo2 from the aquaskyline/SOAPdenovo2 repository with 'make' at the root of the unpacked folder using the pre-configured Makefile. Download a public benchmark read set such as the GAGE Staphylococcus aureus short-read data and write a configuration file ([LIB] section with q1=/q2= FASTQ entries). Run the sparse mode via 'sparse_pregraph' (supplying the mandatory -z estimated genome size, slightly larger than the real genome size, for memory allocation) under '/usr/bin/time -v' to capture peak resident memory, then run the standard dense 'pregraph' of the predecessor SOAPdenovo on the identical reads under the same capture. Assemble contigs from both graph builds ('contig -g graph_prefix -R'), compute contig N50 for each, and record mode, peak_memory_mb, and contig_n50_bp in a two-row memory_benchmark.tsv. Accept the benchmark if sparse-mode peak memory is lower — the paper reports memory consumption ~2/3 lower at the point of largest memory consumption — while recording any observed N50 reduction, since the sparse pregraph module typically needs a 2-10 bp smaller k-mer to match the original pregraph's performance.

## Related tools

- **SOAPdenovo2** (Provides the sparse_pregraph module whose graph-construction peak memory is being profiled) — github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo** (Predecessor assembler whose dense pregraph build serves as the memory baseline on identical reads)

## Examples

```
/usr/bin/time -v ./SOAPdenovo2 sparse_pregraph -s config_file -K 63 -z 5000000000 -R -o sparse_prefix 1>pregraph.log 2>pregraph.err
```

## Evaluation signals

- Sparse-mode peak resident memory from /usr/bin/time -v is lower than the dense pregraph run on the identical read set (paper reports ~2/3 lower at peak).
- memory_benchmark.tsv exists with exactly the columns mode, peak_memory_mb, contig_n50_bp and one row per mode.
- Contig N50 is computed for both assemblies from the resulting *.contig files; any N50 reduction in the sparse build is recorded, not silently dropped.
- The sparse_pregraph run succeeded with a -z value slightly larger than the real genome size (README: it 'should be set a litter larger than the real genome size, it is used to allocate memory').
- Both runs used the same input reads, k-mer settings, and capture method so the memory comparison is controlled.

## Limitations

- The sparse pregraph module usually needs a 2-10 bp smaller k-mer length to achieve the same performance as the original pregraph module, which is the assembly-continuity tradeoff.
- The -z genome size estimate must be slightly larger than the true genome size; an undersized value risks allocation failure during sparse graph construction.
- The ~2/3 memory reduction figure comes from the YH human genome reassembly in the paper; smaller benchmark genomes may show different ratios.

## Evidence

- [abstract] Memory consumption reduced by ~2/3 at peak usage: "memory consumption was ~2/3 lower during the point of largest memory consumption."
- [readme] Sparse pregraph needs smaller k-mer: [evidence span withheld — anchor status: external_doc]
- [readme] sparse_pregraph requires -z genome size: [evidence span withheld — anchor status: external_doc]
- [readme] Sparse graph construction step in README: [evidence span withheld — anchor status: external_doc]
- [abstract] New algorithm reduces memory in graph construction: "which has the advantage of a new algorithm design that reduces memory consumption in graph construction"
