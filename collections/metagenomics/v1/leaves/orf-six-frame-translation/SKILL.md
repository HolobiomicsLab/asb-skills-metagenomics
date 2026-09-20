---
name: orf-six-frame-translation
description: Use when you need to enumerate all possible short peptide precursors
  from nucleotide FASTA genomes/contigs — e.g. estimating candidate short ORFs for
  RiPP precursor databases or benchmarking motif-ORF vs all-ORF search-space sizes.
when_to_use_negative:
- When the goal is a small, high-precision candidate set and known RiPP modification
  enzymes are detectable — the motif-ORF strategy (HMMer + 10 kbp BGC windows) already
  reduces candidates by ~4 orders of magnitude (67 vs 565,138 on the ACTI genomes)
- When peptides have sequence similarity to known RiPPs — the all-ORF mode is unnecessary
  search-space inflation; motif-ORF suffices
- On raw unassembled short reads — assemble reads into contigs first; MetaMiner itself
  does this before translation
edam_operation: http://edamontology.org/operation_0446
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: MetaMiner
  role: 'Implements the all-ORF strategy: 6-frame translation of DNA sequence followed
    by collection of ORFs shorter than the 200 aa threshold, as one of its two precursor-database
    construction strategies'
  repo: https://github.com/mohimanilab/MetaMiner
- name: HMMer
  role: Used in the complementary motif-ORF strategy to detect modification enzymes
    and define 10 kbp putative biosynthetic gene cluster windows, benchmarked against
    the all-ORF counts
- name: antiSMASH
  role: Provides BGC and RiPP-encoding cluster counts on the same genomes to contextualize
    the all-ORF candidate enumeration (e.g. 1,140 BGCs including 168 RiPP-encoding
    on 36 ACTI strains)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:ffe84d9c2b1365cbe916e0271077f5f4e51284b00e9fa8e1d745b0ec10cc3816
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - HMMer
  - MetaMiner
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0446
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# orf-six-frame-translation

## Summary

Enumerates candidate short ORFs by translating microbial genome (or metagenome contig) sequences in all six frames and collecting open reading frames shorter than a pre-defined length threshold (default 200 amino acids), feeding the all-ORF candidate precursor database for MetaMiner RiPP peptidogenomics.

## When to use

Use when you need to enumerate all possible short peptide precursors from nucleotide FASTA genomes/contigs — e.g. estimating candidate short ORFs for RiPP precursor databases or benchmarking motif-ORF vs all-ORF search-space sizes.

## When NOT to use

- When the goal is a small, high-precision candidate set and known RiPP modification enzymes are detectable — the motif-ORF strategy (HMMer + 10 kbp BGC windows) already reduces candidates by ~4 orders of magnitude (67 vs 565,138 on the ACTI genomes)
- When peptides have sequence similarity to known RiPPs — the all-ORF mode is unnecessary search-space inflation; motif-ORF suffices
- On raw unassembled short reads — assemble reads into contigs first; MetaMiner itself does this before translation

## Inputs

- Nucleotide FASTA genome assemblies or assembled metagenomic contigs (if input is short reads, MetaMiner first assembles them into contigs)
- ORF length threshold (default 200 amino acids)

## Outputs

- Set of translated protein sequences from 6-frame translation
- Candidate short ORF (<200 aa) sequences forming the all-ORF precursor database
- Per-genome candidate counts for comparison with motif-ORF mode (e.g. acti_bgc_orf_counts.csv)

## How to apply

Translate each input DNA sequence (genome FASTA or assembled contigs) into protein sequence in all six frames using 6-frame translation, then scan each translated frame for open reading frames and retain every ORF shorter than the pre-defined threshold (default 200 amino acids). In MetaMiner's all-ORF strategy this candidate set is paired with the motif-ORF strategy — which instead runs HMMer on sequences to detect modification enzymes and builds a 10 kbp window centered at each hit as the putative biosynthetic gene cluster — so the two modes can be compared for candidate-count reduction. On the 36 ACTI strain genomes this yielded 565,138 all-ORF short ORFs versus only 67 motif-ORF candidates (a roughly four-order-of-magnitude reduction), so use all-ORF only when motif-based detection would miss novel RiPPs lacking sequence similarity to known modification-enzyme motifs. Aggregate per-genome counts into a table (e.g. antiSMASH BGC count, RiPP cluster count, motif-ORF count, all-ORF count) to sanity-check the candidate database size before spectral searching.

## Related tools

- **MetaMiner** (Implements the all-ORF strategy: 6-frame translation of DNA sequence followed by collection of ORFs shorter than the 200 aa threshold, as one of its two precursor-database construction strategies) — https://github.com/mohimanilab/MetaMiner
- **HMMer** (Used in the complementary motif-ORF strategy to detect modification enzymes and define 10 kbp putative biosynthetic gene cluster windows, benchmarked against the all-ORF counts)
- **antiSMASH** (Provides BGC and RiPP-encoding cluster counts on the same genomes to contextualize the all-ORF candidate enumeration (e.g. 1,140 BGCs including 168 RiPP-encoding on 36 ACTI strains))

## Examples

```
Run the RIPPQUEST workflow at GNPS (https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22RIPPQUEST%22%7D) with genome FASTA inputs in all-ORF mode to enumerate <200 aa ORFs via 6-frame translation
```

## Evaluation signals

- Candidate counts should be reproducible per genome: e.g. 2 all-ORF short ORFs for Streptomyces roseosporus NRRL 11379 and 565,138 total across the 36 ACTI genomes
- All reported ORFs must satisfy the length cutoff — no ORF of 200 amino acids or longer appears in the candidate set
- All six translation frames of each contig must be represented in the translated protein sequence set
- Aggregate count table should include paired motif-ORF vs all-ORF counts per genome; motif-ORF counts should be dramatically smaller (~4 orders of magnitude on ACTI strains)
- RiPP-encoding cluster counts from antiSMASH (168 of 1,140 BGCs on ACTI) should be consistent with the genomes from which ORF candidates were derived

## Limitations

- The all-ORF strategy generates a massive candidate database (565,138 candidates vs 67 for motif-ORF on the ACTI genomes), inflating search space and computational cost
- Blind modification searches are not performed for cyclic peptides due to computational complexity, constraining what all-ORF candidates can be matched against
- Only 99% of GNPS spectra lack paired genomics/metagenomics data and cannot be searched, so all-ORF candidate databases are only useful where matched sequencing exists
- No blind modification searches for cyclic peptides due to computational complexity

## Evidence

- [readme] README pointing to GNPS workflow execution: [evidence span withheld — anchor status: not_found]
