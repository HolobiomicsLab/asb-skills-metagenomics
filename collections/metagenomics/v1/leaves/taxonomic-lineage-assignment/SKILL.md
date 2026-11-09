---
name: taxonomic-lineage-assignment
description: Use when similarity matrices between reads and reference genomes have
  been computed (e.g. by LiME's ClusterLCP/ClusterBWT_DA) and each read must be assigned
  to a TaxID at a specific rank such as species or phylum.
when_to_use_negative:
- The input is raw reads without precomputed ebwt/lcp/da data structures and merged
  read+genome collections — the preprocessing step (BCR/eGSA build and eGap merge
  with --lcp and --da) must be done first
- Reads must be aligned to full reference sequences or need alignment coordinates
  — LiME is alignment-free and mapping-free and only reports TaxID and a similarity
  score, not alignments
- A single fixed taxonomic rank is not required and only an approximate higher-level
  assignment is wanted without lineage-based ranking — in that case the HIGHER=1 compilation
  handles it, but per-rank lineage assignment as described here is not what is needed
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: Provides the Classify executable that assigns reads to a taxon at a chosen
    TaxRank using the LineageFile and the similarity matrices; compiled with EBWT=1
    (LiMEeBWT) or EBWT=0 (LiMEDA), optionally HIGHER=1
  repo: https://github.com/veronicaguerrini/LiME
- name: BCR
  role: Builds the ebwt, lcp and da data structures for the read and genome collections
    prior to classification; can also be used incrementally to update genome structures
    with reads
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: eGSA
  role: External-memory alternative to BCR for building the generalized enhanced suffix
    array (GSA, LCP, optional BWT); output converted with EGSAtoBCR
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges the ebwt data structures of the read set R and genome set G with options
    --lcp and --da (optionally --trlcp k) to produce the merged collections that Classify
    consumes
  repo: https://github.com/felipelouza/egap
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:64813e55a4e70e156eb50581c192c7ee9b15b8c605fc16cfdd04e4764ade5f9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGAP
  - eGSA
  - eGap
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# taxonomic-lineage-assignment

## Summary

Assign metagenomic reads to taxa at a chosen taxonomic rank (genome through phylum) using LiME's Classify step, which consumes the similarity matrices from ClusterBWT_DA plus a semicolon-separated taxonomic lineage file and emits per-read labels C/U/A/H with TaxID and maxSim.

## When to use

Use when similarity matrices between reads and reference genomes have been computed (e.g. by LiME's ClusterLCP/ClusterBWT_DA) and each read must be assigned to a TaxID at a specific rank such as species or phylum.

## When NOT to use

- The input is raw reads without precomputed ebwt/lcp/da data structures and merged read+genome collections — the preprocessing step (BCR/eGSA build and eGap merge with --lcp and --da) must be done first
- Reads must be aligned to full reference sequences or need alignment coordinates — LiME is alignment-free and mapping-free and only reports TaxID and a similarity score, not alignments
- A single fixed taxonomic rank is not required and only an approximate higher-level assignment is wanted without lineage-based ranking — in that case the HIGHER=1 compilation handles it, but per-rank lineage assignment as described here is not what is needed

## Inputs

- Output files from ClusterBWT_DA (similarity matrices for alpha-clusters)
- LineageFile: semicolon-separated taxonomic lineage (e.g. Reference_database.csv with format Accession_number;Species_TaxID;Genus_TaxID;Family_TaxID;Order_TaxID;Class_TaxID;Phylum_TaxID)
- Total number of reads in S and total number of reference genomes (e.g. 930 for Refs.fasta)
- Threshold beta value (e.g. 0.25) used by ClusterBWT_DA
- Paired-end read data structures for forward and reverse-complement strands (four merged collections per dataset)

## Outputs

- OutputFile: comma-separated file with records 'C/U/A/H,IdSeqRead,TaxID,maxSim' (one status label per read/pair)
- Per-rank counts of classified (C), unclassified (U), ambiguous (A) and higher-rank (H) reads
- Sensitivity, precision, F1 and specificity metrics at the chosen taxonomic rank (e.g. species or phylum)
- Relative abundance estimates aggregated by TaxID at the chosen rank

## How to apply

Run Classify on the output files of ClusterBWT_DA, passing the total number of reads, the number of reference genomes (930 for Refs.fasta), an OutputFile, the LineageFile in the format Accession_number;Species_TaxID;Genus_TaxID;Family_TaxID;Order_TaxID;Class_TaxID;Phylum_TaxID (Reference_database.csv for the Lindgreen et al. datasets), a TaxRank (TaxRank=1 for species, TaxRank=6 for phylum), and the number of threads. Classify assigns every read a status label C (classified), U (unclassified, when the maximum similarity is below the beta threshold, beta=0.25 in the benchmark), A (ambiguous, resolved for paired-end data by re-examining mates with maximum mate-summed similarity and a tolerance of 0.02 on the maximum), or H (classified at higher ranks when compiled with HIGHER=1). Parse the comma-separated OutputFile records 'C/U/A/H,IdSeqRead,TaxID,maxSim' to obtain one label per read pair, then aggregate by TaxID for sensitivity, precision, F1, specificity or relative-abundance estimates. Because reads' strand direction is unknown, run the full pipeline on both forward and reverse-complement data structures (four merged collections per paired-end dataset, e.g. Reads1F+Refs, Reads1RC+Refs, Reads2F+Refs, Reads2RC+Refs).

## Related tools

- **LiME** (Provides the Classify executable that assigns reads to a taxon at a chosen TaxRank using the LineageFile and the similarity matrices; compiled with EBWT=1 (LiMEeBWT) or EBWT=0 (LiMEDA), optionally HIGHER=1) — https://github.com/veronicaguerrini/LiME
- **BCR** (Builds the ebwt, lcp and da data structures for the read and genome collections prior to classification; can also be used incrementally to update genome structures with reads) — https://github.com/giovannarosone/BCR_LCP_GSA
- **eGSA** (External-memory alternative to BCR for building the generalized enhanced suffix array (GSA, LCP, optional BWT); output converted with EGSAtoBCR) — https://github.com/felipelouza/egsa
- **eGap** (Merges the ebwt data structures of the read set R and genome set G with options --lcp and --da (optionally --trlcp k) to produce the merged collections that Classify consumes) — https://github.com/felipelouza/egap

## Examples

```
Classify 4 setB2_1F.out setB2_1RC.out setB2_2F.out setB2_2RC.out 20249373 930 OutputFile.csv Reference_database.csv 1 8
```

## Evaluation signals

- Every read pair in the OutputFile carries exactly one status label among C/U/A/H
- Reproduced species-level SEN/PREC/F1/specificity for LiME are consistent with Table S1: very high precision and specificity, 99.9% of positive control reads correctly assigned and less than 0.01% of negative control reads classified
- Phylum-level sensitivity of 95.55% (setA2) and 95.95% (setB2) and F1 of 97.66% (setA2) and 97.91% (setB2) are matched on the Lindgreen et al. benchmark datasets
- Estimated phylum abundances are concordant with the phylum composition designed in the Lindgreen et al. benchmarking datasets
- Unclassified entries only occur when the maximum similarity score is below the beta threshold (0.25); ambiguous entries are resolved via mate-summed similarity within a 0.02 tolerance

## Limitations

- Only ambiguous entries are re-examined with paired-end information; single-end reads cannot exploit mate-based disambiguation
- Classification results on real data (e.g. SRR1804065) required prior BLAST filtering at 98% sequence identity with a customized reference database
- Version numbers of compared tools (Magic-BLAST, CLARK-S, Centrifuge, Kraken 2, TaxMaps) are not stated, complicating exact benchmark reproduction
- eGap running time depends on the --trlcp k threshold chosen for truncating lcp values, which affects preprocessing cost

## Evidence

- [readme] OutputFile is the comma-separated file where the classification results will be stored according to the format: *C/U/A/H,IdSeqRead,TaxID,maxSim*: "OutputFile is the comma-separated file where the classification results will be stored according to the format: *C/U/A/H,IdSeqRead,TaxID,maxSim*"
- [methods] In Step 3, we perform the read assignment: every read is assigned to a specific taxon either at a given taxonomic level: "We perform the read assignment: every read is assigned to a specific taxon either at a given taxonomic level"
- [results] For the experiments reported in Figs. 2 and 3, we set the taxonomic rank of classification to species: "For the experiments reported in Figs. 2 and 3, we set the taxonomic rank of classification to species"
- [intro] we are able not only to classify the reads at a specific taxonomic rank such as genomes, or any level between species and phylum: "we are able not only to classify the reads at a specific taxonomic rank such as genomes, or any level between species and phylum"
- [methods] that approximately reach, with the entry R¯i, a similarity value equal to Mi (i.e., with tolerance of 0.02): [evidence span withheld — anchor status: not_found]
