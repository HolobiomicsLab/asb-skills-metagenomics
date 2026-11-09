---
name: read-classification-hypothesis-testing
description: Use when you must decide, from a streaming read prefix (FASTQ/base-called
  nanopore reads), whether a read belongs to a target set versus random chance, and
  a compressed pan-genome index is preferable to full alignment.
when_to_use_negative:
- When there is substantial sequence similarity between depletion-target reads and
  non-target reads (e.g. conserved genes between species) — the null index built from
  reversed sequences may then be insufficient to distinguish target from chance matches
- When operating directly on raw nanopore electrical signal via the Read Until API
  — SPUMONI operates on base-called sequences, so a separate base caller must run
  upstream
- When per-base alignment coordinates (as minimap2 provides) are required rather than
  a target/nontarget classification decision
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: SPUMONI
  role: Builds the positive and null pan-genome indexes and performs streaming read
    classification via MS/PML computation and KS-statistic hypothesis testing
  repo: https://github.com/oma219/spumoni
- name: MONI
  role: Original batch MS-finding algorithm that SPUMONI's online PML procedure replaces;
    SPUMONI is about 3 times faster than MONI with similar or greater accuracy and
    streaming support
  repo: https://github.com/maxrossi91/moni
- name: pbsim2
  role: Simulates nanopore reads (e.g. for validation scenarios where positive and
    null statistic distributions are known to differ)
  repo: https://github.com/yukiteruono/pbsim2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:13390eeafb3bb69c7e2986a2260e197d8deedd0722ab9cd56a21481d70729a9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - MONI
  - SPUMONI
  - pbsim2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# read-classification-hypothesis-testing

## Summary

Classify sequencing reads against a target set by comparing matching statistics (MSs) or pseudomatching lengths (PMLs) computed against a positive pan-genome index against a null index built from the reverse of the positive-index sequences, using a Kolmogorov-Smirnov (KS) statistic threshold and majority-of-regions voting, as implemented in SPUMONI.

## When to use

Use when you must decide, from a streaming read prefix (FASTQ/base-called nanopore reads), whether a read belongs to a target set versus random chance, and a compressed pan-genome index is preferable to full alignment.

## When NOT to use

- When there is substantial sequence similarity between depletion-target reads and non-target reads (e.g. conserved genes between species) — the null index built from reversed sequences may then be insufficient to distinguish target from chance matches
- When operating directly on raw nanopore electrical signal via the Read Until API — SPUMONI operates on base-called sequences, so a separate base caller must run upstream
- When per-base alignment coordinates (as minimap2 provides) are required rather than a target/nontarget classification decision

## Inputs

- Target reference sequences in FASTA format (<reference.fa>) for building the positive pan-genome index
- Base-called sequencing reads (e.g. nanopore FASTQ) to classify
- Pre-built SPUMONI index prefix containing both the positive (target) and null (reversed-sequence) indexes

## Outputs

- Per-read classification report file (written with the -c option) marking each read as target or nontarget
- Per-region MS or PML values for each 90 bp nonoverlapping region of each read
- KS statistics comparing observed versus null statistic distributions per region

## How to apply

Build a positive index over the target reference sequences including their reverse complements, plus a null index constructed from the reverse (not reverse-complement) of the positive-index sequences, using 'spumoni build -r <reference.fa> -M -P -m -o <prefix>' (or pull the pre-built oma219/spumoni Docker image). Stream query reads through 'spumoni run', computing MSs (with -M) or PMLs (with -P) over 90 bp nonoverlapping regions of each read against the indexed reference; longer MSs indicate a better approximate match to the index. For each region, subtract the null-index mean statistic from the observed statistic and compute the Kolmogorov-Smirnov (KS) statistic comparing observed versus null distributions — this hypothesis-testing framework makes principled ejection choices based on whether observed statistic lengths exceed what random chance would produce. Classify each read as target if the majority of its 90 bp regions exceed the KS-stat threshold (0.25 for MS, 0.10 for PML), and use the -c option to write classifications to a report file. Validate on simulated reads (e.g. E. coli via pbsim2) where positive and null statistic distributions are known to differ, confirming the report separates target from nontarget reads by majority-of-regions voting at the specified thresholds; the same single KS threshold is used across experiments since it was optimized on real nanopore data.

## Related tools

- **SPUMONI** (Builds the positive and null pan-genome indexes and performs streaming read classification via MS/PML computation and KS-statistic hypothesis testing) — https://github.com/oma219/spumoni
- **MONI** (Original batch MS-finding algorithm that SPUMONI's online PML procedure replaces; SPUMONI is about 3 times faster than MONI with similar or greater accuracy and streaming support) — https://github.com/maxrossi91/moni
- **pbsim2** (Simulates nanopore reads (e.g. for validation scenarios where positive and null statistic distributions are known to differ)) — https://github.com/yukiteruono/pbsim2

## Examples

```
spumoni build -r ref.fa -M -P -m -o idx && spumoni run -i idx -q reads.fq -c report.txt -M
```

## Evaluation signals

- On simulated reads where positive and null statistic distributions are known to differ, the classification report separates target from nontarget reads by majority-of-regions voting
- KS statistics are computed per 90 bp region after subtracting the null-index mean statistic, and reads are only called target when the majority of regions exceed the threshold (0.25 for MS, 0.10 for PML)
- The -c option produces a report file with per-read classifications that can be diffed against known truth labels
- Accuracy should be in the same range as minimap2 when run against an index containing many strains per species
- The null index must contain the reverse (not reverse-complement) of the positive-index sequences; verify index construction used -M -P -m flags so both MS and PML statistics are available

## Limitations

- A single KS-stat threshold is used for all experiments; the optimal threshold is expected to be a function of read accuracy and the reference used, and no per-experiment thresholding method is provided
- The null index (reverse of positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads, e.g. conserved genes between species
- SPUMONI operates on base-called sequences, not raw electrical signal; users must run a separate base caller upstream, and it cannot yet directly interact with the Read Until API as control software on a nanopore sequencer
- The read-level compression scheme (outputting PMLs plus mismatch characters) is described but not yet used as an aggregate statistic for read-ejection decisions

## Evidence

- [intro] SPUMONI adds a “null index” together with a hypothesis testing framework to make principled ejection choices: "SPUMONI adds a “null index” together with a hypothesis testing framework to make principled ejection choices depending on whether the observed MS lengths are longer than what would be expected by"
- [methods] The easier way would be to use the Docker/Singularity images which have SPUMONI pre-installed and ready to use: "The easier way would be to use the Docker/Singularity images which have SPUMONI pre-installed and ready to use"
- [discussion] we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets: "we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets"
- [readme] SPUMONI is a software tool for performing rapid read classifications on sequencing reads using a read's matching statistics: "SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).**"
- [intro] This optimized PML-finding procedure makes SPUMONI about 3 times faster than MONI, while achieving similar (often greater) accuracy: "This optimized PML-finding procedure makes SPUMONI about 3 times faster than MONI, while achieving similar (often greater) accuracy and allowing it to operate on streaming data"
