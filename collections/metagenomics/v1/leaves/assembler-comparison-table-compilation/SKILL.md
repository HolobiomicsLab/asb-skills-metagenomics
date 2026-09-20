---
name: assembler-comparison-table-compilation
description: Use when benchmarking de novo short-read assemblers on datasets with
  known reference genomes (GAGE Staphylococcus aureus, Rhodobacter sphaeroides, human
  chr14; Assemblathon1) and you need a quantitative improvement table versus a predecessor
  or competing assembler.
when_to_use_negative:
- Input is single-cell or metagenomic sequencing data — the README directs such projects
  to MEGAHIT (scaffolded via SOAPdenovo-fusion) instead of SOAPdenovo2
- The reference genome or GAGE-style ground truth needed to compute correct assembly
  length and error counts is unavailable, so N50/error columns cannot be filled
- You only need to run a single assembly, not compare assemblers — use the plain SOAPdenovo2
  pipeline skill instead
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: Assembler under test; run via its all/pregraph (or sparse_pregraph)-contig-map-scaff
    pipeline
  repo: github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1.05
  role: Predecessor baseline run on the same GAGE and Assemblathon1 datasets for the
    improvement table
- name: ALLPATHS-LG
  role: External comparator whose contig N50 (~1.53x shorter) and scaffold N50 (~1.84x
    shorter) are tabulated on Assemblathon1
- name: GapCloser
  role: Gap-closure module whose improvement (Supplementary Method 5) is part of the
    evaluated SOAPdenovo2 pipeline
  repo: github.com/aquaskyline/SOAPdenovo2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:e689ad3aa4645b8d742a58fb014777268ece6b3791b8839c18964bbf3f0d2d58
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALLPATHS-LG
  - GapCloser
  - SOAPdenovo
  - SOAPdenovo1.05
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# assembler-comparison-table-compilation

## Summary

Compiles a head-to-head benchmark table (e.g. SOAPdenovo2 vs SOAPdenovo1.05 vs ALLPATHS-LG) on GAGE and Assemblathon1 datasets, recording correct assembly length increase, contig/scaffold N50, and structural/substitution/copy-number error counts, mirroring the paper's Table 1 and Figure 1.

## When to use

Use when benchmarking de novo short-read assemblers on datasets with known reference genomes (GAGE Staphylococcus aureus, Rhodobacter sphaeroides, human chr14; Assemblathon1) and you need a quantitative improvement table versus a predecessor or competing assembler.

## When NOT to use

- Input is single-cell or metagenomic sequencing data — the README directs such projects to MEGAHIT (scaffolded via SOAPdenovo-fusion) instead of SOAPdenovo2
- The reference genome or GAGE-style ground truth needed to compute correct assembly length and error counts is unavailable, so N50/error columns cannot be filled
- You only need to run a single assembly, not compare assemblers — use the plain SOAPdenovo2 pipeline skill instead

## Inputs

- GAGE benchmark paired-end read sets for Staphylococcus aureus, Rhodobacter sphaeroides, and human chr14
- Assemblathon1 benchmark dataset
- Configuration file (e.g. example.config) listing libraries with avg_ins, reverse_seq, asm_flags, rd_len_cutoff, rank, pair_num_cutoff, map_len, and q1/q2 or b= read file entries
- Reference genomes for GAGE evaluation
- SOAPdenovo2 (and SOAPdenovo v1.05, ALLPATHS-LG) executables or source from github.com/aquaskyline/SOAPdenovo2

## Outputs

- Comparison table (e.g. gage_assembly_evaluation_table.csv) with correct assembly length increase, contig N50, scaffold N50, and structural/substitution/copy-number error counts per assembler, mirroring Table 1 / Figure 1
- Per-assembly outputs (*.contig, *.scafSeq) and assembly logs (ass.log, ass.err)
- Memory-consumption comparison (peak usage, ~2/3 reduction reported for SOAPdenovo2)

