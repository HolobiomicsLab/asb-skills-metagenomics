---
name: rle-bwt-backward-search
description: Use when classifying sequencing reads (e.g., nanopore reads) against
  a large, repetitive pan-genome reference and you need per-position matching statistics
  or PMLs computed in O(r) space with a streaming one-pass algorithm.
when_to_use_negative:
- Input is raw nanopore electrical signal — SPUMONI operates on base-called sequences,
  so a separate base caller must run upstream
- The goal is full alignment of reads rather than rapid classification/ejection decisions
  — an aligner like minimap2 is the appropriate tool
- There is substantial sequence similarity between depletion-target and non-target
  reads (e.g., conserved genes between species) — the null index may be insufficient
  for principled ejection
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SPUMONI
  role: Implements the one-pass PML finder over the RLE BWT with thresholds and performs
    read classification; built and run from github.com/oma219/spumoni
  repo: https://github.com/oma219/spumoni
- name: MONI
  role: Baseline batch matching-statistics algorithm over the r-index that SPUMONI's
    online PML computation replaces
  repo: https://github.com/maxrossi91/moni
- name: r-index
  role: Run-length-encoded BWT index structure used for backward LF-mapping search
  repo: https://github.com/maxrossi91/r-index
- name: Big-BWT
  role: Prefix-free-parsing tool used to build the BWT of highly repetitive reference
    collections
  repo: https://github.com/alshai/Big-BWT
- name: pfp-thresholds
  role: Computes the threshold structures from the prefix-free parsing that delimit
    BWT jumps during the PML computation
  repo: https://github.com/maxrossi91/pfp-thresholds
- name: sdsl-lite
  role: Succinct data structure library underlying the RLE BWT and rank/select support
    used by the backward search
  repo: https://github.com/simongog/sdsl-lite
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:6fcd9359807dd7c8c60118480e35608a52a5c5bd1b9e861a851ea4d30045faaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# rle-bwt-backward-search

## Summary

Performs backward LF-mapping search over a run-length-encoded BWT (r-index) with MONI threshold-delimited BWT jumps to compute matching statistics or pseudomatching lengths (PMLs) for query reads against a pan-genome index, enabling streaming read classification as implemented in SPUMONI.

## When to use

Use when classifying sequencing reads (e.g., nanopore reads) against a large, repetitive pan-genome reference and you need per-position matching statistics or PMLs computed in O(r) space with a streaming one-pass algorithm.

## When NOT to use

- Input is raw nanopore electrical signal — SPUMONI operates on base-called sequences, so a separate base caller must run upstream
- The goal is full alignment of reads rather than rapid classification/ejection decisions — an aligner like minimap2 is the appropriate tool
- There is substantial sequence similarity between depletion-target and non-target reads (e.g., conserved genes between species) — the null index may be insufficient for principled ejection

## Inputs

- Reference genome FASTA (single or multi-strain pan-genome)
- Base-called sequencing reads (FASTA/FASTQ)
- RLE BWT and MONI threshold structures built from the reference

## Outputs

- Per-position pseudomatching lengths (PMLs) or matching statistics (MSs) for each read
- Per-read classification report (e.g., SPUMONI -c classification output)

## How to apply

Build the index by constructing the run-length-encoded BWT of the reference FASTA (e.g., with Big-BWT / r-index) plus the MONI threshold structures (pfp-thresholds). For each query read, perform backward LF-mapping over the RLE BWT, using the thresholds to delimit BWT jumps: maintain a length variable that increments while the BWT character matches and resets on mismatch, emitting one pseudomatching length per position. Run the one-pass PML finder in a streaming fashion so computation can be suspended and resumed as batches of base-called reads arrive. SPUMONI's optimized PML-finding procedure is about 3 times faster than MONI's batch MS-finding; MSs or PMLs longer than expected by chance (via the null index and KS-stat hypothesis testing) support classification and ejection decisions.

## Related tools

- **SPUMONI** (Implements the one-pass PML finder over the RLE BWT with thresholds and performs read classification; built and run from github.com/oma219/spumoni) — https://github.com/oma219/spumoni
- **MONI** (Baseline batch matching-statistics algorithm over the r-index that SPUMONI's online PML computation replaces) — https://github.com/maxrossi91/moni
- **r-index** (Run-length-encoded BWT index structure used for backward LF-mapping search) — https://github.com/maxrossi91/r-index
- **Big-BWT** (Prefix-free-parsing tool used to build the BWT of highly repetitive reference collections) — https://github.com/alshai/Big-BWT
- **pfp-thresholds** (Computes the threshold structures from the prefix-free parsing that delimit BWT jumps during the PML computation) — https://github.com/maxrossi91/pfp-thresholds
- **sdsl-lite** (Succinct data structure library underlying the RLE BWT and rank/select support used by the backward search) — https://github.com/simongog/sdsl-lite

## Examples

```
git clone https://github.com/oma219/spumoni && cd spumoni && mkdir build && cd build && cmake .. && make install; spumoni build -r ref.fa -M -P -m -o idx; spumoni run -r idx -p reads.fa -P -c
```

## Evaluation signals

- Per-read PML values produced by a reimplementation must match the published SPUMONI PML-mode output on the same index and reads (e.g., ZymoMC reads)
- Classification calls (-c output) agree with the published SPUMONI report per read, recorded in a validation table such as pml_validation_report.tsv
- Runtime is comparable to or faster than MONI batch MS computation (SPUMONI reported ~3x faster than MONI) while operating in O(r) space
- PML/MS length distribution differs between target-matching reads and reads matching only the null (reverse-strand) index, consistent with the KS-stat hypothesis test
- Each read position yields exactly one PML value, with lengths bounded by the read length

## Limitations

- SPUMONI operates on base-called sequences, not raw electrical signal; it cannot directly interact with the Read Until API and requires a separate base caller upstream
- The null index (reverse of positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target and non-target reads, e.g., conserved genes between species
- A single KS-stat threshold is used for all experiments; the optimal threshold depends on read accuracy and the reference, and no per-experiment thresholding method is provided
- Throughput advantage over minimap2 is limited for human reads; improvement is expected only as the number of human genomes in the reference increases

## Evidence

- [intro] SPUMONI replaces MONI's "batch" MS-finding algorithm with a faster online algorithm that calculates a different quantity related to the MS, called the "pseudomatching length (PML)": [evidence span withheld — anchor status: not_found]
- [intro] This optimized PML-finding procedure makes SPUMONI about 3 times faster than MONI, while achieving similar (often greater) accuracy and allowing it to operate on streaming data: "This optimized PML-finding procedure makes SPUMONI about 3 times faster than MONI, while achieving similar (often greater) accuracy and allowing it to operate on streaming data"
- [methods] git clone https://github.com/oma219/spumoni cd spumoni mkdir build && cd build cmake .. make install: "git clone https://github.com/oma219/spumoni
cd spumoni
mkdir build && cd build
cmake ..
make install"
- [readme] SPUMONI is a software tool for performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).: "SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).**"
- [discussion] it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API: "it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API"
