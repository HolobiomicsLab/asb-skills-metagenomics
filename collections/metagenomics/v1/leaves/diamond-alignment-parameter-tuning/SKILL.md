---
name: diamond-alignment-parameter-tuning
description: Use when DIAMOND-based retrieval of phylogenetic markers yields low query
  coverage, particularly with low-coverage read data as query, and masking or sensitivity
  settings may be limiting alignments.
when_to_use_negative:
- Query coverage is already high (e.g. the high-quality-assembly benchmark reached
  92.2% mean query coverage), so masking has little room to help
- The analysis goal is maximizing percent identity only; disabling tantan can slightly
  reduce identity while raising coverage
- The aligner or pipeline does not expose DIAMOND's tantan masking option (the skill
  is specific to DIAMOND-driven searches)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: Patchwork
  role: Alignment-based marker retrieval tool whose DIAMOND search parameters (--masking,
    --ultra-sensitive, --frameshift) are being tuned; accepts assemblies or raw reads
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Sequence aligner performing the BLASTX search; its tantan masking is toggled
    via --masking 0 to change query coverage
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Implementation language of Patchwork in which the parameter comparison workflow
    runs
- name: SPAdes
  role: Produces the de novo assembly of short reads used as the query when tuning
    parameters on assembly data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:b0061cd40bf6ca2b75fd33798229c70c0302d9271869efe60146b4e4f375f914
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  - SPAdes
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# diamond-alignment-parameter-tuning

## Summary

Tunes DIAMOND alignment parameters used inside alignment-based marker retrieval (e.g. Patchwork), specifically disabling tantan masking with --masking 0 to improve query coverage of recovered markers, especially for read-based queries.

## When to use

Use when DIAMOND-based retrieval of phylogenetic markers yields low query coverage, particularly with low-coverage read data as query, and masking or sensitivity settings may be limiting alignments.

## When NOT to use

- Query coverage is already high (e.g. the high-quality-assembly benchmark reached 92.2% mean query coverage), so masking has little room to help
- The analysis goal is maximizing percent identity only; disabling tantan can slightly reduce identity while raising coverage
- The aligner or pipeline does not expose DIAMOND's tantan masking option (the skill is specific to DIAMOND-driven searches)

## Inputs

- Query: assembled contigs (e.g. SPAdes de novo assembly of Illumina short reads, K-mer 55) or raw WGS reads (e.g. D. gyrociliatus Illumina reads, PRJEB37657)
- Reference: amino-acid marker sequences (e.g. 815 D. gyrociliatus USCOs from GenBank GCA_904063045.1)
- Baseline DIAMOND/Patchwork configuration to diff against

## Outputs

- Pairwise comparison of percent identity and query coverage for masking-enabled vs masking-disabled runs
- Recovered marker sequences per configuration, evaluated against the reference

## How to apply

Run the alignment-based retrieval tool (Patchwork, which drives DIAMOND BLASTX) under its baseline settings — in the paper: --ultra-sensitive --frameshift 15 with default BLOSUM62 scoring (gap open 11, gap extension 1) and tantan masking enabled. Then repeat the identical run with the sole change of adding --masking 0 to disable DIAMOND's tantan masking. Evaluate each run against the reference markers by computing percent identity (exact matches in all aligned positions) and total aligned positions (query coverage), and tabulate the two runs side by side. Accept the masking-disabled configuration when it shows higher query coverage with only a slight decrease in percent identity, as observed for D. gyrociliatus read-based queries against USCO references.

## Related tools

- **Patchwork** (Alignment-based marker retrieval tool whose DIAMOND search parameters (--masking, --ultra-sensitive, --frameshift) are being tuned; accepts assemblies or raw reads) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Sequence aligner performing the BLASTX search; its tantan masking is toggled via --masking 0 to change query coverage) — https://github.com/bbuchfink/diamond
- **Julia** (Implementation language of Patchwork in which the parameter comparison workflow runs)
- **SPAdes** (Produces the de novo assembly of short reads used as the query when tuning parameters on assembly data)

## Examples

```
patchwork --query assembly_k55.fasta --reference uscos.fasta --ultra-sensitive --frameshift 15 --masking 0
```

## Evaluation signals

- Masking-disabled run must show higher query coverage (total aligned positions) than the masking-enabled run
- Percent identity in the masking-disabled run decreases only slightly relative to the baseline
- Improvement should be most pronounced for read-based (as opposed to assembly-based) queries
- Each run is scored by computing exact matches over all aligned positions (percent identity) and total aligned positions (query coverage) against the reference markers
- Same sensitivity settings (--ultra-sensitive --frameshift 15, BLOSUM62, gap open 11 / gap extend 1) must be held constant between the compared runs so masking is the only variable

## Limitations

- Disabling tantan masking trades a small decrease in percent identity for the query-coverage gain
- Benefit is mainly observed for read-based data; assembly-based queries may see smaller gains
- No concrete numeric cutoffs are given for when the coverage gain justifies the identity loss (the article notes cutoff choices are easiest near 100% identity)

## Evidence

- [results] query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument): "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [results] On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%: "On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%"
- [results] most markers were well above the 90% aligned position trimming threshold: "most markers were well above the 90% aligned position trimming threshold"
- [readme] Works with already assembled contigs _or_ raw reads: "Works with already assembled contigs _or_ raw reads"
