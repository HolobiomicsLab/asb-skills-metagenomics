---
name: pseudomatching-length-computation
description: Use when classifying base-called sequencing reads against a large, repetitive
  pan-genome index in streaming mode, and per-position matching statistics (or the
  related PMLs) are needed faster than MONI's batch MS-finding allows.
when_to_use_negative:
- You need true matching statistics rather than the related PML quantity, and MONI's
  batch MS semantics must be preserved exactly
- The task requires operating directly on raw nanopore electrical signal — SPUMONI
  operates on base-called sequences, so a separate base caller must run upstream
- Depletion-target and non-target reads share substantial sequence similarity (e.g.
  conserved genes between species), where the reversed-sequence null index may be
  insufficient
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SPUMONI
  role: Host tool providing the RLE-BWT index, online PML-finding procedure, and classification
    mode
  repo: github.com/oma219/spumoni
- name: MONI
  role: Prior batch MS-finding algorithm whose computation SPUMONI's online PML procedure
    replaces; also supplies threshold structures
  repo: github.com/maxrossi91/moni
- name: r-index
  role: Run-length-encoded BWT index that PML computation LF-maps over
  repo: github.com/maxrossi91/r-index
- name: Big-BWT
  role: Prefix-free-parsing BWT construction used to build the index
  repo: github.com/alshai/Big-BWT
- name: pfp-thresholds
  role: Computes the MONI thresholds that delimit BWT jumps during PML finding
  repo: github.com/maxrossi91/pfp-thresholds
- name: sdsl-lite
  role: Succinct data structure library underpinning the index components
  repo: github.com/simongog/sdsl-lite
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:71c00e3331e1f644e6772fc0bba22c99004b781411b8c342b241801db9ad0729
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Big-BWT
  - MONI
  - SPUMONI
  - pfp-thresholds
  - r-index
  - sdsl-lite
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pseudomatching-length-computation

## Summary

SPUMONI's one-pass algorithm that computes pseudomatching lengths (PMLs) over an r-index (RLE BWT plus MONI thresholds) in place of MONI's batch matching-statistic (MS) computation, enabling streaming nanopore read classification about 3x faster than MONI.

## When to use

Use when classifying base-called sequencing reads against a large, repetitive pan-genome index in streaming mode, and per-position matching statistics (or the related PMLs) are needed faster than MONI's batch MS-finding allows.

## When NOT to use

- You need true matching statistics rather than the related PML quantity, and MONI's batch MS semantics must be preserved exactly
- The task requires operating directly on raw nanopore electrical signal — SPUMONI operates on base-called sequences, so a separate base caller must run upstream
- Depletion-target and non-target reads share substantial sequence similarity (e.g. conserved genes between species), where the reversed-sequence null index may be insufficient

## Inputs

- Reference pan-genome FASTA (e.g. many strains per species, or ZymoMC mock-community references)
- Base-called sequencing reads in FASTA/FASTQ (e.g. ZymoMC nanopore reads)
- RLE BWT of the reference plus MONI threshold structures (from pfp-thresholds / Big-BWT construction)

## Outputs

- One pseudomatching length (PML) per position of each query read
- Per-read classification / ejection report (e.g. pml_validation_report.tsv, SPUMONI -c report file)
- Optionally, compressed read representation as PMLs plus mismatch characters

## How to apply

Build the pan-genome index by cloning github.com/oma219/spumoni, compiling (cmake .., make install), and running 'spumoni build -r <ref.fa> -M -P -m -o <prefix>' to obtain the run-length-encoded BWT and the MONI threshold structures (built via Big-BWT/pfp-thresholds). Then implement or invoke the one-pass PML finder: backward LF-mapping over the RLE BWT with BWT jumps delimited by the MONI thresholds, incrementing a length variable on BWT-character match and resetting on mismatch, emitting one PML per query position in O(m log log n) time and O(r) space. Stream batches of reads (e.g. ZymoMC nanopore reads) through the finder; SPUMONI can suspend and resume the MS/PML computation as it awaits each new batch, which is what allows streaming operation. Finally, compare per-read PMLs against the published SPUMONI run in PML mode ('spumoni run -r <index> -p reads.fa -P -c') and feed the PMLs plus the null index / KS-statistic hypothesis test into the read-ejection (classification) decision.

## Related tools

- **SPUMONI** (Host tool providing the RLE-BWT index, online PML-finding procedure, and classification mode) — github.com/oma219/spumoni
- **MONI** (Prior batch MS-finding algorithm whose computation SPUMONI's online PML procedure replaces; also supplies threshold structures) — github.com/maxrossi91/moni
- **r-index** (Run-length-encoded BWT index that PML computation LF-maps over) — github.com/maxrossi91/r-index
- **Big-BWT** (Prefix-free-parsing BWT construction used to build the index) — github.com/alshai/Big-BWT
- **pfp-thresholds** (Computes the MONI thresholds that delimit BWT jumps during PML finding) — github.com/maxrossi91/pfp-thresholds
- **sdsl-lite** (Succinct data structure library underpinning the index components) — github.com/simongog/sdsl-lite

## Examples

```
git clone https://github.com/oma219/spumoni; cd spumoni; mkdir build && cd build; cmake ..; make install; spumoni build -r ref.fa -M -P -m -o idx; spumoni run -r idx -p reads.fa -P -c
```

## Evaluation signals

- Per-read PML values from the custom one-pass implementation match those produced by published SPUMONI in PML mode on the same index and reads
- Classification calls (spumoni run ... -c output) agree with the published SPUMONI report per read, recorded in pml_validation_report.tsv
- One PML is emitted per query position, with the running length incrementing on BWT-character match and resetting on mismatch
- Runtime is approximately 3x faster than MONI while achieving similar or greater accuracy, consistent with the paper's claim
- Computation correctly suspends and resumes across successive batches of streaming read data

## Limitations

- SPUMONI operates on base-called sequences, not raw electrical signal; it cannot yet directly interact with the Read Until API as control software on a nanopore sequencer, and users must run a separate base caller upstream
- A single KS-stat threshold is used for all experiments; the optimal threshold is expected to depend on read accuracy and the reference used, and no per-experiment thresholding method is provided
- The null index (reverse of positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads, e.g. conserved genes between species
- The read-level compression scheme (outputting PMLs plus mismatch characters) is described but not yet used as an aggregate statistic for read-ejection decisions

## Evidence

- [intro] SPUMONI replaces MONI's batch MS-finding with a faster online PML algorithm: [evidence span withheld — anchor status: not_found]
- [intro] PML procedure makes SPUMONI ~3x faster than MONI with streaming support: "This optimized PML-finding procedure makes SPUMONI about 3 times faster than MONI, while achieving similar (often greater) accuracy and allowing it to operate on streaming data"
- [intro] Null index hypothesis testing for ejection decisions: "SPUMONI adds a “null index” together with a hypothesis testing framework to make principled ejection choices depending on whether the observed MS lengths are longer than what would be expected by"
- [readme] SPUMONI builds RLE BWT and MONI thresholds; PML via LF-mapping: "SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).**"
- [discussion] SPUMONI base-called-sequence limitation: "it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API"
- [abstract] Streaming read classification via compressed index: "SPUMONI uses a compressed index to rapidly generate exact or approximate matching statistics in a streaming fashion."
