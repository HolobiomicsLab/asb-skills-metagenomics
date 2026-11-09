---
name: paired-end-read-local-reassembly
description: Use when a draft scaffold FASTA contains N-wildcard gap regions between
  contigs and a paired-end short-read dataset (e.g. Illumina GAGE PE reads) is available
  to fill those gaps by local reassembly.
when_to_use_negative:
- The draft assembly has no N-wildcard gap regions (contigs only, no scaffolds), so
  there is nothing for GapCloser to fill
- No paired-end or mate-pair read information is available, since gap replacement
  relies on 'the context and PE reads information'
- Gaps must be resolved using only reads from the current alignment cycle rather than
  the cumulative read set, i.e. when reproducing the original SOAPdenovo GapCloser
  behavior is the goal rather than the SOAPdenovo2 improvement
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: GapCloser
  role: Standalone module that replaces scaffold gap wildcards using read context
    and PE reads information; the SOAPdenovo2 version collects reads aligned in all
    previous iterative cycles for local reassembly
  repo: github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo2
  role: Assembler whose scaff step performs gap closure via the -F flag and whose
    workflow (pregraph/contig/map/scaff) produces the scaffolds that GapCloser fills
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo
  role: Predecessor assembler whose original GapCloser used only reads aligned in
    the current cycle; baseline for comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:e2ebfa497c9cd94d97b01f02a76f4e3ce2ad16886bdfe260cbeef4f86cf37432
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapCloser
  - SOAPdenovo
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# paired-end-read-local-reassembly

## Summary

Locally reassembles N-wildcard gap regions in draft scaffolds using paired-end (PE) short reads and read context, as implemented in the GapCloser module of SOAPdenovo2. SOAPdenovo2's GapCloser considers all reads aligned in previous iterative cycles — not only the current cycle — to better resolve conflicting bases in repetitive regions during gap closure.

## When to use

Use when a draft scaffold FASTA contains N-wildcard gap regions between contigs and a paired-end short-read dataset (e.g. Illumina GAGE PE reads) is available to fill those gaps by local reassembly.

## When NOT to use

- The draft assembly has no N-wildcard gap regions (contigs only, no scaffolds), so there is nothing for GapCloser to fill
- No paired-end or mate-pair read information is available, since gap replacement relies on 'the context and PE reads information'
- Gaps must be resolved using only reads from the current alignment cycle rather than the cumulative read set, i.e. when reproducing the original SOAPdenovo GapCloser behavior is the goal rather than the SOAPdenovo2 improvement

## Inputs

- Draft scaffold FASTA file containing N-wildcard gap regions between contigs
- Paired-end (PE) short-read dataset (FASTA/FASTQ, e.g. GAGE benchmark PE reads)
- SOAPdenovo2-style configuration file with library parameters (avg_ins, reverse_seq, asm_flags=4 for gap-closure-only libraries, rank, pair_num_cutoff, map_len)

## Outputs

- Updated scaffold FASTA file with locally reassembled sequences substituted into gap regions
- Gap-closing statistics: counts of closed gaps and number/total length of residual N-runs per scaffold

## How to apply

Compile SOAPdenovo2/GapCloser from source with 'make' at the root of the unpacked folder (or 'make debug=1' for a debug build) to obtain a GapCloser binary. Load the draft scaffold FASTA with N-wildcard gaps together with the GAGE paired-end short-read dataset; the GapCloser module replaces scaffold gap wildcards using read context and PE reads information. Unlike the original SOAPdenovo GapCloser, collect all reads aligned to each scaffold gap across previous iterative cycles for local reassembly of the gap region, which better resolves conflicting bases in repetitive regions. Substitute the locally reassembled sequences into the scaffold gaps and write updated scaffolds to a FASTA file. In the SOAPdenovo2 assembler pipeline, gap closure is triggered with the '-F (fill gaps in scaffold)' option during the scaff step, and libraries dedicated to gap closure are flagged with asm_flags=4 in the configuration file.

## Related tools

- **GapCloser** (Standalone module that replaces scaffold gap wildcards using read context and PE reads information; the SOAPdenovo2 version collects reads aligned in all previous iterative cycles for local reassembly) — github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo2** (Assembler whose scaff step performs gap closure via the -F flag and whose workflow (pregraph/contig/map/scaff) produces the scaffolds that GapCloser fills) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo** (Predecessor assembler whose original GapCloser used only reads aligned in the current cycle; baseline for comparison)

## Examples

```
GapCloser -a scaffold.fasta -b PE_reads.fastq -o gapClosed_scaffold.fasta -l 100 -p 8
```

## Evaluation signals

- Closed-gap counts are tabulated for every scaffold after the run
- Number and total length of residual N-runs over the post-closing scaffold set are computed and reported
- Improvement in gap closing is consistent with the module's reported design goal of improved gap closing optimized for large genomes
- Updated scaffold FASTA is valid and gap substitutions respect the allowed length difference between estimated and filled gap (-G gapLenDiff, default 50)
- Repetitive-region gaps show fewer conflicting-base failures than the original SOAPdenovo GapCloser because all previously aligned reads are reused

## Limitations

- Gap closure quality depends on PE read coverage and insert-size distribution; libraries dedicated to gap closure must be flagged asm_flags=4 in the config
- The allowed length difference between estimated and filled gap (default 50 bp via -G gapLenDiff) bounds how large a gap discrepancy can be corrected
- The README notes K-mer uniqueness constraints: larger K-mers need deep sequencing depth and longer read length to guarantee overlap at any genomic location, which affects local reassembly in repeats

## Evidence

- [results] GapCloser PE-read gap filling: "The GapCloser module was designed to replace these wildcards using the context and PE reads information"
- [supplementary] GapCloser improvement in SOAPdenovo2: "Supplementary Method 5 P8 Improvement of the GapCloser module in SOAPdenovo2."
- [abstract] Gap closing design goal: "improves gap closing, and optimizes for large genome"
- [readme] asm_flags for gap closure: [evidence span withheld — anchor status: external_doc]
- [readme] Fill gaps option in scaff step: [evidence span withheld — anchor status: external_doc]
