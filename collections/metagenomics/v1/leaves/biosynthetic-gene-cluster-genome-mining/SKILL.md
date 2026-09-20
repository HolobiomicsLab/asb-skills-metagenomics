---
name: biosynthetic-gene-cluster-genome-mining
description: Use when you have complete strain genomes (e.g. RefSeq FASTA) and need
  per-genome counts of BGCs, RiPP-encoding clusters, and short ORF candidates under
  motif-ORF vs all-ORF mining modes to benchmark mining sensitivity.
when_to_use_negative:
- Input is raw tandem MS spectra rather than genome sequences — MetaMiner's MS-DPR
  spectral search and 1% FDR identification steps, not genome mining, are the appropriate
  stage.
- Input is unassembled short reads without a target genome — first assemble reads
  into contigs, but expect degraded mining quality compared to complete RefSeq genomes.
- The question concerns known-structure dereplication rather than candidate enumeration
  — use Dereplicator+/Dereplicator against known structures instead.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3174
tools:
- name: antiSMASH
  role: Detects BGCs in each genome and classifies RiPP-encoding clusters (e.g. 70
    BGCs including 29 RiPP-encoding clusters in the paper's genomes; 1,140 BGCs /
    168 RiPP clusters across 36 ACTI genomes in the reproduction task).
- name: MetaMiner
  role: Generates short ORF candidate databases via the motif-ORF strategy (HMMer
    enzyme hits + 10 kbp putative BGC windows) and the all-ORF strategy (6-frame translation,
    ORFs < 200 aa).
  repo: https://github.com/mohimanilab/MetaMiner
- name: HMMer
  role: Runs on translated genome sequences to detect modification enzymes, whose
    genomic neighborhoods define the motif-ORF putative biosynthetic gene cluster
    windows.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:53cb46d3d5372bfd7db6d9dcc7ecfb5cb4aec2d69605e9427aff1db1d2930a30
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - HMMer
  - MetaMiner
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# biosynthetic-gene-cluster-genome-mining

## Summary

Enumerate biosynthetic gene clusters (BGCs), RiPP-encoding clusters, and candidate short ORFs across complete microbial genomes by combining antiSMASH cluster detection with MetaMiner's motif-ORF and all-ORF peptidogenomics strategies, quantifying how targeted motif-ORF mining shrinks the candidate space (e.g. 67 vs 565,138 short ORFs across 36 ACTI genomes).

## When to use

Use when you have complete strain genomes (e.g. RefSeq FASTA) and need per-genome counts of BGCs, RiPP-encoding clusters, and short ORF candidates under motif-ORF vs all-ORF mining modes to benchmark mining sensitivity.

## When NOT to use

- Input is raw tandem MS spectra rather than genome sequences — MetaMiner's MS-DPR spectral search and 1% FDR identification steps, not genome mining, are the appropriate stage.
- Input is unassembled short reads without a target genome — first assemble reads into contigs, but expect degraded mining quality compared to complete RefSeq genomes.
- The question concerns known-structure dereplication rather than candidate enumeration — use Dereplicator+/Dereplicator against known structures instead.

## Inputs

- Complete strain genome sequences in nucleotide FASTA format (e.g. from NCBI RefSeq)
- Number of genomes (e.g. the 36 ACTI actinobacteria strain genomes)

## Outputs

- Per-genome antiSMASH BGC counts and RiPP-encoding cluster counts
- Per-genome motif-ORF short ORF candidate counts (10 kbp windows around HMMer-detected modification enzymes)
- Per-genome all-ORF short ORF candidate counts (ORFs < 200 amino acids from 6-frame translation)
- Aggregated counts table, e.g. acti_bgc_orf_counts.csv with one row per genome

## How to apply

