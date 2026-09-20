---
name: metagenomic-read-classification
description: Use when classifying paired-end metagenomic reads (e.g. simulated setA2/setB2
  or HMP metagenome SRR1804065) against a reference genome FASTA in alignment-free
  fashion, computing sensitivity/precision/F1/specificity.
when_to_use_negative:
- When an alignment- or mapping-based classifier is required (e.g. variant-aware or
  spliced alignment reporting) — LiME is explicitly alignment-free, mapping-free and
  assembly-free
- When reads lack a suitable curated reference database; LiME's precision/specificity
  claims rest on the Lindgreen et al. simulated benchmarks and a BLAST-filtered HMP
  sample with a customized reference database
- When only raw, unfiltered real metagenomes are available without the BLAST 98%-identity
  filtering step used in the paper's real-data protocol
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: LiME
  role: Core three-step classifier (ClusterLCP, ClusterBWT_DA, Classify) with LiMEeBWT
    (EBWT=1) and LiMEDA (EBWT=0) variants
  repo: https://github.com/veronicaguerrini/LiME
- name: BCR
  role: Builds ebwt/lcp/da data structures from FASTA input (can update genome structures
    incrementally with reads)
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: egsa
  role: External-memory alternative to build generalized suffix array, LCP and BWT
    (converted to ebwt/lcp/da via EGSAtoBCR)
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges the ebwt/lcp/da of read set R and genome set G with --lcp and --da
    options (optionally --trlcp k)
  repo: https://github.com/felipelouza/egap
- name: MagicBlast
  role: Benchmark classifier; strongest agreement baseline on real HMP data (~95.2%
    of classified reads classified identically)
- name: Kraken 2
  role: Benchmark classifier compared on simulated and real datasets
- name: CLARK-S
  role: Benchmark classifier compared on simulated and real datasets
- name: Centrifuge
  role: Benchmark classifier compared on simulated and real datasets
- name: TaxMaps
  role: Benchmark classifier compared on simulated and real datasets
- name: BLAST
  role: Used to filter the real metagenome SRR1804065 at 98% sequence identity and
    derive the customized reference database
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:98a13ad22dfa414e2a0b905ea5afb398449d427535554139db9afbe6c7aa36a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - BLAST
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGAP
  - eGSA
  - eGap
  - egsa
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# metagenomic-read-classification

## Summary

Lightweight, alignment-free metagenomic read classification with LiME, which compares reads against reference genomes via the extended Burrows-Wheeler transform (ebwt), document array (da) and LCP array, assigning each paired-end read to a taxon at species level (TaxRank=1) with high precision (99.9% positive-control assignment, <0.01% negative-control classification).

## When to use

Use when classifying paired-end metagenomic reads (e.g. simulated setA2/setB2 or HMP metagenome SRR1804065) against a reference genome FASTA in alignment-free fashion, computing sensitivity/precision/F1/specificity.

## When NOT to use

- When an alignment- or mapping-based classifier is required (e.g. variant-aware or spliced alignment reporting) — LiME is explicitly alignment-free, mapping-free and assembly-free
- When reads lack a suitable curated reference database; LiME's precision/specificity claims rest on the Lindgreen et al. simulated benchmarks and a BLAST-filtered HMP sample with a customized reference database
- When only raw, unfiltered real metagenomes are available without the BLAST 98%-identity filtering step used in the paper's real-data protocol

## Inputs

- Paired-end metagenomic read FASTA files (e.g. setB2_1.fasta and setB2_2.fasta, 100 bp reads)
- Reference genome database FASTA (Refs.fasta, 930 genomes)
- Taxonomic lineage file (semicolon-separated, e.g. ./Datasets/Reference_database.csv)
- ebwt, lcp and da data structures for merged read+genome collections (from BCR/egsa + eGap)

## Outputs

- Comma-separated classification OutputFile with records 'C/U/A/H,IdSeqRead,TaxID,maxSim' (C=Classified, U=unclassified, A=Ambiguous, H=higher ranks)
- Species-level accuracy metrics: sensitivity (SEN), precision (PREC), F1 score, specificity, and ambiguous/unclassified read counts per dataset and LiME variant
- Agreement tables vs. benchmark classifiers (Jaccard-based rates rid/ras against Magic-BLAST-classified read sets)

## How to apply

