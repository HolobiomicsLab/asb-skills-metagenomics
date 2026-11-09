---
name: nanopore-long-read-to-short-read-conversion
description: Use when a nanopore FASTQ (e.g., .fq.gz from ONT) must be converted into
  NGS-liked short-read sets to run conventional NGS pipelines for hotspot mutation
  detection in a hotspot panel.
when_to_use_negative:
- Input is already short-read NGS FASTQ data — no long-to-short conversion is needed
  and the extraction would discard information
- The goal is metagenomic taxonomical classification — the authors route that use
  case to the separate Nano2NGS-Meta module (nano2ngs-meta.pl)
- Reads/bases require explicit error correction before analysis — the current Nano2NGS
  version only converts long reads into short reads with no error correction for reads
  and/or bases
- Target variant types are copy number variation, structural variation, gene fusion,
  or gene expression — these are only planned future algorithm development for Nano2NGS
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: Nano2NGS-Muta
  role: nano2ngs-muta.pl converts the nanopore long reads into N NGS-liked short-read
    sets via paired seed sequences and zero-mismatch extraction, then supports weighted
    integration of per-set hotspot calls
  repo: https://github.com/langjidong/Nano2NGS
- name: GATK Best Practice pipeline
  role: conventional NGS analysis pipeline applied to each of the N extracted short-read
    sets to call hotspot mutations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac033
    title: 'Nano2NGS-Muta: a framework for converting nanopore sequencing data to
      NGS-liked sequencing data for hotspot mutation detection'
  source_content_hash: sha256:36bbf85b6bc5089ff26029b4318053acfdb8133ac275f58eb6499338ebe71dd7
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
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# nanopore-long-read-to-short-read-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts Oxford Nanopore long reads into NGS-liked short-read sets using Nano2NGS-Muta's paired-seed extraction, so standard NGS pipelines (e.g., GATK Best Practice) can detect hotspot mutations while statistical weighting controls nanopore-specific non-random errors and indels.

## When to use

Use when a nanopore FASTQ (e.g., .fq.gz from ONT) must be converted into NGS-liked short-read sets to run conventional NGS pipelines for hotspot mutation detection in a hotspot panel.

## When NOT to use

- Input is already short-read NGS FASTQ data — no long-to-short conversion is needed and the extraction would discard information
- The goal is metagenomic taxonomical classification — the authors route that use case to the separate Nano2NGS-Meta module (nano2ngs-meta.pl)
- Reads/bases require explicit error correction before analysis — the current Nano2NGS version only converts long reads into short reads with no error correction for reads and/or bases
- Target variant types are copy number variation, structural variation, gene fusion, or gene expression — these are only planned future algorithm development for Nano2NGS

## Inputs

- Nanopore sequencing FASTQ file (.fq or .fq.gz)
- Hotspot BED configure file (chr, start, end, gene symbol, mutation, optional transcription ID)
- check-hotspot.list file containing all mutation positions
- L-length reference windows used to derive the N paired seed sequences

## Outputs

- N NGS-liked short-read sets extracted from the nanopore reads
- Per-short-read-set hotspot detection results from the NGS pipeline
- hotspot_mutation_calls.txt with final target hotspot mutation calls

## How to apply

Run nano2ngs-muta.pl with -fq for the nanopore FASTQ input, -read_len 101 for the NGS-liked read length, -position_step 10 as the step size per hotspot site, -configure pointing to the hotspot BED file (chr, start, end, gene symbol, mutation, optional transcription ID), and -process to parallelize (default 1; the paper's example uses 8). The tool derives N paired seed sequences from L-length reference windows that exclude hotspot positions and extracts short reads from original nanopore reads with zero mismatch, yielding N NGS-liked short-read sets. Because one probe may span multiple mutations and the configure file keeps only the minimal position, also prepare a check-hotspot.list containing ALL mutation positions so none are lost from final calling. Apply the NGS mutation-analysis pipeline (e.g., GATK Best Practice) to each of the N short-read sets, then integrate per-set detections with a weighted-sum equation using position-based weights that sum to 1, filtering out results not detected across the extracted datasets. Write the final calls to hotspot_mutation_calls.txt and validate every target hotspot mutation against the complete check-hotspot.list.

## Related tools

- **Nano2NGS-Muta** (nano2ngs-muta.pl converts the nanopore long reads into N NGS-liked short-read sets via paired seed sequences and zero-mismatch extraction, then supports weighted integration of per-set hotspot calls) — https://github.com/langjidong/Nano2NGS
- **GATK Best Practice pipeline** (conventional NGS analysis pipeline applied to each of the N extracted short-read sets to call hotspot mutations)

## Examples

```
perl nano2ngs-muta.pl -fq nanopore.fq -read_len 101 -position_step 10 -configure Hotspot.bed -process 8
```

## Evaluation signals

- The N paired seed sequences must exclude hotspot positions and cut L-length windows from the long reads with zero mismatch, yielding N short-read sets
- Each converted short read has length equal to -read_len (e.g., 101 bp), extracted at -position_step increments (e.g., 10) around each hotspot site
- Integration weights over the N short-read sets must sum to 1, and results not detected across the extracted datasets are filtered out
- Every final target hotspot mutation in hotspot_mutation_calls.txt must be checked against the complete check-hotspot.list, including mutations lost from the configure BED by minimal-position truncation
- Hotspot BED rows keep only the minimal position when one probe spans multiple mutations, so the check-hotspot.list must contain more positions than the BED in those cases

## Limitations

- The current Nano2NGS version performs no error correction for reads and/or bases, so converted short reads may contain lower or similar error rates to the long reads
- Regardless of the analysis pipeline used, limit of detection (LoD) analysis and background noise evaluation with actual data are necessary, as different algorithms produce different result features
- Analyzing each of the N sampling datasets increases analysis time and computing-resource consumption; selection of read length, hotspot position, and software efficiency remain open problems
- Read-length choice is an explicit limitation of Nano2NGS-Muta reported in the discussion

## Evidence

- [abstract] Long reads can be converted into short reads and then processed through existing NGS analysis pipelines in combination with statistical methods for hotspot mutation detection.: "Long reads can be converted into short reads and then processed through existing NGS analysis pipelines in combination with statistical methods for hotspot mutation detection."
- [methods] only converts long reads into a set of short reads: "only converts long reads into a set of short reads"
- [readme] The step size for hotspot site on the NGS-Liked reads, default is 10: "-position_step  <Step Size>            The step size for hotspot site on the NGS-Liked reads, default is 10"
- [readme] Because one probe may contain two or more mutations, the configure file must choose the minimal position, others may be loss.: "Because one probe may contain two or more mutations, the configure file must choose the minimal position, others may be loss."
- [readme] You should prepare a check-hotspot.list file for calling final target hotspot mutation.: "Note: You should prepare a check-hotspot.list file for calling final target hotspot mutation."