## How to apply

Build SOAPdenovo2 from source with 'make' (per README: compile at SOAPdenovo-V2.04 root), then run its standard six-module pipeline — read error correction, (sparse) de Bruijn graph construction, contig assembly, PE reads mapping, scaffolding, and GapCloser gap closure — on each GAGE read set (S. aureus, R. sphaeroides, human chr14) and the Assemblathon1 dataset, typically via 'all -s config_file -K 63 -R'. Run SOAPdenovo v1.05 on the same datasets as the predecessor baseline and include ALLPATHS-LG as an external comparator. Evaluate each assembly against the GAGE references, computing correct assembly length increase, contig N50, scaffold N50, and structural, substitution, and copy-number error counts. Rationale: the paper's abstract claims SOAPdenovo2 'greatly surpasses its predecessor' on both length and accuracy, so the table must capture length (correct assembly length, N50s) and accuracy (three error classes) together, plus memory consumption (~2/3 lower at peak) as an efficiency column.

## Related tools

- **SOAPdenovo2** (Assembler under test; run via its all/pregraph (or sparse_pregraph)-contig-map-scaff pipeline) — github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1.05** (Predecessor baseline run on the same GAGE and Assemblathon1 datasets for the improvement table)
- **ALLPATHS-LG** (External comparator whose contig N50 (~1.53x shorter) and scaffold N50 (~1.84x shorter) are tabulated on Assemblathon1)
- **GapCloser** (Gap-closure module whose improvement (Supplementary Method 5) is part of the evaluated SOAPdenovo2 pipeline) — github.com/aquaskyline/SOAPdenovo2

## Examples

```
SOAPdenovo2 all -s config_file -K 63 -R -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- Compiled error reductions versus SOAPdenovo1 match Table 1: structural error down 90.12%, substitution error 92.13%, copy number error 69.47%
- On GAGE, correct assembly length increase of SOAPdenovo2 over SOAPdenovo1 falls in the reported ~3x to ~80x range (Tables 2 and 3)
- On Assemblathon1, computed contig N50 and scaffold N50 ratios vs ALLPATHS-LG reproduce approximately 1.53x and 1.84x
- Peak memory usage of SOAPdenovo2 is ~2/3 lower than SOAPdenovo1 at the point of largest memory consumption
- Each row of the output CSV carries all three error classes plus contig/scaffold N50 and correct assembly length, mirroring the paper's Table 1 / Figure 1 layout

## Limitations

- SOAPdenovo runs only on 64-bit Linux with a minimum of 5G physical memory; human-sized genomes require about 150 GB
- K-mer size is bounded (-K min 13, max 63/127); the sparse_pregraph module requires a -z genome-size estimate set slightly larger than the real genome size and typically needs a 2-10 bp smaller k-mer than pregraph for equal performance
- README k-mer guidance is inconsistent (options allow 13-63/127 while the FAQ says 'odd numbers between 13 and 31'), so k-mer choice needs manual validation
- Findings are specific to Illumina GA short-read data and GAGE/Assemblathon1 benchmarks; the YH scaffold N50 gain (~50-fold) does not transfer to other datasets

## Evidence

- [abstract] Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo and is competitive to other assemblers on both assembly length and accuracy.: "Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo and is competitive to other assemblers on both assembly length and accuracy."
- [results] SOAPdenovo2 contig/scaffold N50 ~1.53x/1.84x longer than ALLPATHS-LG on Assemblathon1: "SOAPdenovo2 produced contig N50 and scaffold N50 that were approximately 1.53 and 1.84-times longer"
- [results] Error reduction percentages vs SOAPdenovo1 cited from Table 1: "reduction of structural error by 90.12%"
- [readme] README build instruction grounding the make-based invocation: [evidence span withheld — anchor status: external_doc]
- [readme] System requirements constraining the benchmark hardware: [evidence span withheld — anchor status: external_doc]
