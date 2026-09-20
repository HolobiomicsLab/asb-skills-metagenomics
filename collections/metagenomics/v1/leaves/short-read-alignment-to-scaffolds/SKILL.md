---
name: short-read-alignment-to-scaffolds
description: Use when a draft scaffold FASTA contains N-run wildcards between contigs
  and paired-end short-read data is available to locally reassemble and fill scaffold
  gap regions.
when_to_use_negative:
- The assembly is a fresh de novo run from reads only — use the full SOAPdenovo2 pregraph-contig-map-scaff
  pipeline with the -F flag instead of standalone gap closing
- The scaffolds contain no N-wildcard gaps to fill
- Only single-end or mate-pair data with inadequate physical coverage of gaps is available
  and no PE library with asm_flags=4 can be supplied
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: SOAPdenovo2
  role: Source distribution containing the improved GapCloser module that fills scaffold
    gaps using reads from all prior iterative cycles
  repo: github.com/aquaskyline/SOAPdenovo2
- name: GapCloser
  role: Standalone module that replaces scaffold N wildcards using read context and
    PE reads information for local gap reassembly
  repo: github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo
  role: Predecessor assembler whose GapCloser considered only current-cycle reads;
    used as the comparison baseline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:8ff1317f235531f1e7ca220b53b67a08fb05669180982641d82bba0614fdcbd0
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
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# short-read-alignment-to-scaffolds

## Summary

Close N-wildcard gaps in draft scaffolds by locally reassembling all short reads aligned at each gap using SOAPdenovo2's GapCloser, which considers reads aligned across all previous iterative cycles rather than only the current cycle.

## When to use

Use when a draft scaffold FASTA contains N-run wildcards between contigs and paired-end short-read data is available to locally reassemble and fill scaffold gap regions.

## When NOT to use

- The assembly is a fresh de novo run from reads only — use the full SOAPdenovo2 pregraph-contig-map-scaff pipeline with the -F flag instead of standalone gap closing
- The scaffolds contain no N-wildcard gaps to fill
- Only single-end or mate-pair data with inadequate physical coverage of gaps is available and no PE library with asm_flags=4 can be supplied

## Inputs

- Draft scaffold FASTA containing N-wildcard gap regions between contigs
- GAGE benchmark paired-end (PE) short-read dataset (FASTA/FASTQ/BAM) declared in a SOAPdenovo configuration file with asm_flags=4 libraries
- Compiled GapCloser binary from the SOAPdenovo2 source tree

## Outputs

- Updated scaffold FASTA with locally reassembled sequences substituted into gap wildcards
- Per-scaffold closed-gap counts
- Remaining gap-length statistics (number and total length of residual N-runs) per scaffold

## How to apply

Compile SOAPdenovo2/GapCloser from the unpacked source folder with 'make' (or 'make debug=1' for a debug build) to obtain the GapCloser binary. Load the draft scaffold FASTA with N-wildcard gap regions together with a GAGE-style paired-end short-read dataset organized in a configuration file where libraries carry asm_flags=4 for gap-closure-only reads. Run GapCloser so it collects all reads aligned to each scaffold gap — the SOAPdenovo2 method considers reads aligned in previous iterative cycles, not just the current cycle — and performs local reassembly of each gap using read context and PE reads information. Substitute the locally reassembled sequences into the scaffold gaps, write the updated scaffolds to FASTA, then count closed gaps and tabulate residual N-run number and total length per scaffold. Accept the run when gap-closing statistics are tabulated for every scaffold and the improvement is consistent with the module's design goal of improved gap closing optimized for large genomes.

## Related tools

- **SOAPdenovo2** (Source distribution containing the improved GapCloser module that fills scaffold gaps using reads from all prior iterative cycles) — github.com/aquaskyline/SOAPdenovo2
- **GapCloser** (Standalone module that replaces scaffold N wildcards using read context and PE reads information for local gap reassembly) — github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo** (Predecessor assembler whose GapCloser considered only current-cycle reads; used as the comparison baseline)

## Examples

```
./SOAPdenovo2 all -s config_file -K 63 -R -F -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- Closed-gap counts are tabulated for every scaffold in the post-closing scaffold set
- Residual gap statistics (number and total length of remaining N-runs) are computed over all scaffolds after closing
- Gap-closing improvement is consistent with the module's reported design goal of improved gap closing optimized for large genomes
- The filled sequence is consistent with the paired-end read context and PE reads information around each gap
- The GapCloser binary was built successfully from source (make / make debug=1) before running

## Limitations

- Conflicting bases in repetitive regions can still impede gap closure even with the multi-cycle read strategy
- The module is optimized for large genomes; behavior on small or metagenomic assemblies may differ (the README points to MEGAHIT for metagenomic data)
- Gap length must not differ from the estimated insert-size-derived gap length by more than the allowed difference (gapLenDiff, default 50) or the fill is rejected

## Evidence

- [supplementary] GapCloser considers reads from all iterative cycles: "Supplementary Method 5 P8 Improvement of the GapCloser module in SOAPdenovo2."
- [results] GapCloser uses read context and PE information: "The GapCloser module was designed to replace these wildcards using the context and PE reads information"
- [abstract] Gap closing optimized for large genomes: "improves gap closing, and optimizes for large genome"
- [readme] asm_flags=4 dedicates a library to gap closure: [evidence span withheld — anchor status: external_doc]
- [readme] Scaffold output format: [evidence span withheld — anchor status: external_doc]
