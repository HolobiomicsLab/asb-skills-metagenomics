---
name: de-bruijn-graph-contig-scaffolding
description: Use when assembling Illumina GA short reads (FASTA/FASTQ/BAM, paired-end
  or mate-pair libraries) into a de novo draft genome for large plant/animal or human-sized
  genomes and you need scaffold N50 and error metrics comparable to SOAPdenovo1, ALLPATHS-LG,
  or Assemblathon1/GAGE benchmarks.
when_to_use_negative:
- Input is single-cell or metagenomics data — the README directs such data to MEGAHIT,
  which generates longer contigs with less memory
- Only a few CPUs/memory available — the README requires a 64-bit Linux system with
  at least 5 GB physical memory (~150 GB for human-sized genomes)
- The goal is comparing against published baseline assemblies rather than producing
  a new one — use evaluation-focused skills on existing Assemblathon1/GAGE assemblies
  instead
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: SOAPdenovo2
  role: 'Primary assembler: runs pregraph/sparse_pregraph, contig, map, and scaff
    steps and hosts the improved Error Correction, sparse DBG, contig building, Scaffolding,
    and GapCloser modules'
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: GapCloser
  role: Gap closure module that replaces scaffold wildcards (N gaps) using context
    and PE reads information
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1
  role: Predecessor baseline used for comparison (initial release and v1.05 used in
    Assemblathon1)
- name: ALLPATHS-LG
  role: External assembler whose Assemblathon1 contig/scaffold N50 values anchor the
    comparison (SOAPdenovo2 ~1.53x/1.84x longer)
- name: SOAPdenovo-fusion
  role: Preparation module that converts MEGAHIT contigs into files usable by SOAPdenovo's
    map and scaff modules for scaffolding
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:5ba0d27730bcc993f2bf8f6778ccb86131ba94f3cacd86d3e745733e1f2085e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/SOAPdenovo2
  tools:
  - ALLPATHS-LG
  - GapCloser
  - SOAPdenovo
  - SOAPdenovo-fusion
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
yes_rate: 0.8333333333333334
n_claims: 12
---

# de-bruijn-graph-contig-scaffolding

## Summary

Assembles Illumina GA short reads into contigs and scaffolds using SOAPdenovo2's six-module de Bruijn graph pipeline (read error correction, DBG construction, contig assembly, PE reads mapping, scaffold construction, gap closure with GapCloser), with reduced memory consumption and improved continuity/accuracy over SOAPdenovo1.

## When to use

Use when assembling Illumina GA short reads (FASTA/FASTQ/BAM, paired-end or mate-pair libraries) into a de novo draft genome for large plant/animal or human-sized genomes and you need scaffold N50 and error metrics comparable to SOAPdenovo1, ALLPATHS-LG, or Assemblathon1/GAGE benchmarks.

## When NOT to use

- Input is single-cell or metagenomics data — the README directs such data to MEGAHIT, which generates longer contigs with less memory
- Only a few CPUs/memory available — the README requires a 64-bit Linux system with at least 5 GB physical memory (~150 GB for human-sized genomes)
- The goal is comparing against published baseline assemblies rather than producing a new one — use evaluation-focused skills on existing Assemblathon1/GAGE assemblies instead

## Inputs

- Illumina GA short reads in FASTA, FASTQ, or BAM format, organized as single-end (f=/q=/p=) or paired-end/mate-pair (f1=/f2=, q1=/q2=, b=) libraries
- Configuration file with global max_rd_len and one [LIB] section per library (avg_ins, reverse_seq, asm_flags, rd_len_cutoff, rank, pair_num_cutoff, map_len)

## Outputs

- *.contig — contig sequences assembled without mate-pair information
- *.scafSeq — scaffold sequences (final contig sequences extracted by breaking at gap regions)
- Auxiliary graph/statistics files under the graph output prefix (Appendix B files)

## How to apply

