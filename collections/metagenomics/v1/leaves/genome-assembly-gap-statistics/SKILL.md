---
name: genome-assembly-gap-statistics
description: Use when a draft assembly contains N-wildcard gaps between contigs in
  scaffold sequences and the goal is to close them with short-read data (e.g. GAGE
  PE datasets) and quantify remaining gap content.
when_to_use_negative:
- The input consists of unassembled contigs with no scaffold gap regions to close
- The goal is initial scaffold construction or PE-read mapping rather than gap closure
  (use the pregraph/contig/map/scaff steps without -F instead)
- Only long reads or non-Illumina data are available — GapCloser is designed for Illumina
  GA short-read assemblies
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: Assembler providing the GapCloser module and the scaff -F gap-filling step;
    compiled from source with 'make'
  repo: github.com/aquaskyline/SOAPdenovo2
- name: GapCloser
  role: Standalone module that replaces N-wildcard gaps using read context and PE
    reads information, collecting all reads aligned in previous iterative cycles for
    local reassembly
  repo: github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo
  role: Predecessor whose GapCloser used only current-cycle aligned reads; baseline
    for comparison of gap-closing improvements
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:df55ccbd5b3703b9ddda37e46a74aac8637d44dac4f67d3cdbee192ae4a8be6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapCloser
  - SOAPdenovo
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# genome-assembly-gap-statistics

## Summary

Reconstructs and evaluates the SOAPdenovo2 GapCloser-style gap closing step, replacing N-wildcard gap regions in scaffold FASTA files via local reassembly with paired-end reads, then tabulating closed-gap counts and residual N-run statistics.

## When to use

Use when a draft assembly contains N-wildcard gaps between contigs in scaffold sequences and the goal is to close them with short-read data (e.g. GAGE PE datasets) and quantify remaining gap content.

## When NOT to use

- The input consists of unassembled contigs with no scaffold gap regions to close
- The goal is initial scaffold construction or PE-read mapping rather than gap closure (use the pregraph/contig/map/scaff steps without -F instead)
- Only long reads or non-Illumina data are available — GapCloser is designed for Illumina GA short-read assemblies

## Inputs

- Draft scaffold FASTA file with N-wildcard gap regions between contigs
- Paired-end short-read dataset (FASTA/FASTQ, e.g. GAGE benchmark PE reads)
- SOAPdenovo2-style configuration file with [LIB] sections (avg_ins, reverse_seq, asm_flags, rd_len_cutoff, rank, pair_num_cutoff, map_len)

## Outputs

- Updated scaffold FASTA file (e.g. *.scafSeq) with reassembled sequences substituted into gaps
- Per-scaffold table of closed-gap counts
- Residual gap-length statistics (number and total length of remaining N-runs)

## How to apply

Compile SOAPdenovo2/GapCloser by running 'make' at the unpacked source folder root ('make debug=1' for a debug build) to obtain the GapCloser binary. Prepare a draft scaffold FASTA containing N-wildcard gap regions and a configuration file listing PE libraries with library parameters such as avg_ins, reverse_seq, asm_flags=4 (reads used only for gap closure), rank, pair_num_cutoff (minimum 3 for PE reads), and map_len (minimum 32 for PE reads). Run the gap closing step (e.g. the scaff module with the -F flag, or the GapCloser module directly), which replaces scaffold gap wildcards using read context and PE reads information; unlike the original SOAPdenovo GapCloser, SOAPdenovo2's version collects all reads aligned in previous iterative cycles, not only the current cycle, improving conflict resolution in repetitive regions. Substitute locally reassembled sequences into the gaps and write updated scaffolds to FASTA (e.g. *.scafSeq). Finally, count closed gaps and compute residual gap statistics (number and total length of remaining N-runs) per scaffold, using -G (gapLenDiff, default 50) as the allowed length difference between estimated and filled gaps.

## Related tools

- **SOAPdenovo2** (Assembler providing the GapCloser module and the scaff -F gap-filling step; compiled from source with 'make') — github.com/aquaskyline/SOAPdenovo2
- **GapCloser** (Standalone module that replaces N-wildcard gaps using read context and PE reads information, collecting all reads aligned in previous iterative cycles for local reassembly) — github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo** (Predecessor whose GapCloser used only current-cycle aligned reads; baseline for comparison of gap-closing improvements)

## Examples

```
make && ./SOAPdenovo-127mer scaff -g graph_prefix -F -G 50 1>scaff.log 2>scaff.err
```

## Evaluation signals

- Closed-gap counts are tabulated for every scaffold in the post-closing scaffold set
- Number and total length of residual N-runs are computed over the final scaffold FASTA and are reduced relative to the input scaffold
- Gap closing improvement is consistent with the module's design goal of improved gap closing optimized for large genomes
- Filled gap lengths stay within the -G gapLenDiff tolerance (default 50 bp) of estimated insert-size-based gap lengths
- On benchmarks such as GAGE, gap closing contributes to SOAPdenovo2 surpassing SOAPdenovo1 on assembly length and accuracy

## Limitations

- Conflicting bases in repetitive regions may remain unresolved even with the improved all-cycles read collection strategy
- SOAPdenovo2 requires substantial memory (minimum 5 GB; ~150 GB for human-sized genomes), so gap closing on large genomes is resource-intensive
- Reads longer than max_rd_len in the configuration file are cut to that length, which can limit gap-spanning evidence
- Libraries must be ranked correctly; libraries with the same rank are used simultaneously during scaffolding

## Evidence

- [results] GapCloser replaces scaffold gap wildcards with PE reads: "The GapCloser module was designed to replace these wildcards using the context and PE reads information"
- [supplementary] Improvement of the GapCloser module documented in supplementary methods: "Supplementary Method 5 P8 Improvement of the GapCloser module in SOAPdenovo2."
- [abstract] Gap closing improved and optimized for large genomes: "improves gap closing, and optimizes for large genome"
- [readme] scaff -F flag fills gaps in scaffold: [evidence span withheld — anchor status: external_doc]
- [readme] Scaffold output contains gap regions from which contigs can be recovered: [evidence span withheld — anchor status: external_doc]
