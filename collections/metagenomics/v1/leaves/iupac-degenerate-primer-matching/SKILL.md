---
name: iupac-degenerate-primer-matching
description: Use when raw amplicon FASTQ reads (16S/ITS/18S/28S) still carry primer
  sequences that contain degenerate IUPAC bases, and primers must be removed before
  quality filtering or OTU clustering.
when_to_use_negative:
- Reads have already been primer-trimmed and only quality filtering remains — use
  MICCA 'filter' (VSEARCH/sickle) instead
- Primers contain no ambiguous bases and a simple exact-match trim outside of Cutadapt
  is sufficient
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: MICCA
  role: Pipeline providing the 'trim' command that orchestrates primer removal
  repo: https://github.com/compmetagen/micca
- name: Cutadapt
  role: Underlying adapter/primer trimming engine; performs the IUPAC degenerate motif
    matching
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:e7d3730f19068c5f1f167dd19f2f343c579f2491549994410407dfb714a73067
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - Cutadapt
  - MICCA
  - VSEARCH
  - sickle
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# iupac-degenerate-primer-matching

## Summary

Removes primer sequences from raw amplicon reads by matching IUPAC degenerate primer motifs, as implemented by the MICCA 'trim' command, which wraps Cutadapt. This handles primers containing ambiguous bases (e.g., mixed 16S rDNA or ITS primer sets) that exact-string matching would miss.

## When to use

Use when raw amplicon FASTQ reads (16S/ITS/18S/28S) still carry primer sequences that contain degenerate IUPAC bases, and primers must be removed before quality filtering or OTU clustering.

## When NOT to use

- Reads have already been primer-trimmed and only quality filtering remains — use MICCA 'filter' (VSEARCH/sickle) instead
- Primers contain no ambiguous bases and a simple exact-match trim outside of Cutadapt is sufficient

## Inputs

- Raw amplicon sequencing reads in FASTQ format (single-end 454/Illumina/Ion Torrent or overlapping paired-end)
- Primer sequence(s) expressed with IUPAC degenerate base codes

## Outputs

- Primer-trimmed FASTQ reads (e.g., trimmed_reads.fastq) ready for the MICCA quality-filter step

## How to apply

Take the raw reads (e.g., the 454-based 16S-10 simulated dataset from ftp://ftp.fmach.it/metagenomics/micca/scirep/) and run the MICCA 'trim' command, which wraps Cutadapt (doi: 10.14806/ej.17.1.200). Supply the primer sequences in IUPAC degenerate notation so that Cutadapt matches all variants encoded by ambiguous bases (e.g., R, Y, N) in the primer motif. MICCA executes Cutadapt against the primer(s), stripping matched regions from the reads and writing primer-free reads that feed into the subsequent 'filter' step (sliding-window quality trimming, N-trimming, and min/max length filtering). Rationale: degenerate primer motifs are common in amplicon studies targeting conserved genes, and Cutadapt's IUPAC support ensures all primer variants are detected without enumerating them manually.

## Related tools

- **MICCA** (Pipeline providing the 'trim' command that orchestrates primer removal) — https://github.com/compmetagen/micca
- **Cutadapt** (Underlying adapter/primer trimming engine; performs the IUPAC degenerate motif matching)

## Evaluation signals

- No primer-derived bases remain at the 5' ends of the trimmed reads when searched with the same IUPAC motif
- Read count is preserved or reduced only by reads shorter than the allowed minimum after trimming
- Downstream MICCA 'filter' step yields filtered_reads.fastq with the expected sliding-window/length-filter survivors
- Cite Cutadapt (doi: 10.14806/ej.17.1.200) since MICCA wraps it for the 'trim' command

## Limitations

- Degenerate matching only finds primer variants representable in the supplied IUPAC motif; primers outside the motif pass through untrimmed
- Very short or heavily errored primer regions may escape detection, propagating primer contamination into quality filtering

## Evidence

- [methods] to remove primer sequences from the raw reads, exploiting Cutadapt's support for IUPAC degenerate primer motifs: [evidence span withheld — anchor status: not_found]
- [readme] Cutadapt (doi: 10.14806/ej.17.1.200) used in the ``trim`` command: "Cutadapt (doi: 10.14806/ej.17.1.200) used in the ``trim`` command"
- [readme] supports **single-end** (Roche 454, Illumina MiSeq/HiSeq ,Ion Torrent) and **overlapping paired-end** reads (Illumina MiSeq/HiSeq);: "supports **single-end** (Roche 454, Illumina MiSeq/HiSeq ,Ion Torrent) and **overlapping paired-end** reads (Illumina MiSeq/HiSeq);"