Download complete genomes as nucleotide FASTA from NCBI RefSeq and run antiSMASH on each genome to detect BGCs and classify RiPP-encoding clusters (reference benchmark: 1,140 BGCs including 168 RiPP-encoding across 36 ACTI actinobacteria genomes). Run MetaMiner in motif-ORF mode: it runs HMMer on translated sequences to detect modification enzymes and builds a 10 kbp window centered at each enzyme as the putative biosynthetic gene cluster, yielding a small, targeted candidate set (e.g. 67 short ORFs genome-wide). Run MetaMiner in all-ORF mode in parallel: 6-frame translation of the genome followed by collecting all ORFs shorter than the default 200 amino acid threshold, yielding an exhaustive but ~four-orders-of-magnitude larger candidate set (e.g. 565,138 short ORFs across the same 36 genomes). Assemble per-genome counts into a table with columns for BGC count, RiPP cluster count, motif-ORF short ORF count, and all-ORF short ORF count. The motif-ORF/all-ORF ratio is the key sensitivity-vs-coverage decision point: motif-ORF misses peptides without sequence similarity to known RiPPs, while all-ORF is exhaustive but computationally expensive downstream.

## Related tools

- **antiSMASH** (Detects BGCs in each genome and classifies RiPP-encoding clusters (e.g. 70 BGCs including 29 RiPP-encoding clusters in the paper's genomes; 1,140 BGCs / 168 RiPP clusters across 36 ACTI genomes in the reproduction task).)
- **MetaMiner** (Generates short ORF candidate databases via the motif-ORF strategy (HMMer enzyme hits + 10 kbp putative BGC windows) and the all-ORF strategy (6-frame translation, ORFs < 200 aa).) — https://github.com/mohimanilab/MetaMiner
- **HMMer** (Runs on translated genome sequences to detect modification enzymes, whose genomic neighborhoods define the motif-ORF putative biosynthetic gene cluster windows.)

## Evaluation signals

- Aggregate BGC count matches the reference benchmark (1,140 BGCs, of which 168 RiPP-encoding, across the 36 ACTI genomes).
- Motif-ORF and all-ORF short ORF totals reproduce ~67 vs ~565,138 genome-wide, and 2 vs 14,694 for Streptomyces roseosporus NRRL 11379 specifically.
- Motif-ORF/all-ORF candidate ratio is roughly four orders of magnitude smaller, confirming the windowed (10 kbp, enzyme-centered) strategy constrains the candidate space.
- Output CSV has one row per genome with the four expected count columns (BGC, RiPP, motif-ORF, all-ORF).
- All-ORF counts per genome are consistent with 6-frame translation and the default 200 amino acid ORF length threshold.

## Limitations

- Motif-ORF strategy is not sensitive: peptides lacking sequence similarity to known RiPPs are missed by the HMMer modification-enzyme anchoring.
- All-ORF mode generates a very large candidate database (e.g. 565,138 short ORFs across 36 genomes), inflating computational cost and multiple-testing burden downstream.
- Only 35 of roughly 20,000 RiPP-encoding BGCs in IMG-ABC have been experimentally connected to their RiPPs, limiting ground truth for validating detected clusters.
- No blind modification searches are performed for cyclic peptides due to computational complexity.
- The 99% of GNPS spectra not accompanied by genomics/metagenomics data cannot be searched at all.

## Evidence

- [discussion] runs HMMer (Eddy, 2011) on the resulting sequences to detect all the modification enzymes: "runs HMMer (Eddy, 2011) on the resulting sequences to detect all the modification enzymes"
- [discussion] 10 kbp putative BGC windows: "constructs a 10kbp window centered at each modification enzyme (this window is called the putative biosynthetic gene cluster)"
- [discussion] all-ORF 200 aa threshold and 6-frame translation: "MetaMiner (i) translates DNA sequence into protein sequence using 6-frame translation"
- [discussion] two precursor-peptide strategies: "MetaMiner uses two strategies to search for precursor peptides, motif-ORF and all-ORF strategy."
- [discussion] ORF length threshold: "identifies all the ORFs shorter than a pre-defined threshold (default is 200 amino acids)"
- [discussion] antiSMASH benchmark on study genomes: "AntiSMASH identified 70 biosynthetic gene clusters in these genomes, including 29 RiPP-encoding clusters."
- [discussion] motif-ORF sensitivity limitation: "This strategy is not sensitive, as the peptides that do not have sequence similarity to the known RiPPs are missed."
- [readme] MetaMiner availability as command line tool / workflow: [evidence span withheld — anchor status: not_found]
