---
name: scaffold-gap-closing
description: Use when a draft scaffold FASTA contains unresolved N-wildcard gap regions
  and paired-end Illumina short reads are available to fill those gaps via local reassembly.
when_to_use_negative:
- The draft assembly has no N-wildcard gaps left to close
- Only single-end reads or libraries with asm_flags not including 4 (gap closure)
  are available
- The goal is de novo contig assembly rather than gap filling within existing scaffolds
  (use SOAPdenovo2's pregraph/contig modules or MEGAHIT instead)
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: SOAPdenovo2
  role: Source distribution providing the GapCloser module; compiled from source with
    'make' to obtain the GapCloser binary
  repo: github.com/aquaskyline/SOAPdenovo2
- name: GapCloser
  role: Standalone module that replaces scaffold gap wildcards using read context
    and PE reads information, collecting all reads aligned to each gap for local reassembly
  repo: github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo
  role: Predecessor whose GapCloser used only reads aligned in the current cycle;
    used as the comparison baseline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:d2762b2f022aeeeb095f5d399243aa2439655e42338347d9a5ef896e024af4fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapCloser
  - SOAPdenovo
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scaffold-gap-closing

## Summary

Reconstructs the SOAPdenovo2 GapCloser workflow that replaces N-wildcard gap regions in draft scaffolds by local reassembly using read context and paired-end (PE) reads, collecting reads aligned in all previous iterative cycles to better resolve conflicting bases in repetitive regions.

## When to use

Use when a draft scaffold FASTA contains unresolved N-wildcard gap regions and paired-end Illumina short reads are available to fill those gaps via local reassembly.

## When NOT to use

- The draft assembly has no N-wildcard gaps left to close
- Only single-end reads or libraries with asm_flags not including 4 (gap closure) are available
- The goal is de novo contig assembly rather than gap filling within existing scaffolds (use SOAPdenovo2's pregraph/contig modules or MEGAHIT instead)

## Inputs

- Draft scaffold FASTA with N-wildcard gap regions
- Paired-end (PE) short-read dataset (e.g. GAGE benchmark FASTA/FASTQ reads)
- SOAPdenovo2-style configuration file describing read libraries (avg_ins, reverse_seq, asm_flags, rank, pair_num_cutoff, map_len)

## Outputs

- Updated scaffold FASTA with gap wildcards replaced by locally reassembled sequences
- Per-scaffold table of closed-gap counts
- Remaining gap-length statistics (number and total length of residual N-runs)

## How to apply

Compile the GapCloser binary by running 'make' (or 'make debug=1' for a debug build) at the unpacked SOAPdenovo2 source root. Prepare a draft scaffold FASTA containing N-wildcard gaps and a configuration file describing PE libraries (avg_ins, reverse_seq, asm_flags=4 for gap-closure-only reads, rd_len_cutoff, rank, pair_num_cutoff with minimum 3 for PE and 5 for mate-pair, and map_len with minimum 32 for PE and 35 for mate-pair). Run the GapCloser module so it collects all reads aligned to each scaffold gap — unlike the original SOAPdenovo GapCloser, SOAPdenovo2's GapCloser considers reads aligned in previous iterative cycles, not just the current cycle — and performs local reassembly of the gap region. Substitute the reassembled sequences into the scaffold gaps and write the updated scaffolds to a FASTA file. Tabulate closed-gap counts and residual N-run statistics (number and total length of remaining gap regions) per scaffold; benchmark performance against GAGE or Assemblathon1 data where available.

## Related tools

- **SOAPdenovo2** (Source distribution providing the GapCloser module; compiled from source with 'make' to obtain the GapCloser binary) — github.com/aquaskyline/SOAPdenovo2
- **GapCloser** (Standalone module that replaces scaffold gap wildcards using read context and PE reads information, collecting all reads aligned to each gap for local reassembly) — github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo** (Predecessor whose GapCloser used only reads aligned in the current cycle; used as the comparison baseline)

## Examples

```
${bin} scaff -g graph_prefix -F 1>scaff.log 2>scaff.err
```

## Evaluation signals

- Closed-gap counts are tabulated for every scaffold in the post-closing scaffold set
- Remaining gap-length statistics (number and total length of residual N-runs) are computed over the post-closing scaffold set
- Improvement in gap closing is consistent with the module's design goal of improved gap closing optimized for large genomes
- Filled gaps respect the -G gapLenDiff limit on allowed length difference between estimated and filled gap (default 50 bp)
- Resulting scaffolds show improved continuity relative to SOAPdenovo1 on the same benchmark (e.g. GAGE or Assemblathon1)

## Limitations

- Conflicting bases in repetitive regions can still challenge gap closure; the all-cycle read collection was designed to mitigate this but does not eliminate it
- Allowed length difference between estimated and filled gaps (-G, default 50) may reject fills when insert-size estimates are inaccurate
- Requires 64-bit Linux with at least 5 GB RAM; large genomes like human need ~150 GB memory
- Libraries must have asm_flags set to include 4 (gap closure) or their reads will not be used for gap filling

## Evidence

- [results] GapCloser considers all prior-cycle reads: "The GapCloser module was designed to replace these wildcards using the context and PE reads information"
- [abstract] Gap closing optimization goal: "improves gap closing, and optimizes for large genome"
- [readme] asm_flags for gap closure only: [evidence span withheld — anchor status: external_doc]
- [readme] Running the scaff step with -F enables gap filling: [evidence span withheld — anchor status: external_doc]
- [results] Gap closure among the six modules: "six modules that handle read error correction"
