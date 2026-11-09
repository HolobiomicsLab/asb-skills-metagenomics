---
name: sparse-memory-efficient-graph-assembly
description: Use when assembling Illumina short reads with SOAPdenovo2 and peak memory
  during de Bruijn graph construction is the binding constraint, e.g. large genomes
  where the standard dense pregraph build exhausts RAM.
when_to_use_negative:
- Memory is not the constraint and maximal assembly continuity matters more — the
  sparse design requires a k-mer 2-10 bp shorter, which can reduce contig N50 versus
  the standard pregraph
- Input is not Illumina short reads — SOAPdenovo is specially designed to assemble
  Illumina GA short reads and 64-bit Linux with at least 5G physical memory is required
- Genome size is unknown and cannot be estimated, since -z (estimated genome size)
  is mandatory for sparse_pregraph and is used to allocate memory
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: Provides the sparse_pregraph module that builds the sparse de Bruijn graph,
    reducing memory consumption in graph construction
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo
  role: Predecessor assembler whose standard dense pregraph build serves as the memory
    and contig-N50 baseline for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:ea9871b5e8629f6628efbfc3042608d7a075ad669ac840ef1be2670e38f7ca6f
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sparse-memory-efficient-graph-assembly

## Summary

Constructs a sparse de Bruijn graph with SOAPdenovo2's sparse_pregraph module, which groups large numbers of linear unique k-mers instead of storing each k-mer independently, cutting peak memory during graph construction (reported ~2/3 lower at peak) at the cost of requiring a shorter k-mer size.

## When to use

Use when assembling Illumina short reads with SOAPdenovo2 and peak memory during de Bruijn graph construction is the binding constraint, e.g. large genomes where the standard dense pregraph build exhausts RAM.

## When NOT to use

- Memory is not the constraint and maximal assembly continuity matters more — the sparse design requires a k-mer 2-10 bp shorter, which can reduce contig N50 versus the standard pregraph
- Input is not Illumina short reads — SOAPdenovo is specially designed to assemble Illumina GA short reads and 64-bit Linux with at least 5G physical memory is required
- Genome size is unknown and cannot be estimated, since -z (estimated genome size) is mandatory for sparse_pregraph and is used to allocate memory

## Inputs

- Illumina GA short-read FASTA/FASTQ (or BAM) paired-end files organized in a SOAPdenovo config_file with [LIB] sections (q1=/q2= paths)
- Estimated genome size (for the -z parameter)
- Chosen k-mer size K (odd, min 13, max 63/127)

## Outputs

- Graph files under the output prefix (*.preGraphID, edge/preArc files) suitable for the contig, map, and scaff steps
- *.contig contig sequences without mate-pair information
- Benchmark table (e.g. memory_benchmark.tsv) with mode, peak_memory_mb, contig_n50_bp columns

## How to apply

Compile SOAPdenovo2 with 'make' at the unpacked source root and run the sparse_pregraph step instead of pregraph: 'sparse_pregraph -s config_file -K <kmer> -z <genomeSize> -R -o graph_prefix', where -z (estimated genome size, mandatory) should be set slightly larger than the true genome size because it drives memory allocation, and -g (maxKmerEdgeLength, default 15) controls the number of skipped intermediate k-mers per edge. Because the sparse pregraph module usually needs a k-mer 2-10 bp smaller than the original pregraph module to achieve comparable performance, choose K accordingly and proceed with contig, map, and scaff steps on the same graph prefix. To quantify the memory tradeoff, capture peak resident memory with /usr/bin/time -v during graph construction for both sparse_pregraph and the dense pregraph build of SOAPdenovo (predecessor) on identical reads, then assemble contigs from both and compare contig N50. Accept the result if sparse-mode peak memory is lower (paper reports ~2/3 lower at the point of largest memory consumption) while recording any N50 reduction.

## Related tools

- **SOAPdenovo2** (Provides the sparse_pregraph module that builds the sparse de Bruijn graph, reducing memory consumption in graph construction) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo** (Predecessor assembler whose standard dense pregraph build serves as the memory and contig-N50 baseline for comparison)

## Examples

```
/usr/bin/time -v ./SOAPdenovo2 sparse_pregraph -s config_file -K 53 -z 5000000000 -R -o sparse_graph_prefix 1>pregraph.log 2>pregraph.err
```

## Evaluation signals

- Sparse-mode peak resident memory (captured with /usr/bin/time -v) during graph construction is lower than the dense pregraph build on the same read set; paper reports ~2/3 lower at the point of largest memory consumption
- The k-mer used for sparse_pregraph is 2-10 bp smaller than the K used in the original pregraph module to achieve comparable performance
- Sparse_pregraph completes only when the mandatory -z genome size is supplied and set slightly larger than the real genome size
- Contig N50 computed from both builds is recorded; any N50 reduction in the sparse build is quantified rather than ignored
- The benchmark output table contains exactly the expected rows (sparse and dense modes) with columns mode, peak_memory_mb, contig_n50_bp

## Limitations

- The sparse pregraph module usually needs a k-mer 2-10 bp smaller than the original pregraph to achieve the same performance, introducing an assembly-continuity tradeoff
- The -z genome size estimate for sparse pregraph must be set a little larger than the real genome size since it is used to allocate memory; a poor estimate can waste memory or fail
- SOAPdenovo runs on 64-bit Linux with a minimum of 5G physical memory, and big genomes like human require about 150 GB even with these improvements

## Evidence

- [abstract] Memory consumption reduced by ~2/3 at peak usage: "memory consumption was ~2/3 lower during the point of largest memory consumption."
- [abstract] Sparse graph reduces memory in graph construction: "the advantage of a new algorithm design that reduces memory consumption in graph construction"
- [readme] Sparse pregraph needs smaller k-mer: [evidence span withheld — anchor status: external_doc]
- [readme] Genome size allocates memory: [evidence span withheld — anchor status: external_doc]
- [readme] Sparse pregraph CLI usage: [evidence span withheld — anchor status: external_doc]