Organize all libraries in a config file with [LIB] sections specifying avg_ins, reverse_seq, asm_flags, rd_len_cutoff, rank, pair_num_cutoff (min 3 for PE, 5 for mate-pair) and map_len (min 32 for PE, 35 for mate-pair). Compile from source by running 'make' at the root of SOAPdenovo-V2.04 ('make debug=1' only for a debug build), then run '${bin} all -s config_file -K 63 -R -o graph_prefix' or stepwise: pregraph (or sparse_pregraph with -z genome size set slightly larger than the real genome), contig -g -R, map -s -g, and scaff -g -F to fill gaps. Extract final contigs by breaking scaffold sequences at gap regions in *.scafSeq. Rationale: SOAPdenovo2's sparse de Bruijn graph, improved Error Correction, contig building, Scaffolding, and GapCloser modules cut peak memory ~2/3 and greatly improved Assemblathon1 scaffold N50 (nearly an order of magnitude over SOAPdenovo1) while reducing structural, substitution, and copy number errors by 90.12%, 92.13%, and 69.47%.

## Related tools

- **SOAPdenovo2** (Primary assembler: runs pregraph/sparse_pregraph, contig, map, and scaff steps and hosts the improved Error Correction, sparse DBG, contig building, Scaffolding, and GapCloser modules) — https://github.com/aquaskyline/SOAPdenovo2
- **GapCloser** (Gap closure module that replaces scaffold wildcards (N gaps) using context and PE reads information) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1** (Predecessor baseline used for comparison (initial release and v1.05 used in Assemblathon1))
- **ALLPATHS-LG** (External assembler whose Assemblathon1 contig/scaffold N50 values anchor the comparison (SOAPdenovo2 ~1.53x/1.84x longer))
- **SOAPdenovo-fusion** (Preparation module that converts MEGAHIT contigs into files usable by SOAPdenovo's map and scaff modules for scaffolding)

## Examples

```
SOAPdenovo2 all -s example.config -K 63 -R -F -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- Scaffold N50 on Assemblathon1 should be nearly an order of magnitude longer than SOAPdenovo1's and ~1.84x ALLPATHS-LG's; contig N50 ~1.53x ALLPATHS-LG's (Table 1, Figure 1)
- Error metrics vs SOAPdenovo1 must show structural error reduced by 90.12%, substitution error by 92.13%, and copy number error by 69.47%
- Peak memory consumption should be ~2/3 lower than SOAPdenovo1 at the point of largest memory consumption
- Reads longer than max_rd_len must be cut to that length; the -z genome-size estimate for sparse_pregraph should be slightly larger than the real genome size
- The pipeline completes all six modules — read error correction, de Bruijn graph construction, contig assembly, PE reads mapping, scaffold construction, gap closure — producing non-empty *.contig and *.scafSeq outputs

## Limitations

- Runs on 64-bit Linux with minimum 5 GB physical memory; human-sized genomes require about 150 GB memory
- K-mer size accepts odd values (README FAQ says 13–31; option list allows 13–63/127); larger K needs deeper sequencing and longer reads to guarantee overlap
- Sparse pregraph usually needs a 2–10 bp smaller K than the original pregraph module to achieve the same performance
- Libraries with the same rank are used simultaneously during scaffolding — ranks must be set so insert sizes from small to large provide adequate physical coverage

## Evidence

- [abstract] new algorithm design reduces memory consumption in graph construction: "we have developed its successor, SOAPdenovo2, which has the advantage of a new algorithm design that reduces memory consumption in graph construction"
- [results] six modules pipeline: "six modules that handle read error correction"
- [results] GapCloser design: "The GapCloser module was designed to replace these wildcards using the context and PE reads information"
- [results] Assemblathon1 error reductions: "copy number error by 69.47% (Table 1, Figure 1)"
- [results] N50 vs ALLPATHS-LG: "SOAPdenovo2 produced contig N50 and scaffold N50 that were approximately 1.53 and 1.84-times longer"
- [readme] install and run: [evidence span withheld — anchor status: external_doc]
- [readme] typical run command: [evidence span withheld — anchor status: external_doc]
- [readme] system requirements: [evidence span withheld — anchor status: external_doc]
- [readme] output files: "*.scafSeq
  scaffold sequences (final contig sequences can be extracted by breaking down scaffold sequences at gap regions)."
