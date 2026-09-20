---
name: nucmer-whole-genome-alignment
description: Use when a set of candidate duplicate viral scaffolds (already rough-grouped,
  e.g. by Mash dist) needs exact pairwise nucleotide alignment to compute percent
  identity and percent coverage before dereplication (longest or composite mode).
when_to_use_negative:
- The goal is binning scaffolds into vMAGs rather than dereplicating them — vRhyme's
  main binning mode, not its Mash/Nucmer dereplication function, is the right tool
- Input scaffolds are already dereplicated or derive from distinct taxa with no expected
  near-identical pairs, so alignment-based duplicate detection adds cost without benefit
- Comparison should be at the protein level (e.g., redundancy checking with Mmseqs2)
  rather than nucleotide sequence similarity
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: vRhyme
  role: Hosts the dereplication function (--derep_only, --method longest/composite)
    that consumes the Nucmer alignments and applies the 0.97/0.70 or 0.99/0.20 thresholds
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Mash
  role: Sketches scaffolds (k = 31, size 1000) and forms rough groups of candidate
    duplicates so Nucmer only aligns within groups
  repo: https://github.com/marbl/mash
- name: Nucmer
  role: Performs the pairwise nucleotide whole-genome alignments used to compute sequence
    similarity and overlap fraction for dereplication decisions
  repo: https://mummer4.github.io/install/install.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:e12aa47ef77853244266f21b4afb95a7eff7a205c1a0dd77ca670d930516cd5b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - Mash
  - Nucmer
  - vRhyme
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nucmer-whole-genome-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Nucleotide-level whole-genome pairwise alignment with Nucmer as used inside vRhyme's scaffold dereplication module, where Mash-sketching forms rough duplicate groups and Nucmer alignments quantify exact sequence similarity and overlap to decide which scaffolds to merge or discard.

## When to use

Use when a set of candidate duplicate viral scaffolds (already rough-grouped, e.g. by Mash dist) needs exact pairwise nucleotide alignment to compute percent identity and percent coverage before dereplication (longest or composite mode).

## When NOT to use

- The goal is binning scaffolds into vMAGs rather than dereplicating them — vRhyme's main binning mode, not its Mash/Nucmer dereplication function, is the right tool
- Input scaffolds are already dereplicated or derive from distinct taxa with no expected near-identical pairs, so alignment-based duplicate detection adds cost without benefit
- Comparison should be at the protein level (e.g., redundancy checking with Mmseqs2) rather than nucleotide sequence similarity

## Inputs

- Nucleotide scaffold FASTA from viral prediction (e.g., VIBRANT or VirSorter output)
- Mash distance groupings of candidate duplicate scaffolds (from Mash sketch + Mash dist, k = 31, sketch size 1000)
- vRhyme dereplication configuration (--method longest or composite, --derep_only)

## Outputs

- Nucmer alignment records (delta/coords) between candidate duplicate scaffolds within each rough group
- Final representative/composite sequences written to dereplicated_scaffolds.fasta
- Log indicating convergence: no remaining Nucmer alignments between retained scaffolds

## How to apply

First sketch all input nucleotide scaffolds with Mash (default k = 31, sketch size = 1000) and run Mash dist to form rough groups of candidate duplicates. Within each group, align candidates pairwise using Nucmer with defaults -c 1000, -b 1000, -g 1000 and at most 3 fragmentations per scaffold (--nuc_split 3). For --method longest, reduce each duplicate group to its longest representative at a minimum of 0.97 sequence similarity and 0.70 overlap coverage fraction; for --method composite, dereplicate identical scaffolds and merge overlapping pairs at a minimum of 0.99 similarity and 0.20 overlap fraction, with free-ends sensitivity 0.10 and overlap sensitivity 0.25. In composite mode, iteratively merge any pairs passing the thresholds until no Nucmer alignments between remaining scaffolds are detected, then write representatives to dereplicated_scaffolds.fasta. The rationale is that sketch-level grouping is fast but approximate, so Nucmer's full alignments supply the identity/coverage evidence that dereplication decisions require.

## Related tools

- **vRhyme** (Hosts the dereplication function (--derep_only, --method longest/composite) that consumes the Nucmer alignments and applies the 0.97/0.70 or 0.99/0.20 thresholds) — https://github.com/AnantharamanLab/vRhyme
- **Mash** (Sketches scaffolds (k = 31, size 1000) and forms rough groups of candidate duplicates so Nucmer only aligns within groups) — https://github.com/marbl/mash
- **Nucmer** (Performs the pairwise nucleotide whole-genome alignments used to compute sequence similarity and overlap fraction for dereplication decisions) — https://mummer4.github.io/install/install.html

## Examples

```
nucmer -c 1000 -b 1000 -g 1000 --nuc_split 3 ref_scaffold.fna query_scaffold.fna
```

## Evaluation signals

- The output FASTA exists and parses as valid nucleotide FASTA
- No remaining scaffold pairs in dereplicated_scaffolds.fasta satisfy the configured thresholds (composite defaults 0.99 identity / 0.20 overlap; longest defaults 0.97 identity / 0.70 overlap)
- For --method longest, each Mash/Nucmer duplicate group collapses to exactly one (the longest) representative scaffold
- Nucmer ran successfully on each candidate group with the configured parameters (-c 1000, -b 1000, -g 1000, --nuc_split 3)
- The composite merging loop terminates, i.e., a pass detects no Nucmer alignments between remaining scaffolds

## Limitations

- In composite mode, merging scaffold pairs over their region of sequence overlap produces artificially chimeric representative scaffolds
- vRhyme is built to run on viral sequences; performance on whole (non-viral) metagenomes has not been fully evaluated and it is not meant to bin microbes
- Mash and Nucmer must be installed and functional (test_vRhyme.py must report Success for both) or the dereplication function cannot run

## Evidence

- [methods] vRhyme workflow card, scaffold dereplication module: [evidence span withheld — anchor status: not_found]
- [methods] vRhyme workflow card, composite dereplication behavior: [evidence span withheld — anchor status: not_found]
