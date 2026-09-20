---
name: modification-enzyme-hmm-detection
description: 'Use when assembled contigs or RefSeq genomes paired with tandem mass
  spectra must be mined for RiPPs: after 6-frame translation, modification-enzyme
  HMM hits define which 10 kbp windows feed motif-ORF/all-ORF precursor prediction
  in MetaMiner.'
when_to_use_negative:
- The spectra under study are not accompanied by genomics/metagenomics data — as with
  the other 99% of GNPS spectra — so enzyme-based genome mining cannot link them to
  candidate RiPPs.
- The genome sequence of the RiPP producer is unavailable (e.g., the lacticin producer);
  in that case spectra must instead be searched against previously described biosynthetic
  gene clusters rather than HMM-mined from a genome.
- The target natural product is a non-ribosomal peptide or unknown compound class
  outside the RiPP modification-enzyme HMM set — Dereplicator+/non-RiPP spectral family
  analysis is the wrong path here for precursor construction.
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3172
tools:
- name: HMMer
  role: Runs profile HMM searches on the 6-frame translated sequences to detect all
    RiPP modification enzymes that anchor putative BGC windows
- name: MetaMiner
  role: Wraps the 6-frame translation, HMMer enzyme detection, 10 kbp BGC window construction,
    and motif-ORF/all-ORF precursor database building into one peptidogenomics command-line
    pipeline
  repo: https://github.com/mohimanilab/MetaMiner
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:c27605cc5318d5eca1113eec503199a9aa28d4899336565b371b3d14c576eb91
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - HMMer
  - MS-DPR
  - MetaMiner
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# modification-enzyme-hmm-detection

## Summary

Scans 6-frame translated genomic or metagenomic contigs with HMMer profile HMMs to detect RiPP modification enzymes, delimiting each hit as a 10 kbp putative biosynthetic gene cluster (BGC) window from which candidate precursor peptides are enumerated.

## When to use

Use when assembled contigs or RefSeq genomes paired with tandem mass spectra must be mined for RiPPs: after 6-frame translation, modification-enzyme HMM hits define which 10 kbp windows feed motif-ORF/all-ORF precursor prediction in MetaMiner.

## When NOT to use

- The spectra under study are not accompanied by genomics/metagenomics data — as with the other 99% of GNPS spectra — so enzyme-based genome mining cannot link them to candidate RiPPs.
- The genome sequence of the RiPP producer is unavailable (e.g., the lacticin producer); in that case spectra must instead be searched against previously described biosynthetic gene clusters rather than HMM-mined from a genome.
- The target natural product is a non-ribosomal peptide or unknown compound class outside the RiPP modification-enzyme HMM set — Dereplicator+/non-RiPP spectral family analysis is the wrong path here for precursor construction.

## Inputs

- Assembled nucleotide contigs (from metagenomic short reads) or NCBI RefSeq producer genomes
- RiPP modification-enzyme profile HMMs (HMMer-format .hmm files)
- Configurable ORF length threshold (default 200 amino acids) for the all-ORF strategy
- 10 kbp window size for putative BGC delimitation

## Outputs

- List of detected modification enzyme hits per contig with genomic coordinates
- Putative BGC windows (10 kbp regions centered on each enzyme hit)
- Candidate precursor ORF sets per strategy, e.g. all_orf_candidates.fasta annotated with parent BGC window (motif-ORF and all-ORF)

## How to apply

First assemble metagenomic short reads into contigs and perform 6-frame translation of the DNA sequence (as in MetaMiner's peptidogenomics pipeline, github.com/mohimanilab/MetaMiner). Run HMMer (Eddy, 2011) with RiPP modification-enzyme profile HMMs (covering classes such as lanthipeptides, lassopeptides, LAPs, linaridins, glycocins, cyanobactins, and proteusins) against the translated sequences to detect all modification enzymes. For each detected enzyme hit, construct a 10 kbp window centered on the hit — this window is the putative biosynthetic gene cluster. Within each window, extract candidate precursor peptides using the two MetaMiner strategies: the motif-ORF strategy (precursor ORFs matching known RiPP sequence motifs) and the all-ORF strategy, which enumerates all ORFs shorter than a pre-defined threshold (default 200 amino acids) without motif filtering. The rationale is that modification enzymes are more reliably detected than the short, poorly conserved leader/core precursor ORFs themselves, so enzyme hits anchor where candidate RiPPs can be predicted for later spectral search.

## Related tools

- **HMMer** (Runs profile HMM searches on the 6-frame translated sequences to detect all RiPP modification enzymes that anchor putative BGC windows)
- **MetaMiner** (Wraps the 6-frame translation, HMMer enzyme detection, 10 kbp BGC window construction, and motif-ORF/all-ORF precursor database building into one peptidogenomics command-line pipeline) — https://github.com/mohimanilab/MetaMiner

## Examples

```
hmmsearch --tblout enzyme_hits.tbl ripp_modification_enzymes.hmm sixframe_translated_contigs.fasta
```

## Evaluation signals

- All reported candidate ORFs are shorter than the configured length threshold (default 200 amino acids) and map within 10 kbp windows centered on HMM-detected modification enzymes.
- Output FASTA (e.g., all_orf_candidates.fasta) is valid FASTA and every ORF is annotated with its parent putative BGC window.
- Positive-control check: running the full downstream pipeline on the 18 isolated known RiPPs (prochlorosins, geobacillin, sublancin, haloduracin, lacticin, bicereusins, flavecins) with their producer genomes yields 18/18 identifications at 1% FDR.
- On the ACTI dataset, enzyme-driven motif-ORF mining should recover the 3 unknown + 5 known RiPPs at 0% FDR, whereas all-ORF recovers only 2 known RiPPs — a mode-sensitivity comparison to confirm the enzyme detection feeds both strategies correctly.
- Modification classes detected should be restricted to the RiPP classes MetaMiner supports (lanthipeptides, lassopeptides, LAPs, linaridins, glycocins, cyanobactins, proteusins), not generic ORF calls.

## Limitations

- The motif-ORF strategy is not sensitive, as peptides without sequence similarity to known RiPPs are missed; all-ORF compensates but yields a much larger candidate search space (e.g., 14,694 short ORFs across six BGCs in one genome).
- Blind modification searches are not performed for cyclic peptides due to inherent computational complexity, which limits which detected enzymes can be exploited downstream.
- Only ~1% of GNPS spectra have paired genomics/metagenomics data; without a paired genome or metagenome, modification-enzyme mining cannot be applied to those spectra.

## Evidence
