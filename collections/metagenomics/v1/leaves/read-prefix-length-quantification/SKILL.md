---
name: read-prefix-length-quantification
description: Use when comparing adaptive/real-time nanopore signal mappers (RawHash,
  UNCALLED, Sigmap) on how early each stops sequencing a mapped read, i.e., estimating
  time and cost via read-prefix lengths and chunk counts from PAF records.
when_to_use_negative:
- When the goal is mapping accuracy or throughput benchmarking rather than sequencing
  time/cost via prefix length or chunk count — use accuracy/throughput evaluation
  instead
- When input data is r10.4.1 or non-FAST5/POD5-era signal data, since UNCALLED only
  supports legacy r9.4.1 data
- When PAF outputs lack the extended tags (e.g., chunk/stop-time tags from the UNCALLED/Sigmap
  template) needed to recover prefix length or chunk counts
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash
  role: Real-time raw nanopore signal mapper whose early-stopping behavior (maps on
    a read prefix) is quantified by average prefix length; PAF outputs parsed for
    prefix/chunk statistics
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline real-time signal mapper; its PAF template and outputs provide the
    sequenced prefix length compared against RawHash on large genomes
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Streaming raw-signal mapper baseline; its PAF outputs provide chunk counts
    used to compute the average chunks-per-read ratio vs RawHash
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:cad33da6b84a59ea758eb9d1a65dff38b225eb2c1daf451173be0c469fd6de14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# read-prefix-length-quantification

## Summary

Quantifies sequencing time/cost benefits of real-time raw-signal mappers by computing the average sequenced read prefix length (in bases) and the average number of sequenced signal chunks per mapped read, from PAF outputs of RawHash, UNCALLED, and Sigmap on r9.4.1 FAST5 datasets (D1-D5).

## When to use

Use when comparing adaptive/real-time nanopore signal mappers (RawHash, UNCALLED, Sigmap) on how early each stops sequencing a mapped read, i.e., estimating time and cost via read-prefix lengths and chunk counts from PAF records.

## When NOT to use

- When the goal is mapping accuracy or throughput benchmarking rather than sequencing time/cost via prefix length or chunk count — use accuracy/throughput evaluation instead
- When input data is r10.4.1 or non-FAST5/POD5-era signal data, since UNCALLED only supports legacy r9.4.1 data
- When PAF outputs lack the extended tags (e.g., chunk/stop-time tags from the UNCALLED/Sigmap template) needed to recover prefix length or chunk counts

## Inputs

- PAF mapping output files from RawHash on D1-D5 FAST5 datasets (test/data/d1_sars-cov-2_r94 to d5_human_na12878_r94)
- PAF mapping output files from UNCALLED on the same datasets
- PAF mapping output files from Sigmap on the same datasets
- FAST5 raw signal files for D1-D5 (r9.4.1 flowcell data)

## Outputs

- Per-dataset, per-tool average sequenced read prefix length in bases (RawHash vs UNCALLED)
- Per-dataset, per-tool average number of sequenced signal chunks per read (RawHash vs Sigmap)
- Comparison table mirroring Table 4, e.g. table4_prefix_chunk_stats.csv

## How to apply

Load the PAF mapping outputs produced by RawHash, UNCALLED, and Sigmap on the D1-D5 FAST5 datasets (d1_sars-cov-2_r94 through d5_human_na12878_r94), which follow the modified PAF template shared by UNCALLED and Sigmap. Parse each mapped-read record with the pafstats tooling to extract the sequenced read prefix length (in bases) at the time mapping stopped, since RawHash stops as soon as a read prefix maps to the reference genome. For the RawHash vs UNCALLED comparison, average the sequenced prefix length over all mapped reads per dataset and tool; for the RawHash vs Sigmap comparison, count the sequenced signal chunks consumed per mapped read and average them per dataset and tool. Assemble the per-dataset, per-tool averages into a table mirroring Table 4 (e.g., table4_prefix_chunk_stats.csv). The rationale is that shorter read prefixes and fewer chunks directly translate into lower sequencing time and cost in adaptive sampling.

## Related tools

- **RawHash** (Real-time raw nanopore signal mapper whose early-stopping behavior (maps on a read prefix) is quantified by average prefix length; PAF outputs parsed for prefix/chunk statistics) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline real-time signal mapper; its PAF template and outputs provide the sequenced prefix length compared against RawHash on large genomes) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Streaming raw-signal mapper baseline; its PAF outputs provide chunk counts used to compute the average chunks-per-read ratio vs RawHash) — https://github.com/haowenz/sigmap

## Examples

```
uncalled map -t 16 E.coli fast5_list.txt > uncalled_out.paf
```

## Evaluation signals

- Computed averages reproduce the paper's reported direction: RawHash consumes more sequenced chunks than Sigmap (about 1.58x on average) while sequencing a shorter read prefix than UNCALLED on large genomes (e.g., Green Algae, Human)
- All mapped-read records parse successfully under the modified UNCALLED/Sigmap PAF template, with valid prefix-length and chunk-count values per read
- Per-dataset averages fall in plausible ranges for each dataset (e.g., short prefixes for small viral genomes like SARS-CoV-2, larger prefixes for human NA12878)
- The output table has one row per dataset-tool pair covering all of D1-D5 for both metrics
- RawHash prefix lengths on large genomes are shorter than UNCALLED's, consistent with RawHash stopping as soon as a read prefix maps

## Limitations

- RawHash uses on average 1.58x more chunks per read than Sigmap, which can proportionally worsen its time and cost relative to Sigmap
- UNCALLED can only be applied to legacy r9.4.1 data, restricting comparisons to r9.4.1 datasets
- The chunk-based comparison is not directly comparable to the prefix-length comparison since Sigmap is not a real-time adaptive-sampling mapper in the same sense; relative time/cost conclusions are tool-pair specific

## Evidence

- [results] RawHash reduces sequencing time and cost for large genomes by using shorter read prefixes than UNCALLED, but uses on average 1.58x more chunks per read than Sigmap: [evidence span withheld — anchor status: not_found]
- [intro] RawHash stops as soon as a read prefix maps to the reference genome: "RawHash generates the hash values from raw signals and queries the hash table"
- [methods] Modified PAF template shared by UNCALLED and Sigmap: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [readme] UNCALLED r9.4.1 restriction: [evidence span withheld — anchor status: external_doc]
