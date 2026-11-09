---
name: open-reading-frame-extraction
description: Use when you have assembled nucleotide contigs and need to enumerate
  every candidate short-ORF RiPP precursor inside modification-enzyme-flanked BGC
  windows, without relying on known RiPP sequence motifs.
when_to_use_negative:
- When known RiPP sequence motifs can constrain candidates — the motif-ORF strategy
  gives a smaller search space; all-ORF is the fallback when motif filtering would
  miss novel precursors
- When the goal is full-length protein or gene annotation rather than short (<200
  aa) candidate RiPP precursor peptides within BGC windows
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: HMMer
  role: Scans six-frame translated contigs with RiPP modification-enzyme profile HMMs
    to locate enzyme hits that define the 10 kbp BGC windows from which ORFs are extracted
  repo: https://github.com/mohimanilab/MetaMiner
- name: MetaMiner
  role: Implements the all-ORF precursor candidate generation as part of its peptidogenomics
    RiPP discovery pipeline
  repo: https://github.com/mohimanilab/MetaMiner
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:fcd711657d7e6ec3a3b36ffb8a19fabd733404204c92bf6c6eff150eeb86d160
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - HMMer
  - MetaMiner
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# open-reading-frame-extraction

## Summary

MetaMiner's all-ORF genome-mining strategy exhaustively extracts short open reading frames (default <200 amino acids) from six-frame translated contigs within 10 kbp putative biosynthetic gene cluster windows, maximizing candidate RiPP precursor coverage at the cost of a larger search space.

## When to use

Use when you have assembled nucleotide contigs and need to enumerate every candidate short-ORF RiPP precursor inside modification-enzyme-flanked BGC windows, without relying on known RiPP sequence motifs.

## When NOT to use

- When known RiPP sequence motifs can constrain candidates — the motif-ORF strategy gives a smaller search space; all-ORF is the fallback when motif filtering would miss novel precursors
- When the goal is full-length protein or gene annotation rather than short (<200 aa) candidate RiPP precursor peptides within BGC windows

## Inputs

- Assembled nucleotide contigs (from short-read assembly or provided genome assemblies)
- RiPP modification-enzyme profile HMMs for use with HMMer
- ORF length threshold parameter (default: 200 amino acids)

## Outputs

- all_orf_candidates.fasta: candidate ORF sequences annotated with their parent putative BGC window

## How to apply

Translate each assembled nucleotide contig into protein sequence in all six frames, then run HMMer (Eddy, 2011) with RiPP modification-enzyme profile HMMs against the six-frame translations to detect all modification enzymes. For each detected enzyme hit, construct a 10 kbp window centered on the hit and label it a putative biosynthetic gene cluster. Within each window, extract all ORFs from the six-frame translation that are shorter than a configurable length threshold (default 200 amino acids), writing the annotated candidates to all_orf_candidates.fasta. Unlike the motif-ORF strategy, this all-ORF approach applies no RiPP sequence-motif filter, so it yields a much larger candidate set (e.g., 14,694 short ORFs across six BGCs in one genome) and trades search space for sensitivity to novel precursors that lack similarity to known RiPPs.

## Related tools

- **HMMer** (Scans six-frame translated contigs with RiPP modification-enzyme profile HMMs to locate enzyme hits that define the 10 kbp BGC windows from which ORFs are extracted) — https://github.com/mohimanilab/MetaMiner
- **MetaMiner** (Implements the all-ORF precursor candidate generation as part of its peptidogenomics RiPP discovery pipeline) — https://github.com/mohimanilab/MetaMiner

## Evaluation signals

- Verify that all_orf_candidates.fasta exists and is valid FASTA
- Check that every reported ORF sequence length is below the configured threshold (default 200 amino acids)
- Confirm each ORF maps within a 10 kbp window centered on a HMMer-detected modification enzyme hit
- Compare candidate counts against expectations: an all-ORF run should produce a much larger set than motif-ORF (e.g., 14,694 short ORFs across six BGCs in one genome)

## Limitations

- The all-ORF strategy does not filter by RiPP sequence motifs, producing a very large candidate search space that increases downstream scoring burden
- ORFs shorter than the threshold but outside a 10 kbp modification-enzyme window are not considered candidates
- The motif-ORF alternative misses peptides without sequence similarity to known RiPPs, which is the motivation for — and burden of — this exhaustive approach

## Evidence
