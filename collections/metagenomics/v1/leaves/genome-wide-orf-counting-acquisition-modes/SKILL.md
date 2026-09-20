---
name: genome-wide-orf-counting-acquisition-modes
description: Use when benchmarking genome-mining candidate generation on a genome
  set (e.g., 36 ACTI RefSeq FASTA assemblies) where you need per-genome BGC and RiPP
  counts from antiSMASH plus short-ORF candidate counts under both motif-ORF and all-ORF
  modes.
when_to_use_negative:
- The motifs of the RiPP class of interest show no sequence similarity to known RiPPs
  — the motif-ORF strategy is explicitly not sensitive and will miss such peptides;
  all-ORF mode is the fallback despite the huge candidate count
- The analysis goal is NRPS/non-ribosomal peptide discovery, where short-ORF precursor
  databases are irrelevant (many spectral families belong to non-ribosomal peptides,
  per Dereplicator+ annotation)
- No paired genomics/metagenomics data exist for the spectra to be searched — MetaMiner
  cannot run at all (99% of GNPS spectra lack paired genomic data)
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: antiSMASH
  role: Detects and classifies biosynthetic gene clusters (including RiPP-encoding
    clusters) per genome to contextualize the ORF counts
- name: MetaMiner
  role: Implements the motif-ORF and all-ORF precursor-peptide search strategies and
    builds 10 kbp putative BGC windows; command-line tool for the ORF acquisition
    modes
  repo: github.com/mohimanilab/MetaMiner
- name: HMMer
  role: Detects all modification enzymes on translated sequences; each hit defines
    the center of a 10 kbp motif-ORF window
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:d215a0d0631ee13bc042705a0b6f8abc21d62feae069e16925fbb468d1a0650f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HMMer
  - MetaMiner
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# genome-wide-orf-counting-acquisition-modes

## Summary

Counts short ORF precursor candidates across whole genomes using MetaMiner's two acquisition strategies — motif-ORF (10 kbp windows around HMMer-detected modification enzymes) and all-ORF (6-frame translation with a 200-aa cutoff) — to quantify how sharply the motif-guided mode trims the candidate space versus exhaustive 6-frame scanning.

## When to use

Use when benchmarking genome-mining candidate generation on a genome set (e.g., 36 ACTI RefSeq FASTA assemblies) where you need per-genome BGC and RiPP counts from antiSMASH plus short-ORF candidate counts under both motif-ORF and all-ORF modes.

## When NOT to use

- The motifs of the RiPP class of interest show no sequence similarity to known RiPPs — the motif-ORF strategy is explicitly not sensitive and will miss such peptides; all-ORF mode is the fallback despite the huge candidate count
- The analysis goal is NRPS/non-ribosomal peptide discovery, where short-ORF precursor databases are irrelevant (many spectral families belong to non-ribosomal peptides, per Dereplicator+ annotation)
- No paired genomics/metagenomics data exist for the spectra to be searched — MetaMiner cannot run at all (99% of GNPS spectra lack paired genomic data)

## Inputs

- Nucleotide FASTA genome assemblies from NCBI RefSeq (e.g., 36 ACTI actinobacterial strain genomes)
- antiSMASH BGC annotation results per genome (BGC counts and RiPP-encoding cluster classification)
- HMMer-detected modification enzyme hits on translated genome sequences (drives motif-ORF window construction)

## Outputs

- Per-genome counts of short ORF candidates under motif-ORF mode (10 kbp putative BGC windows)
- Per-genome counts of short ORF candidates under all-ORF mode (6-frame translation, <200 aa)
- Aggregated table (e.g., acti_bgc_orf_counts.csv) with columns: antiSMASH BGC count, RiPP cluster count, motif-ORF short ORF count, all-ORF short ORF count
- Set of short ORFs (<200 aa) that can be randomized into decoy precursor peptides for downstream FDR estimation

## How to apply

Download the target genomes (here, 36 ACTI actinobacterial strains) in nucleotide FASTA from NCBI RefSeq and run antiSMASH on each to enumerate BGCs and RiPP-encoding clusters (expected aggregate: 1,140 BGCs, 168 RiPP-encoding). For the motif-ORF mode, run HMMer on translated genome sequence to detect modification enzymes, then construct a 10 kbp window centered at each hit (the putative biosynthetic gene cluster) and extract ORFs within those windows only. For the all-ORF mode, perform 6-frame translation of each genome and collect every ORF shorter than the default 200-amino-acid threshold. Aggregate per-genome counts under each mode (expected: 67 motif-ORF vs 565,138 all-ORF short ORFs across the 36 genomes — a roughly four-order-of-magnitude reduction) and write one row per genome to a CSV table.

## Related tools

- **antiSMASH** (Detects and classifies biosynthetic gene clusters (including RiPP-encoding clusters) per genome to contextualize the ORF counts)
- **MetaMiner** (Implements the motif-ORF and all-ORF precursor-peptide search strategies and builds 10 kbp putative BGC windows; command-line tool for the ORF acquisition modes) — github.com/mohimanilab/MetaMiner
- **HMMer** (Detects all modification enzymes on translated sequences; each hit defines the center of a 10 kbp motif-ORF window)

## Evaluation signals

- Aggregate invariant: motif-ORF short-ORF count (expected 67 on 36 ACTI genomes) should be ~4 orders of magnitude smaller than the all-ORF count (expected 565,138)
- antiSMASH totals should reproduce 1,140 BGCs including 168 RiPP-encoding clusters across the 36 ACTI strains
- Per-genome spot check: Streptomyces roseosporus NRRL 11379 should yield 2 motif-ORF vs 14,694 all-ORF short ORFs
- All-ORF mode must apply the default 200-amino-acid ORF length threshold; motif-ORF ORFs must lie inside 10 kbp windows centered on modification enzyme hits
- Output CSV must have exactly one row per genome with all four count columns populated

## Limitations

- The motif-ORF strategy is not sensitive and misses peptides without sequence similarity to known RiPPs, trading recall for a dramatic candidate reduction
- All-ORF mode generates enormous candidate databases (565,138 short ORFs across 36 genomes), which inflates downstream search cost and demands rigorous target-decoy FDR control
- Blind modification searches are not performed for cyclic peptides due to inherent computational complexity, capping what the candidate ORFs can ultimately resolve

## Evidence

- [readme] availability as command-line tool / GNPS workflow: [evidence span withheld — anchor status: not_found]
