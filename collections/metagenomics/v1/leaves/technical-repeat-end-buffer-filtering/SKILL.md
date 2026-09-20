---
name: technical-repeat-end-buffer-filtering
description: Use when MEMs computed from amplified metagenomic short reads (pal-MEM/E-MEM
  style output) must be filtered to remove edge-proximal technical repeats before
  ITR candidate selection.
when_to_use_negative:
- Input is already-filtered MEM output or reads without amplification bias where edge-buffer
  exclusion would discard genuine terminal signal
- Discovering insertion sequences that lack inverted terminal repeats — Palidis/pal-MEM
  is 'only equipped with discovering ISs with ITRs'
- Long-read or isolate assembly data where the 20-nt edge-buffer assumption for short
  amplified reads does not apply
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0080
tools:
- name: pal-MEM
  role: Computes maximal exact matches (modified from E-MEM) whose outputs are subjected
    to the 20-nt end-buffer exclusion filter
  repo: https://github.com/blue-moon22/pal-MEM
- name: Palidis
  role: Pipeline that runs pal-MEM (step 2) and downstream ITR candidate selection
    on the buffered MEM output
  repo: https://github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:8ebc089c29753b06179169b0f522d32792df5a4d3dc3996ee7f095ac746b8496
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/pal-MEM
  tools:
  - Palidis
  - pal-MEM
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# technical-repeat-end-buffer-filtering

## Summary

A post-MEM filtering step from the Palidis pipeline that suppresses technical repeats introduced by amplified read libraries by discarding maximal exact matches (MEMs) whose start or end falls within a 20-nucleotide (40-bit) buffer of either read edge.

## When to use

Use when MEMs computed from amplified metagenomic short reads (pal-MEM/E-MEM style output) must be filtered to remove edge-proximal technical repeats before ITR candidate selection.

## When NOT to use

- Input is already-filtered MEM output or reads without amplification bias where edge-buffer exclusion would discard genuine terminal signal
- Discovering insertion sequences that lack inverted terminal repeats — Palidis/pal-MEM is 'only equipped with discovering ISs with ITRs'
- Long-read or isolate assembly data where the 20-nt edge-buffer assumption for short amplified reads does not apply

## Inputs

- Paired-end or single-end metagenomic FASTA reads (pre-processed from FASTQ.GZ)
- pal-MEM MEM records (read identifier, MEM start, MEM end, MEM length)
- Two-bit-encoded k-mer hash table of reference/query sequences

## Outputs

- mems_end_buffered.tsv — TSV of retained MEM records (read identifier, MEM start, MEM end, length) with edge-buffer MEMs excluded

## How to apply

First encode reads with two-bit encoding (2 bits per nucleotide) and compute maximal exact matches against a k-mer hash table using pal-MEM (default k-mer length 15, minimum match length 24, maximum 100). Then apply the end-buffer filter: discard any MEM whose start or end position lies within 20 nucleotides (40 bits in the two-bit encoding) of either edge of the read, since true ITR pairs should be internal while amplified-library artefacts cluster at read ends. Retain surviving MEM records (read identifier, MEM start, MEM end, length) in a TSV such as mems_end_buffered.tsv. Validate that the file is a well-formed TSV with read and MEM-coordinate fields and that no retained MEM coordinate falls inside the 20-nt buffer at either read edge.

## Related tools

- **pal-MEM** (Computes maximal exact matches (modified from E-MEM) whose outputs are subjected to the 20-nt end-buffer exclusion filter) — https://github.com/blue-moon22/pal-MEM
- **Palidis** (Pipeline that runs pal-MEM (step 2) and downstream ITR candidate selection on the buffered MEM output) — https://github.com/blue-moon22/palidis4

## Examples

```
pal-mem -f1 example_1.fasta -f2 example_1.fasta -o example -l 30 -m 50 -k 18 -t 8
```

## Evaluation signals

- mems_end_buffered.tsv exists and parses as valid TSV with read identifier and MEM start/end/length fields present
- No retained MEM start or end coordinate falls within 20 nucleotides (40 bits) of either read edge
- Equivalent boundary condition br <= j((L-k)+1) <= er-k+1 holds for stored positions in the k-mer hash table
- Retained MEMs have lengths within the configured match-length bounds (pal-MEM defaults: minimum 24, maximum 100)
- Filtered MEM set is a strict subset of the unfiltered pal-MEM MEM output for the same reads

## Limitations

- Palidis/pal-MEM can only discover ISs with ITRs; ISs lacking ITRs are missed entirely
- ITRs shorter than the effective threshold (catalogue ITRs are 25+ nt by default; ITRs can be as short as 10 nt) are missed at default settings
- Lowering the minimum ITR length threshold or k-mer length makes the computation substantially more intensive on complex mixed microbial genomes

## Evidence

- [methods] 20-nucleotide end buffer: [evidence span withheld — anchor status: not_found]
- [readme] pal-MEM purpose: [evidence span withheld — anchor status: external_doc]
- [readme] pal-MEM options: [evidence span withheld — anchor status: external_doc]
- [discussion] Palidis scope limitation: "Palidis is currently only equipped with discovering ISs with ITRs."
