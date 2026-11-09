---
name: hotspot-bed-configuration-parsing
description: 'Use when preparing inputs for Nano2NGS-Muta: a hotspot panel must be
  encoded as a BED-style configure file plus a complete check-hotspot.list before
  running nano2ngs-muta.pl on nanopore FASTQ reads.'
when_to_use_negative:
- If the input is an NGS short-read dataset rather than nanopore long reads — the
  configure/check files exist only to seed extraction from long reads.
- If the goal is metagenomic classification (Nano2NGS-Meta), which uses -fq/-read_len/-time
  and has no hotspot configure file.
- If mutations of interest are genome-wide rather than a hotspot panel; the method
  is described as particularly suitable for hotspot panel mutation detection for clinical
  diagnosis.
tools:
- name: Nano2NGS-Muta
  role: Consumes the parsed hotspot BED configure file via -configure and the check-hotspot.list
    to extract NGS-liked short reads from nanopore FASTQ and call final target hotspot
    mutations
  repo: https://github.com/langjidong/Nano2NGS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac033
    title: 'Nano2NGS-Muta: a framework for converting nanopore sequencing data to
      NGS-liked sequencing data for hotspot mutation detection'
  source_content_hash: sha256:fd4fcf7ce5918441a292c05add8b4dfa3bebb239d3838534c971b698df7ddf48
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/langjidong/Nano2NGS
  repo_url: https://github.com/langjidong/Nano2NGS
  tools:
  - GATK Best Practice pipeline
  - Nano2NGS-Muta
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# hotspot-bed-configuration-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse the hotspot BED configure file (chr, start, end, gene symbol, mutation, optional transcription ID) and the companion check-hotspot.list that Nano2NGS-Muta requires to seed short-read extraction and final hotspot mutation calling from nanopore FASTQ data.

## When to use

Use when preparing inputs for Nano2NGS-Muta: a hotspot panel must be encoded as a BED-style configure file plus a complete check-hotspot.list before running nano2ngs-muta.pl on nanopore FASTQ reads.

## When NOT to use

- If the input is an NGS short-read dataset rather than nanopore long reads — the configure/check files exist only to seed extraction from long reads.
- If the goal is metagenomic classification (Nano2NGS-Meta), which uses -fq/-read_len/-time and has no hotspot configure file.
- If mutations of interest are genome-wide rather than a hotspot panel; the method is described as particularly suitable for hotspot panel mutation detection for clinical diagnosis.

## Inputs

- Hotspot BED configure file (tab-separated: chr, start, end, gene symbol, mutation, optional transcription ID)
- check-hotspot.list file containing ALL mutation positions (same format as the hotspot BED file)
- Nanopore FASTQ(.gz) read file

## Outputs

- Parsed hotspot site table used to seed paired seed sequence extraction (L-length windows cut into N short-read sets)
- Complete mutation-position checklist driving final hotspot_mutation_calls.txt

## How to apply

Read the hotspot BED configure file whose fields are chr, start, end, gene symbol, mutation, and an optional transcription ID, and pass it via the -configure option of nano2ngs-muta.pl; the step size for each hotspot site on the NGS-liked reads is set by -position_step (default 10). Because one probe may span two or more mutations, the configure file must keep only the minimal position, so other mutations on that probe may be lost from extraction. To recover them, also prepare a check-hotspot.list file — formatted just like the hotspot BED file — that contains ALL mutation positions to be evaluated during final target hotspot mutation calling. Parse both files before read extraction so that every final target hotspot mutation can be checked against the complete list and the per-position weights used in the weighted-sum integration can be assigned.

## Related tools

- **Nano2NGS-Muta** (Consumes the parsed hotspot BED configure file via -configure and the check-hotspot.list to extract NGS-liked short reads from nanopore FASTQ and call final target hotspot mutations) — https://github.com/langjidong/Nano2NGS

## Examples

```
perl nano2ngs-muta.pl -fq nanopore.fq -read_len 101 -position_step 10 -configure Hotspot.bed -process 8
```

## Evaluation signals

- Each BED line splits into the expected chr/start/end/gene symbol/mutation (plus optional transcription ID) fields separated by tabs.
- Every mutation position in the panel appears in check-hotspot.list; any position dropped from the configure file by minimal-position truncation is still present in the checklist.
- Final calls in hotspot_mutation_calls.txt are validated against the complete check-hotspot.list, not just the truncated configure file.
- The weighted-sum integration over the N short-read sets uses position-based weights that sum to 1, and results not detected across extracted datasets are filtered out.

## Limitations

- When one probe contains two or more mutations, the configure file keeps only the minimal position and other mutations may be lost; they must be recovered via check-hotspot.list.
- The framework performs no error correction on reads or bases, so converted short reads may contain similar error rates to the original nanopore reads; LoD analysis and background noise evaluation with actual data are required.
- Hotspot placement and read-length selection affect performance; the authors note selection of read length and hotspot positions as open problems.

## Evidence

- [readme] Hotspot bed file, the format likes: chr	start	end	gene symbol	mutation	transcription ID (optional): "-configure	<Hotspot BED File>		Hotspot bed file, the format likes: chr\tstart\tend\tgene symbol\tmutation\ttranscription ID (optional)"
- [readme] Because one probe may span multiple mutations, the configure file keeps only the minimal position; check-hotspot.list must contain all mutation positions.: "Because one probe may contain two or more mutations, the configure file must choose the minimal position, others may be loss. So the check file should contail all the mutation positions."
- [methods] You should prepare a check-hotspot.list file for calling final target hotspot mutation.: "You should prepare a check-hotspot.list file for calling final target hotspot mutation"
- [other] The configure file keeps only the minimal position when one probe spans multiple mutations.: [evidence span withheld — anchor status: not_found]
