---
name: parameter-sensitivity-sweep-analysis
description: 'Use when benchmarking LiME (or an eBWT-based classifier) across parameter
  grids: you need to measure how alpha-clusters of minimum length alpha and threshold
  beta jointly affect classification sensitivity, precision and F1 against known read
  provenances.'
when_to_use_negative:
- The goal is classifying a real metagenome (e.g. SRR1804065) with a single fixed
  parameter setting rather than characterizing parameter sensitivity on simulated
  data with known provenances
- The classifier is not LiME and does not expose an alpha (minimum context length)
  or beta (similarity threshold) parameter, so the grid is not meaningful
- Only summary performance is needed and the full sweep table would not be checked
  against the monotone trade-off or Table S2 near-optimality of alpha=16, beta=0.25
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: 'Performs the classification being swept: ClusterLCP builds the alpha-cluster
    collection, ClusterBWT_DA computes similarities under threshold beta, Classify
    assigns reads to taxa'
  repo: https://github.com/veronicaguerrini/LiME
- name: BCR
  role: Builds ebwt, lcp and da data structures for the fasta collection prior to
    the sweep
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: egsa
  role: Alternative external-memory builder of ebwt, lcp and da (single .gesa file,
    converted with EGSAtoBCR)
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges the read and genome eBWTs with options --lcp and --da (optionally --trlcp
    k) to produce data structures for collection S
  repo: https://github.com/felipelouza/egap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:f099e3cc97820c090307fd70cb1d3f06d374b9e1e6479dcf559de4fb5db428eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - LiME
  - eGap
  - egsa
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# parameter-sensitivity-sweep-analysis

## Summary

Reproduces LiME's parameter sweep over minimum context length alpha (16-22) and similarity threshold beta (0.15-0.40) on simulated metagenomes, quantifying the sensitivity-vs-precision trade-off and identifying the best F1 configuration reported in Additional file 1 Table S2.

## When to use

Use when benchmarking LiME (or an eBWT-based classifier) across parameter grids: you need to measure how alpha-clusters of minimum length alpha and threshold beta jointly affect classification sensitivity, precision and F1 against known read provenances.

## When NOT to use

- The goal is classifying a real metagenome (e.g. SRR1804065) with a single fixed parameter setting rather than characterizing parameter sensitivity on simulated data with known provenances
- The classifier is not LiME and does not expose an alpha (minimum context length) or beta (similarity threshold) parameter, so the grid is not meaningful
- Only summary performance is needed and the full sweep table would not be checked against the monotone trade-off or Table S2 near-optimality of alpha=16, beta=0.25

## Inputs

- Combined fasta file of metagenomic reads and reference genomes (e.g. ReadsF+Refs.fasta, simulated setA2/setB2-style reads plus Refs.fasta with 930 genomes)
- Precomputed ebwt, lcp and da data structures for collection S (built with BCR, egsa, then merged with eGap --lcp --da)
- Known simulated read provenances (ground-truth labels) for computing sensitivity, precision and F1
- Semicolon-separated LineageFile with genome taxonomy information for species-level classification
- Parameter grid values: alpha in [16,22], beta in [0.15,0.40], EBWT variant in {0,1}

## Outputs

- Comma-separated classification OutputFile per configuration with records 'C/U/A/H,IdSeqRead,TaxID,maxSim'
- alpha x beta x variant sweep table of sensitivity, precision and F1 scores
- Validated trade-off trend summary (monotone sensitivity-up / precision-down as beta decreases)

## How to apply

First build the ebwt, lcp and da data structures for the collection S (reads plus 930 reference genomes) with BCR or egsa, merging the read and genome eBWTs with eGap using the --lcp and --da options (optionally --trlcp k). Then, for each alpha in {16,...,22}, run ClusterLCP on the combined fasta file (e.g. ReadsF+Refs.fasta) passing numReads, numRefs, alpha and a thread count to build the alpha-cluster collection. For each compile variant (EBWT=1 default strategy vs EBWT=0 strategy of Step 2) and each beta in {0.15,...,0.40}, run ClusterBWT_DA followed by Classify at the species-level rank (TaxRank=1) using the lineage file, producing a comma-separated OutputFile with records of the form C/U/A,IdSeqRead,TaxID,maxSim. Parse each OutputFile and compute sensitivity, precision and F1 against the known simulated provenances; assemble the alpha x beta x variant table and check that lower beta raises sensitivity while lowering precision, with alpha=16, beta=0.25 among the best F1 configurations as in Table S2.

## Related tools

- **LiME** (Performs the classification being swept: ClusterLCP builds the alpha-cluster collection, ClusterBWT_DA computes similarities under threshold beta, Classify assigns reads to taxa) — https://github.com/veronicaguerrini/LiME
- **BCR** (Builds ebwt, lcp and da data structures for the fasta collection prior to the sweep) — https://github.com/giovannarosone/BCR_LCP_GSA
- **egsa** (Alternative external-memory builder of ebwt, lcp and da (single .gesa file, converted with EGSAtoBCR)) — https://github.com/felipelouza/egsa
- **eGap** (Merges the read and genome eBWTs with options --lcp and --da (optionally --trlcp k) to produce data structures for collection S) — https://github.com/felipelouza/egap

## Examples

```
ClusterLCP ReadsF+Refs.fasta numReads 930 16 4 && ClusterBWT_DA ReadsF+Refs.fasta 100 0.25 4 && Classify 1 OutputClusterBWT_DA numReads 930 Output.tsv LineageFile 1 4
```

## Evaluation signals

- The results table covers all alpha in [16,22] and beta in [0.15,0.40] for both EBWT=1 and EBWT=0 variants (no missing cells)
- The sweep reproduces the monotone trend: decreasing beta increases sensitivity and decreases precision
- alpha=16 with beta=0.25 is among the best F1 configurations, consistent with Additional file 1 Table S2
- Each parsed OutputFile record conforms to the format C/U/A,IdSeqRead,TaxID,maxSim and sensitivity, precision and F1 are computable against the known simulated provenances

## Limitations

- The full parameter sweep results are deferred to supplementary Table S2 (Additional file 1) and are not included in the main article text, so comparison relies on the supplementary table
- The classification does not depend on the construction algorithm used for ebwt, da and lcp, but finding the best construction method is out of the paper's scope
- The paper states the lcp-based clustering requires reading both da(S) and lcp(S), and ClusterBWT_DA needs only da(S) for EBWT=0 but both da(S) and ebwt(S) for EBWT=1, so the two variants have different data-structure dependencies

## Evidence

- [full_text] for a fixed minimum context length alpha, increasing the similarity threshold beta decreases LiME's sensitivity while increasing its precision: [evidence span withheld — anchor status: not_found]
- [methods] In Step 1, inspired by Remark 1, we build the collection Cα of α-clusters: [evidence span withheld — anchor status: not_found]
- [methods] We set a threshold value β (0≤β<1) and we attempt to classify: [evidence span withheld — anchor status: not_found]
- [results] from which we can calculate the quality metrics: sensitivity SEN, precision PREC, and the F1 score: "from which we can calculate the quality metrics: sensitivity SEN, precision PREC, and the F1 score"
- [readme] One can choose between the two strategies of Step 2. by setting parameter EBWT: "One can choose between the two strategies of Step 2. by setting parameter EBWT:"
- [readme] ClusterLCP ReadsF+Refs.fasta numReads numRefs alpha threads: "ClusterLCP ReadsF+Refs.fasta numReads numRefs alpha threads"
