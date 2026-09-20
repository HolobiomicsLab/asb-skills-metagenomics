---
name: biosynthetic-gene-cluster-window-construction
description: Use when assembled nucleotide contigs must be partitioned into putative
  RiPP-encoding BGC regions, by anchoring fixed-size (10 kbp) windows on HMMer-detected
  modification enzyme hits before candidate ORF extraction.
when_to_use_negative:
- Precursor peptides have sequence similarity to known RiPPs and a more sensitive
  motif-ORF strategy would suffice — the all-ORF window approach yields a very large
  candidate set (e.g., 14,694 short ORFs across six BGCs in one genome) at the cost
  of search space
- The BGC boundaries are already known from prior annotation (e.g., antiSMASH-identified
  clusters), so fixed 10 kbp enzyme-centered windows add no information
- Input is raw short reads without assembly — contigs must be produced first, since
  windows are defined on assembled sequences
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: HMMer
  role: Detects RiPP modification enzymes on six-frame-translated contigs; each enzyme
    hit anchors a 10 kbp putative BGC window
- name: MetaMiner
  role: Implements the window-construction step within its all-ORF precursor-candidate
    generation pipeline
  repo: https://github.com/mohimanilab/MetaMiner
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:57270a7fbfaf41796460995bf035cca024a1c21ace4ad058154abf0ed88cf825
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - HMMer
  - MetaMiner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# biosynthetic-gene-cluster-window-construction

## Summary

Constructs putative biosynthetic gene cluster (BGC) regions in MetaMiner by building a 10 kbp window centered on each RiPP modification enzyme detected with HMMer on six-frame-translated nucleotide contigs. These windows define the search space from which candidate precursor ORFs are extracted.

## When to use

Use when assembled nucleotide contigs must be partitioned into putative RiPP-encoding BGC regions, by anchoring fixed-size (10 kbp) windows on HMMer-detected modification enzyme hits before candidate ORF extraction.

## When NOT to use

- Precursor peptides have sequence similarity to known RiPPs and a more sensitive motif-ORF strategy would suffice — the all-ORF window approach yields a very large candidate set (e.g., 14,694 short ORFs across six BGCs in one genome) at the cost of search space
- The BGC boundaries are already known from prior annotation (e.g., antiSMASH-identified clusters), so fixed 10 kbp enzyme-centered windows add no information
- Input is raw short reads without assembly — contigs must be produced first, since windows are defined on assembled sequences

## Inputs

- Assembled nucleotide contigs (from short-read assembly of genomic or metagenomic data)
- RiPP modification-enzyme profile HMMs run via HMMer
- Six-frame-translated protein sequences of the contigs

## Outputs

- Putative biosynthetic gene cluster annotations (10 kbp windows centered on each detected modification enzyme)
- all_orf_candidates.fasta — candidate short ORFs (<200 aa) annotated with their parent putative BGC window

## How to apply

First translate each assembled nucleotide contig into protein sequence in all six frames, then run HMMer (Eddy, 2011) with RiPP modification-enzyme profile HMMs against the six-frame translations to detect all modification enzymes. For every detected modification enzyme hit, construct a 10 kbp window centered on the hit position; each window is labeled a putative biosynthetic gene cluster. The 10 kbp span is a fixed parameter chosen to capture the modification enzyme plus its flanking precursor peptide and accessory genes. Within each window, all open reading frames shorter than the configurable threshold (default 200 amino acids) are extracted as candidate precursor peptides in the all-ORF strategy, which deliberately does not filter by RiPP sequence motifs. Correct application is judged by verifying that every candidate ORF maps within a 10 kbp window centered on a detected modification enzyme hit.

## Related tools

- **HMMer** (Detects RiPP modification enzymes on six-frame-translated contigs; each enzyme hit anchors a 10 kbp putative BGC window)
- **MetaMiner** (Implements the window-construction step within its all-ORF precursor-candidate generation pipeline) — https://github.com/mohimanilab/MetaMiner

## Evaluation signals

- Verify all_orf_candidates.fasta exists and parses as valid FASTA
- Every reported ORF sequence length is below the 200 amino acid threshold (default)
- Every candidate ORF maps within a 10 kbp window centered on a HMMer-detected modification enzyme hit
- Each window is anchored on exactly one detected modification enzyme from the HMMer results on the six-frame translations
- Candidate counts are consistent with the exhaustive all-ORF design (thousands of ORFs per genome, e.g., 14,694 across six BGCs), not a small motif-filtered set

## Limitations

- The 10 kbp fixed window may miss BGC components lying outside the window or include unrelated genes; it is a heuristic centered on modification enzyme hits
- The all-ORF candidate generation does not filter by RiPP sequence motifs, yielding a much larger candidate set (e.g., 14,694 short ORFs across six BGCs in one genome) and a larger search space
- 99% of GNPS spectra lack paired genomics/metagenomics data, so windows cannot be constructed for those samples

## Evidence

- [discussion] constructs a 10kbp window centered at each modification enzyme (this window is called the putative biosynthetic gene cluster): "constructs a 10kbp window centered at each modification enzyme (this window is called the putative biosynthetic gene cluster)"
- [discussion] runs HMMer (Eddy, 2011) on the resulting sequences to detect all the modification enzymes: "runs HMMer (Eddy, 2011) on the resulting sequences to detect all the modification enzymes"
- [discussion] identifies all the ORFs shorter than a pre-defined threshold (default is 200 amino acids): "identifies all the ORFs shorter than a pre-defined threshold (default is 200 amino acids)"
- [readme] MetaMiner is currently available as a workflow at the Global Natural Products Social Molecular Networking (GNPS): [evidence span withheld — anchor status: not_found]
