---
name: six-frame-translation-of-genome-assemblies
description: Use when assembled nucleotide contigs (or metagenomic contigs) must be
  converted to protein sequences in all six frames to enumerate short ORFs (<200 aa)
  as candidate RiPP precursor peptides within putative biosynthetic gene clusters.
when_to_use_negative:
- 'When a motif-based candidate set is preferred: the all-ORF mode does not filter
  by RiPP sequence motifs and yields a much larger candidate set (e.g., 14,694 short
  ORFs across six BGCs in one genome) at the cost of a larger search space'
- When the input is unassembled short reads — MetaMiner requires reads to be first
  assembled into contigs before six-frame translation
- When the spectra of interest lack paired genomics/metagenomics data (about 99% of
  GNPS spectra), since translated ORFs can only be searched against spectra with matched
  genomic context
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaMiner
  role: Pipeline implementing the 6-frame translation, HMMer enzyme detection, 10
    kbp BGC windowing, and all-ORF candidate extraction; available as a command line
    tool and as a GNPS workflow
  repo: https://github.com/mohimanilab/MetaMiner
- name: HMMer
  role: Profile HMM search run on the six-frame translated sequences to detect RiPP
    modification enzymes that anchor the 10 kbp putative BGC windows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:47342871bbc8c6ceb02e613734953e9cf1a3ce0f434ad7b0adf4b4e0c7456fd7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - HMMer
  - MetaMiner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# six-frame-translation-of-genome-assemblies

## Summary

A candidate-generation step from MetaMiner's all-ORF genome-mining mode: assembled nucleotide contigs are translated into protein sequence in all six frames so that HMMer-detected RiPP modification enzymes can anchor 10 kbp putative biosynthetic gene cluster windows from which all short ORFs are exhaustively extracted.

## When to use

Use when assembled nucleotide contigs (or metagenomic contigs) must be converted to protein sequences in all six frames to enumerate short ORFs (<200 aa) as candidate RiPP precursor peptides within putative biosynthetic gene clusters.

## When NOT to use

- When a motif-based candidate set is preferred: the all-ORF mode does not filter by RiPP sequence motifs and yields a much larger candidate set (e.g., 14,694 short ORFs across six BGCs in one genome) at the cost of a larger search space
- When the input is unassembled short reads — MetaMiner requires reads to be first assembled into contigs before six-frame translation
- When the spectra of interest lack paired genomics/metagenomics data (about 99% of GNPS spectra), since translated ORFs can only be searched against spectra with matched genomic context

## Inputs

- Assembled nucleotide contigs (from genome or metagenome assemblies; MetaMiner first assembles short reads into contigs if the input is short-read data)
- RiPP modification-enzyme profile HMMs used with HMMer

## Outputs

- Six-frame translated protein sequences for each contig
- Detected RiPP modification enzyme hits (HMMer)
- Putative biosynthetic gene cluster windows (10 kbp, centered on each modification enzyme)
- all_orf_candidates.fasta: short ORFs (<200 aa) annotated with their parent putative BGC window

## How to apply

Load the assembled nucleotide contigs and translate each contig into protein sequence in all six frames (MetaMiner's 6-frame translation step). Run HMMer (Eddy, 2011) with RiPP modification-enzyme profile HMMs against the six-frame translated sequences to detect all modification enzymes. For each detected modification enzyme, construct a 10 kbp window centered on the enzyme hit and label it a putative biosynthetic gene cluster. Within each window, extract all open reading frames from the six-frame translation shorter than a configurable length threshold (default 200 amino acids), then write the candidates annotated with their parent BGC window to all_orf_candidates.fasta. The rationale is exhaustiveness: unlike the motif-ORF strategy, the all-ORF approach does not filter by RiPP sequence motifs, so no precursor resembling a known RiPP is missed.

## Related tools

- **MetaMiner** (Pipeline implementing the 6-frame translation, HMMer enzyme detection, 10 kbp BGC windowing, and all-ORF candidate extraction; available as a command line tool and as a GNPS workflow) — https://github.com/mohimanilab/MetaMiner
- **HMMer** (Profile HMM search run on the six-frame translated sequences to detect RiPP modification enzymes that anchor the 10 kbp putative BGC windows)

## Evaluation signals

- all_orf_candidates.fasta exists and is valid FASTA
- Every reported ORF sequence length is below the 200 amino acid threshold (or the configured threshold)
- Each extracted ORF maps within a 10 kbp window centered on a HMMer-detected modification enzyme
- Translation covers all six reading frames of every input contig
- Candidate counts are consistent with exhaustive extraction (thousands of short ORFs across BGCs, e.g., 14,694 short ORFs across six BGCs in one genome) rather than motif-filtered subsets

## Limitations

- The all-ORF approach does not filter by RiPP sequence motifs, producing a much larger search space (e.g., 14,694 short ORFs across six BGCs in one genome)
- The ORF length threshold is configurable (default 200 amino acids), so results depend on the chosen cutoff
- Short-read input must be assembled into contigs before six-frame translation; unassembled reads cannot be used directly
- Only about 1% of GNPS spectra are accompanied by genomics/metagenomics data, so most translated-ORF candidate databases cannot be searched against public spectra

## Evidence
