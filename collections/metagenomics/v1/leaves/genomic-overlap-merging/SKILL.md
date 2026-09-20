---
name: genomic-overlap-merging
description: Use when a viral scaffold FASTA may contain redundant/overlapping sequences
  from fragmented genome assemblies and a dereplicated representative set is required
  before binning.
when_to_use_negative:
- The goal is binning whole metagenomes or microbes — vRhyme is built for viral sequences/scaffolds
  and 'is not meant to bin microbes'.
- Preservation of each original scaffold sequence is required — the composite method
  produces chimeric merged representatives rather than selecting existing sequences.
- Scaffolds are already dereplicated or come from complete circular genomes flagged
  for removal pre-binning.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: vRhyme
  role: Runs the dereplication function (--derep_only, --method longest/composite)
    that orchestrates grouping and merging
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Mash
  role: Sketches scaffolds (k=31, size 1000) and forms rough candidate duplicate groups
    via Mash dist
  repo: https://github.com/marbl/mash
- name: Nucmer
  role: Aligns candidate duplicates within groups (default -c 1000, -b 1000, -g 1000,
    --nuc_split 3) to quantify similarity and overlap for merging decisions
  repo: https://mummer4.github.io/install/install.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:3c6c75fb48527c61de04968203d554bc1931ecd7131002d61ccb9e0cb8b41ab9
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# genomic-overlap-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

vRhyme's scaffold dereplication module (Mash + Nucmer) that collapses duplicate viral scaffolds either by keeping the longest representative or by merging overlapping duplicates into a single composite sequence, writing dereplicated_scaffolds.fasta.

## When to use

Use when a viral scaffold FASTA may contain redundant/overlapping sequences from fragmented genome assemblies and a dereplicated representative set is required before binning.

## When NOT to use

- The goal is binning whole metagenomes or microbes — vRhyme is built for viral sequences/scaffolds and 'is not meant to bin microbes'.
- Preservation of each original scaffold sequence is required — the composite method produces chimeric merged representatives rather than selecting existing sequences.
- Scaffolds are already dereplicated or come from complete circular genomes flagged for removal pre-binning.

## Inputs

- Nucleotide scaffold FASTA (e.g., predicted viral scaffolds from VIBRANT or VirSorter)
- Choice of dereplication method flag (--method longest or --method composite)

## Outputs

- dereplicated_scaffolds.fasta containing final representative or composite scaffold sequences

## How to apply

Run vRhyme with --derep_only so binning is skipped and only dereplication executes on the input nucleotide scaffold FASTA. Sketch all scaffolds with Mash (default k=31, sketch size 1000) and use Mash dist to form rough groups of candidate duplicates. Align candidates within each group with Nucmer using defaults -c 1000, -b 1000, -g 1000 and at most 3 fragmentations per scaffold (--nuc_split 3). Choose --method longest to reduce each group to its longest representative at minimum 0.97 sequence similarity and 0.70 overlap coverage fraction, or --method composite to dereplicate identical scaffolds and merge overlapping pairs at minimum 0.99 similarity and 0.20 overlap fraction (free-ends sensitivity 0.10, overlap sensitivity 0.25). The composite method iteratively merges passing pairs over their region of sequence overlap until no Nucmer alignments remain between scaffolds, which can produce artificially chimeric representative scaffolds, so use it when collapsing fragmented duplicates outweighs chimera risk.

## Related tools

- **vRhyme** (Runs the dereplication function (--derep_only, --method longest/composite) that orchestrates grouping and merging) — https://github.com/AnantharamanLab/vRhyme
- **Mash** (Sketches scaffolds (k=31, size 1000) and forms rough candidate duplicate groups via Mash dist) — https://github.com/marbl/mash
- **Nucmer** (Aligns candidate duplicates within groups (default -c 1000, -b 1000, -g 1000, --nuc_split 3) to quantify similarity and overlap for merging decisions) — https://mummer4.github.io/install/install.html

## Examples

```
vRhyme -i scaffolds.fasta -o derep_out --derep_only --method composite
```

## Evaluation signals

- The output FASTA exists and parses as valid nucleotide FASTA.
- No remaining scaffold pairs in the output satisfy the configured identity and overlap thresholds (composite defaults 0.99/0.20; longest defaults 0.97/0.70).
- The vRhyme installation test reports 'Success' for both Mash and Nucmer, which are required for the dereplication function.
- For --method longest, each duplicate group reduces to a single longest representative at >=0.97 similarity and >=0.70 overlap coverage.

## Limitations

- In composite dereplication mode, vRhyme joins scaffold pairs passing the identity/coverage thresholds over their region of sequence overlap, producing artificially chimeric representative scaffolds.
- Nucmer alignments allow a maximum of 3 fragmentations per scaffold (--nuc_split 3), so highly fragmented duplicates beyond this limit may be missed.
- vRhyme is built to run on viral sequences/scaffolds; performance on whole metagenomes has not been fully evaluated and it is not meant to bin microbes.

## Evidence

- [other] In composite dereplication mode, vRhyme joins scaffold pairs that meet the percent identity and percent coverage thresholds over their region of sequence overlap, producing artificially chimeric representative scaffolds.: [evidence span withheld — anchor status: not_found]
