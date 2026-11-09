---
name: de-bruijn-graph-construction
description: Use when assembling Illumina short reads de novo and peak memory during
  de Bruijn graph construction is limiting, or when benchmarking sparse-vs-dense DBG
  memory tradeoffs.
when_to_use_negative:
- Input is metagenomic or single-cell data better served by MEGAHIT, which the SOAPdenovo2
  README notes generates longer contigs with less memory
- The goal is finished scaffolding only and read layout is already assembled into
  contigs — use the map/scaff or SOAPdenovo-fusion steps instead of rebuilding the
  graph
- The read set is so small that memory during graph construction is not a constraint,
  removing the sparse mode's rationale
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: SOAPdenovo2
  role: Provides the sparse_pregraph module that builds the sparse de Bruijn graph,
    reducing memory consumption in graph construction
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo
  role: Predecessor assembler whose standard dense pregraph build serves as the memory
    baseline for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:ce007362f2d1009668e7961606f877868e8d6dca80887fa46624cc2ea8803e79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/SOAPdenovo2
  tools:
  - SOAPdenovo
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# de-bruijn-graph-construction

## Summary

Constructs a de Bruijn graph from short reads, either with the standard pregraph module or SOAPdenovo2's sparse de Bruijn graph mode, which groups linear unique k-mers to cut memory consumption during graph construction at the cost of a shorter k-mer length.

## When to use

Use when assembling Illumina short reads de novo and peak memory during de Bruijn graph construction is limiting, or when benchmarking sparse-vs-dense DBG memory tradeoffs.

## When NOT to use

- Input is metagenomic or single-cell data better served by MEGAHIT, which the SOAPdenovo2 README notes generates longer contigs with less memory
- The goal is finished scaffolding only and read layout is already assembled into contigs — use the map/scaff or SOAPdenovo-fusion steps instead of rebuilding the graph
- The read set is so small that memory during graph construction is not a constraint, removing the sparse mode's rationale

## Inputs

- Illumina short-read FASTA/FASTQ (or BAM) files declared in a SOAPdenovo2 config file with [LIB] sections (avg_ins, reverse_seq, asm_flags, rd_len_cutoff, rank, pair_num_cutoff, map_len)
- Estimated genome size for the sparse_pregraph -z parameter

## Outputs

- Graph files with the chosen output prefix (graph_prefix) usable by the contig/map/scaff steps
- Contig sequences (*.contig) after the contig step
- Benchmark table (e.g. memory_benchmark.tsv) with columns mode, peak_memory_mb, contig_n50_bp

## How to apply

Compile SOAPdenovo2 with 'make' at the root of the unpacked folder, then run the sparse graph build via 'sparse_pregraph' with the config file, a k-mer size (-K), and a genome-size estimate (-z, mandatory and set slightly larger than the real genome size to allocate memory). The sparse method cuts reads into k-mers and combines large numbers of linear unique k-mers as a group rather than storing them independently, reducing memory; it typically needs a k-mer 2-10 bp smaller than the dense pregraph to achieve equivalent performance. For comparison, run the standard dense 'pregraph' step of SOAPdenovo on the identical reads. Capture peak resident memory during graph construction with /usr/bin/time -v, then run contig/map/scaff steps and compute contig N50 for each build to quantify the assembly-continuity tradeoff. Accept the sparse result if peak memory is lower (the paper reports memory consumption ~2/3 lower at the point of largest memory consumption), while recording any N50 reduction.

## Related tools

- **SOAPdenovo2** (Provides the sparse_pregraph module that builds the sparse de Bruijn graph, reducing memory consumption in graph construction) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo** (Predecessor assembler whose standard dense pregraph build serves as the memory baseline for comparison)

## Examples

```
./SOAPdenovo2 sparse_pregraph -s config_file -K 63 -z 5000000000 -R -o graph_prefix 1>pregraph.log 2>pregraph.err
```

## Evaluation signals

- Sparse-mode peak memory during graph construction (captured with /usr/bin/time -v) is lower than the dense SOAPdenovo pregraph baseline on the identical read set
- Contig N50 computed from both builds is recorded, and any N50 reduction for the sparse build is documented rather than ignored
- The sparse_pregraph -z genome-size value exceeds the true genome size (it is only used to allocate memory)
- The k-mer used for sparse pregraph is 2-10 bp smaller than the k-mer used for the dense pregraph to match performance
- Output graph files are accepted by the downstream contig step (contig -g graph_prefix) without errors

## Limitations

- The sparse pregraph module usually needs a k-mer length 2-10 bp smaller than the original pregraph to achieve the same performance, which is the assembly tradeoff
- The -z genome-size parameter is mandatory for sparse_pregraph and must be set slightly larger than the real genome size; underestimation affects memory allocation
- SOAPdenovo requires a 64-bit Linux system with a minimum of 5G physical memory, and about 150 GB for big genomes like human

## Evidence

- [supplementary] Sparse de Bruijn graph cuts reads into k-mers and combines linear unique k-mers: "Supplementary Method 2 P5 Construction of sparse de Bruijn graph in SOAPdenovo2."
- [abstract] Memory reduction at peak: "memory consumption was ~2/3 lower during the point of largest memory consumption."
- [readme] Sparse pregraph needs smaller k-mer: [evidence span withheld — anchor status: external_doc]
- [readme] Genome size parameter for sparse pregraph: [evidence span withheld — anchor status: external_doc]