Build the ebwt, lcp and da data structures for the genome set G (930 genomes in Refs.fasta) and read set R separately with BCR or egsa, then merge them per strand with eGap using --lcp and --da (optionally --trlcp k), producing four collections per paired-end dataset (forward and reverse complement of both mates). Compile LiME twice: EBWT=1 (LiMEeBWT, reads da and ebwt) and EBWT=0 (LiMEDA, reads only da). Run Step 1 with ClusterLCP using alpha=16, Step 2 with ClusterBWT_DA using beta=0.25, and Step 3 with Classify at TaxRank=1 (species) with a semicolon-separated LineageFile (Accession_number;Species_TaxID;Genus_TaxID;Family_TaxID;Order_TaxID;Class_TaxID;Phylum_TaxID), which re-examines ambiguous paired-end reads by summed mate similarity with a 0.02 tolerance around the maximum. Parse the comma-separated OutputFile records 'C/U/A/H,IdSeqRead,TaxID,maxSim' to compute sensitivity, precision, F1 and specificity; the chosen alpha=16, beta=0.25 was near-optimal in the paper's parameter sweep, and higher beta trades sensitivity for precision.

## Related tools

- **LiME** (Core three-step classifier (ClusterLCP, ClusterBWT_DA, Classify) with LiMEeBWT (EBWT=1) and LiMEDA (EBWT=0) variants) — https://github.com/veronicaguerrini/LiME
- **BCR** (Builds ebwt/lcp/da data structures from FASTA input (can update genome structures incrementally with reads)) — https://github.com/giovannarosone/BCR_LCP_GSA
- **egsa** (External-memory alternative to build generalized suffix array, LCP and BWT (converted to ebwt/lcp/da via EGSAtoBCR)) — https://github.com/felipelouza/egsa
- **eGap** (Merges the ebwt/lcp/da of read set R and genome set G with --lcp and --da options (optionally --trlcp k)) — https://github.com/felipelouza/egap
- **MagicBlast** (Benchmark classifier; strongest agreement baseline on real HMP data (~95.2% of classified reads classified identically))
- **Kraken 2** (Benchmark classifier compared on simulated and real datasets)
- **CLARK-S** (Benchmark classifier compared on simulated and real datasets)
- **Centrifuge** (Benchmark classifier compared on simulated and real datasets)
- **TaxMaps** (Benchmark classifier compared on simulated and real datasets)
- **BLAST** (Used to filter the real metagenome SRR1804065 at 98% sequence identity and derive the customized reference database)

## Examples

```
Classify 1 ReadsF+Refs.fasta.out numReads 930 OutputFile LineageFile 1 threads
```

## Evaluation signals

- Every paired-end read pair receives exactly one status label from {C, U, A, H} in the comma-separated OutputFile
- On the positive control (~20M reads of setB2), the correct-assignment rate (classified/total) is approximately 0.999; on the ~5M random-shuffled negative control, the classified rate is below 0.0001
- Reproduced SEN/PREC/F1/specificity values are consistent with supplementary Table S1 values for LiME on setA2 and setB2
- Agreement with Magic-BLAST on the real HMP metagenome: ~95.2% of all classified reads classified identically, ~93.6% to the same taxon, and LiME misses ~42,788 Magic-BLAST-classified reads (Table S5)
- Parameter sweep (alpha in [16,22], beta in [0.15,0.40]) shows monotone sensitivity-up/precision-down as beta decreases, with alpha=16, beta=0.25 among the best F1 configurations (Table S2)

## Limitations

- The paper does not state version numbers of the compared tools, and comparison against state-of-the-art tools is discussed without them
- Finding the best method for building the ebwt/da/lcp data structures is explicitly out of scope; classification is independent of the construction algorithm used
- Full parameter-sweep details (Table S2), setA2 performance (Table S3) and thread scaling (Table S4) are deferred to supplementary files, not the main text
- eGap's --trlcp k option caps computed lcp values at k, which may affect clustering context when k is too small

## Evidence

- [methods] alpha-cluster detection: [evidence span withheld — anchor status: not_found]
- [methods] beta threshold: [evidence span withheld — anchor status: not_found]
- [readme] eGap merge: "To merge the two ebwts associated with the sets *R* and *G*, one could use eGap [https://github.com/felipelouza/egap]"
- [readme] output format: "OutputFile is the comma-separated file where the classification results will be stored"
- [abstract] precision/specificity finding: "99.9% of the positive control reads are correctly assigned and the percentage of classified reads of the negative control is less than 0.01%"
