---
name: scaffold-dereplication
description: Use when a viral scaffold FASTA contains redundant/duplicate sequences
  (e.g., multiple assemblies or fragmented genomes) and you need to collapse them
  pre-binning or investigate how vRhyme's composite merging joins scaffold pairs.
when_to_use_negative:
- Input is a whole (unfiltered) metagenome — vRhyme's performance on entire metagenomes
  has not been fully evaluated and it is built for viral scaffolds
- The goal is microbial binning — vRhyme is explicitly 'not meant to bin microbes'
- Composite mode output must be non-chimeric for downstream taxonomy — composite merging
  can produce artificially chimeric representative scaffolds
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: vRhyme
  role: Orchestrates dereplication (--derep_only, --method longest/composite, --derep_id,
    --frac, -m minimum length); writes dereplicated_scaffolds.fasta
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Mash
  role: Sketches scaffolds (k=31, size=1000) and computes pairwise distances (mash
    dist) to form rough groups of candidate duplicates
  repo: https://github.com/marbl/mash
- name: Nucmer
  role: Nucleotide alignment of candidate duplicates (-c 1000, -b 1000, -g 1000, --nuc_split
    3) to compute sequence similarity and overlap fractions used for threshold decisions
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:ebe832a5c28a055552bddf6f11201ea59d3f2b0408877740d3a3a5db5ef859c5
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
  - BWA
  - Bowtie2
  - Mash
  - Mmseqs2
  - Nucmer
  - Prodigal
  - Python
  - Samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# scaffold-dereplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Standalone scaffold dereplication in vRhyme using Mash sketching for rough duplicate grouping followed by Nucmer alignment and either 'longest' (keep one representative) or 'composite' (merge overlapping pairs) modes, optionally with --derep_only to skip binning entirely.

## When to use

Use when a viral scaffold FASTA contains redundant/duplicate sequences (e.g., multiple assemblies or fragmented genomes) and you need to collapse them pre-binning or investigate how vRhyme's composite merging joins scaffold pairs.

## When NOT to use

- Input is a whole (unfiltered) metagenome — vRhyme's performance on entire metagenomes has not been fully evaluated and it is built for viral scaffolds
- The goal is microbial binning — vRhyme is explicitly 'not meant to bin microbes'
- Composite mode output must be non-chimeric for downstream taxonomy — composite merging can produce artificially chimeric representative scaffolds

## Inputs

- Nucleotide scaffold FASTA (predicted viral scaffolds, optionally length-filtered at 2000bp minimum)
- Optional: pre-circularized/complete viral genome FASTAs already removed via flag_circular.py

## Outputs

- dereplicated_scaffolds.fasta (final representative or composite sequences)
- Dereplication log/table showing which scaffolds were merged or reduced

## How to apply

Load the input nucleotide scaffold FASTA into vRhyme with --derep_only so only the dereplication function executes and binning is skipped. Sketch all scaffolds with Mash (default k = 31, sketch size = 1000) and use Mash dist to form rough groups of candidate duplicates, then align candidates within each group using Nucmer with defaults -c 1000, -b 1000, -g 1000 and --nuc_split 3 (max 3 fragmentations per scaffold). Choose --method longest to reduce each group to its longest representative at a minimum of 0.97 sequence similarity and 0.70 overlap coverage fraction, or --method composite to dereplicate identical scaffolds and merge overlapping pairs at a minimum of 0.99 similarity and 0.20 overlap fraction (free-ends sensitivity 0.10, overlap sensitivity 0.25), iterating merges until no Nucmer alignments remain between scaffolds. Note that composite mode joins pairs passing the thresholds over their region of sequence overlap, which can produce artificially chimeric representative scaffolds — a key caveat when interpreting output.

## Related tools

- **vRhyme** (Orchestrates dereplication (--derep_only, --method longest/composite, --derep_id, --frac, -m minimum length); writes dereplicated_scaffolds.fasta) — https://github.com/AnantharamanLab/vRhyme
- **Mash** (Sketches scaffolds (k=31, size=1000) and computes pairwise distances (mash dist) to form rough groups of candidate duplicates) — https://github.com/marbl/mash
- **Nucmer** (Nucleotide alignment of candidate duplicates (-c 1000, -b 1000, -g 1000, --nuc_split 3) to compute sequence similarity and overlap fractions used for threshold decisions)

## Examples

```
vRhyme -i scaffolds.fna --derep_only --method composite -m 1000 --derep_id 0.99 --frac 0.20
```

## Evaluation signals

- Output FASTA dereplicated_scaffolds.fasta exists and parses as valid nucleotide FASTA
- No remaining scaffold pairs in the output satisfy the configured thresholds (composite defaults 0.99 identity / 0.20 overlap; longest defaults 0.97 / 0.70)
- test_vRhyme.py reports Mash and Nucmer as 'Success' since both are required for the dereplication function
- In longest mode, each duplicate group collapses to a single longest representative; in composite mode, iteration stops only when no Nucmer alignments are detected between remaining scaffolds

## Limitations

- Composite dereplication joins threshold-passing scaffold pairs over their region of sequence overlap, producing artificially chimeric representative scaffolds
- Dereplication with --derep_id 0.97 / --frac 0.70 (longest) versus 0.99 / 0.20 (composite) yields very different collapse behavior; thresholds must match the analysis intent
- Mash and Nucmer must be installed and testable; without them the dereplication function cannot run

## Evidence

- [methods] Sketch all scaffolds with Mash (default k = 31, sketch size = 1000) and use Mash dist to form rough groups of candidate duplicates: [evidence span withheld — anchor status: not_found]
