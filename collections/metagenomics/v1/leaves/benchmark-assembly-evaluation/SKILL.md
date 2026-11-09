---
name: benchmark-assembly-evaluation
description: Use when a new genome assembly (contigs/scaffolds from a de Bruijn graph
  assembler such as SOAPdenovo2) must be benchmarked against a predecessor or competitor
  on a reference-backed dataset.
when_to_use_negative:
- The assembly has no reference genome available, so error classification (structural/substitution/copy-number)
  and coverage against hg19-style references cannot be computed
- The goal is metagenomic or single-cell assembly, where MEGAHIT-style evaluation
  rather than SOAPdenovo2's large-genome benchmarking applies
- Input is raw reads only and no assembly has been produced yet — pregraph/contig/map/scaff
  must run before evaluation
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: Primary assembler under test; run as pregraph/sparse_pregraph, contig, map,
    scaff (with -F for gap filling) to produce .contig and .scafSeq outputs for benchmarking
  repo: github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1
  role: Baseline predecessor assembly against which N50, error reductions, and coverage
    gains are measured
- name: SOAPdenovo1.05
  role: Version used in the Assemblathon1 comparison
- name: ALLPATHS-LG
  role: Competing assembler used as a comparator on the Assemblathon1 benchmark (SOAPdenovo2
    contig/scaffold N50 ~1.53x and ~1.84x longer)
- name: GapCloser
  role: SOAPdenovo2 module that replaces scaffold gap wildcards using context and
    PE reads information; its improvement contributes to scaffold N50 and coverage
    gains
  repo: github.com/aquaskyline/SOAPdenovo2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:299adf4978774d152ec2725498c0333b8afcb24967ab33b03e17e479643916ed
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# benchmark-assembly-evaluation

## Summary

Evaluate de novo short-read genome assemblies by comparing an assembler's output (e.g., SOAPdenovo2 vs. SOAPdenovo1 or ALLPATHS-LG) on benchmark datasets such as Assemblathon1 and GAGE, using contig/scaffold N50, error classes, and genome coverage to judge assembly quality.

## When to use

Use when a new genome assembly (contigs/scaffolds from a de Bruijn graph assembler such as SOAPdenovo2) must be benchmarked against a predecessor or competitor on a reference-backed dataset.

## When NOT to use

- The assembly has no reference genome available, so error classification (structural/substitution/copy-number) and coverage against hg19-style references cannot be computed
- The goal is metagenomic or single-cell assembly, where MEGAHIT-style evaluation rather than SOAPdenovo2's large-genome benchmarking applies
- Input is raw reads only and no assembly has been produced yet — pregraph/contig/map/scaff must run before evaluation

## Inputs

- Benchmark paired-end short-read dataset (Assemblathon1 or GAGE)
- Assembler configuration file (config with [LIB] sections: avg_ins, reverse_seq, asm_flags, rank, pair_num_cutoff, map_len)
- Reference genome for alignment (e.g., NCBI human reference genome hg19)
- Published human SD sequences (134 Mb) for segmental duplication checks
- Assemblies from comparator assemblers (SOAPdenovo1, SOAPdenovo1.05, ALLPATHS-LG)

## Outputs

- Contig and scaffold N50 statistics per assembler/dataset
- Structural, substitution, and copy-number error counts (e.g., 90.12%, 92.13%, 69.47% reductions vs SOAPdenovo1)
- Genome coverage percentage of the reassembled genome
- .contig and .scafSeq assembly files per run

## How to apply

Assemble the benchmark dataset (e.g., Assemblathon1 or GAGE short reads) with the assembler under test and its comparators (SOAPdenovo1, SOAPdenovo1.05, ALLPATHS-LG). Run the full pipeline — pregraph/sparse_pregraph, contig, map, scaff with -F gap filling — producing .contig and .scafSeq outputs. Align assembled contigs and scaffolds to a reference (e.g., NCBI human hg19) to measure contig N50, scaffold N50, and classify errors into structural, substitution, and copy-number error categories. For human data, also align to published human SD sequences (134 Mb) to check segmental duplication recovery, and compute genome coverage (e.g., YH: 81.16% -> 93.91%). Judge success by N50 improvement and error reduction relative to the baseline assembler, not length alone.

## Related tools

- **SOAPdenovo2** (Primary assembler under test; run as pregraph/sparse_pregraph, contig, map, scaff (with -F for gap filling) to produce .contig and .scafSeq outputs for benchmarking) — github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1** (Baseline predecessor assembly against which N50, error reductions, and coverage gains are measured)
- **SOAPdenovo1.05** (Version used in the Assemblathon1 comparison)
- **ALLPATHS-LG** (Competing assembler used as a comparator on the Assemblathon1 benchmark (SOAPdenovo2 contig/scaffold N50 ~1.53x and ~1.84x longer))
- **GapCloser** (SOAPdenovo2 module that replaces scaffold gap wildcards using context and PE reads information; its improvement contributes to scaffold N50 and coverage gains) — github.com/aquaskyline/SOAPdenovo2

## Examples

```
./SOAPdenovo2 all -s config_file -K 63 -R -F -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- Contig and scaffold N50 are computed for every assembler/dataset pair and are expected to improve over the predecessor (e.g., scaffold N50 nearly an order of magnitude longer than SOAPdenovo1 on YH)
- Error counts decompose into structural, substitution, and copy-number categories, each tabulated against the reference (reported reductions: 90.12%, 92.13%, 69.47%)
- Genome coverage is reported before and after reassembly (YH: 81.16% to 93.91%)
- Peak memory consumption is recorded and compared (~2/3 lower at maximum usage for SOAPdenovo2)
- Residual gap statistics (count and total length of remaining N-runs in .scafSeq) are tabulated post gap closure

## Limitations

- SOAPdenovo targets large plant/animal genomes and needs ~5G memory minimum; human-sized genomes require ~150 GB, so small-memory environments may fail
- K-mer size trades uniqueness against required depth: larger K-mers need deeper sequencing and longer reads; odd K between 13 and 31 (up to 63/127) is accepted
- Sparse pregraph usually needs a 2-10 bp smaller kmer to match the original pregraph module's performance
- Insert-size upper bound for large libraries (>1000 bp) defaults to 1.5x avg_ins and may need tuning via -b

## Evidence

- [abstract] Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo and is competitive to other assemblers on both assembly length and accuracy.: "Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo"
- [results] GapCloser design: "The GapCloser module was designed to replace these wildcards using the context and PE reads information"
- [results] Error metrics vs reference: "by aligning them to the NCBI human reference genome hg19"
- [results] SD duplication check: "we aligned the contigs of the first version and the new version to 134 Mb of published human SD sequences"
- [readme] Pipeline steps and gap filling option: [evidence span withheld — anchor status: external_doc]
- [readme] Output file formats: "b. *.scafSeq
  scaffold sequences (final contig sequences can be extracted by breaking down scaffold sequences at gap regions)."
