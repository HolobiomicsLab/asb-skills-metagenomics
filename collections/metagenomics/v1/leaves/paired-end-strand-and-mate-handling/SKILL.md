---
name: paired-end-strand-and-mate-handling
description: Use when classifying paired-end metagenomic reads (e.g. 100 bp Illumina
  mates from SRR1804065 or setB2) with LiME, when strand direction is unknown and
  ambiguous reads must be resolved with mate information.
when_to_use_negative:
- Input is single-end reads — the paired-end procedure (four collections and mate-based
  re-examination) does not apply; use the single-read-collection strategy
- Reads are already strand-oriented and mate-aware classification is unnecessary —
  running both strands duplicates the alpha-cluster and similarity steps for no gain
- Reference structures have not been merged (no ebwt/lcp/da for S) — classification
  requires merged collection data structures, not raw FASTA alone
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: 'Performs the paired-end classification: runs cluster analysis (ClusterLCP,
    ClusterBWT_DA) and Classify over the forward and reverse-complement data structures
    of both mates; invoked for the four collections via LiME_paired.sh'
  repo: https://github.com/veronicaguerrini/LiME
- name: eGSA
  role: Builds the ebwt, lcp and da (generalized enhanced suffix array) structures
    needed per read set before paired-end classification
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges the ebwt structures of the read set and genome set, computing lcp and
    da for the full collection S used in both strands' classification
  repo: https://github.com/felipelouza/egap
- name: BCR_LCP_GSA
  role: 'Alternative preprocessing: builds or incrementally updates the multi-string
    BWT, lcp and da structures for the read and genome sets'
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:12b90d3de8b3ebfe18a5794f79ad2c5c11e652538c764a6efb0e4eb1edc055da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR_LCP_GSA
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGSA
  - eGap
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# paired-end-strand-and-mate-handling

## Summary

Handles paired-end read collections in the LiME eBWT-based metagenomic classifier: because read strand direction is unknown, classification is run over four collections (forward and reverse-complement of both mates), and ambiguous reads are re-examined using mate-pair information.

## When to use

Use when classifying paired-end metagenomic reads (e.g. 100 bp Illumina mates from SRR1804065 or setB2) with LiME, when strand direction is unknown and ambiguous reads must be resolved with mate information.

## When NOT to use

- Input is single-end reads — the paired-end procedure (four collections and mate-based re-examination) does not apply; use the single-read-collection strategy
- Reads are already strand-oriented and mate-aware classification is unnecessary — running both strands duplicates the alpha-cluster and similarity steps for no gain
- Reference structures have not been merged (no ebwt/lcp/da for S) — classification requires merged collection data structures, not raw FASTA alone

## Inputs

- Paired-end read files (e.g. setB2_1.fasta and setB2_2.fasta, 100 bp reads; filtered HMP metagenome SRR1804065)
- Reference genome fasta (Refs.fasta, 930 genomes) or BLAST-filtered customized reference database
- ebwt, lcp and da data structures for the collection S (built via egsa/BCR and merged with eGap)
- LineageFile with genome taxonomy information (e.g. Reference_database.csv)

## Outputs

- Per-read classification file in comma-separated format C/U/A/H,IdSeqRead,TaxID,maxSim
- Ambiguous-entry set resolved in the paired-end re-examination phase

## How to apply

Build ebwt, lcp and da data structures separately for the read set R and genome set G (with egsa or BCR), then merge them with eGap using the --lcp and --da options (optionally --trlcp k to cap lcp values). Since reads' strand direction is unknown, both the original sequences and their reverse complements must be considered, so the alpha-cluster detection (Step 1, parameter alpha) and similarity analysis (Step 2, parameter beta) are repeated for the data structures of both strands of both paired-end reads — four collections in total. For the HMP real metagenome SRR1804065 (100 bp paired-end reads) this is done via LiME_paired.sh with alpha=16, beta=0.25, and TaxRank=1, producing a per-read classification file. In the second phase, re-examine only the ambiguous entries (A records) exploiting the paired-end information to improve assignment before final classification at the chosen taxonomic rank.

## Related tools

- **LiME** (Performs the paired-end classification: runs cluster analysis (ClusterLCP, ClusterBWT_DA) and Classify over the forward and reverse-complement data structures of both mates; invoked for the four collections via LiME_paired.sh) — https://github.com/veronicaguerrini/LiME
- **eGSA** (Builds the ebwt, lcp and da (generalized enhanced suffix array) structures needed per read set before paired-end classification) — https://github.com/felipelouza/egsa
- **eGap** (Merges the ebwt structures of the read set and genome set, computing lcp and da for the full collection S used in both strands' classification) — https://github.com/felipelouza/egap
- **BCR_LCP_GSA** (Alternative preprocessing: builds or incrementally updates the multi-string BWT, lcp and da structures for the read and genome sets) — https://github.com/giovannarosone/BCR_LCP_GSA

## Examples

```
./LiME_paired.sh setB2_1.fasta setB2_2.fasta 16 0.25 1
```

## Evaluation signals

- Every read ID appears with consistent handling across both strands of both mates, and reverse-complement hits resolve to the same taxon as forward hits after the paired-end re-examination phase
- Output file rows conform to the C/U/A/H,IdSeqRead,TaxID,maxSim schema, with ambiguous reads either resolved via mate information or reported as A/U
- On SRR1804065, the resulting classification agrees with Magic-BLAST on ~95.2% of all classified reads (93.6% same taxon), matching Table S5
- ClusterLCP was run on each of the four strand/mate collections with alpha=16, and ClusterBWT_DA with beta=0.25, before Classify with TaxRank=1

## Limitations

- Strand direction of reads is unknown, so all four strand/mate collections must be processed, multiplying the alpha-cluster and similarity computation steps
- The paper does not claim a best construction method for ebwt, da and lcp — although the classification does not depend on the construction algorithm used
- eGap's --trlcp k option caps lcp values at k (values greater than k are replaced by k), which trades lcp precision for merge speed

## Evidence

- [readme] For paired-end read collections, thus, the procedures at Step 1. and Step 2. have to be repeated for the data structures of both strands (forward and reverse complement) of both paired-end reads.: "For paired-end read collections, thus, the procedures at Step 1. and Step 2. have to be repeated for the data structures of both strands (forward and reverse complement) of both paired-end reads."
- [methods] both the original sequences and their reverse complements must be considered in order to keep reads properly oriented: "both the original sequences and their reverse complements must be considered in order to keep reads properly oriented"
- [methods] In this second phase, we only consider ambiguous entries.: "Re-examination of ambiguous reads– In this second phase, we only consider ambiguous entries."
- [results] Run LiME via LiME_paired.sh on the four collections (forward and reverse-complement of both mates), using alpha=16, beta=0.25, and TaxRank=1 for 100 bp paired-end reads: [evidence span withheld — anchor status: not_found]
- [methods] every read is assigned to a specific taxon either at a given taxonomic level: "We perform the read assignment: every read is assigned to a specific taxon either at a given taxonomic level"
