---
name: ambiguous-read-re-examination
description: Use when LiME's Classify step labels paired-end reads as Ambiguous (status
  'A') in the OutputFile and you need to convert those entries into single-taxon assignments
  (C/U) before computing species-level SEN/PREC/F1/specificity on datasets such as
  setA2/setB2.
when_to_use_negative:
- The read collection is single-end — the re-examination relies on mate-summed similarity
  (M' and M'') from paired-end reads, so no mate evidence exists to disambiguate the
  entry
- No reads were flagged Ambiguous ('A') by Classify — the second phase only considers
  ambiguous entries, so there is nothing to re-examine
- You want a fast first-pass taxonomic screen rather than final per-read assignments
  — the standard single-pass classification output (with 'A' entries retained) is
  sufficient
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: LiME
  role: The Classify executable (step 3 of the LiME method) performs the re-examination
    of ambiguous reads using paired-end mate similarity and the LineageFile at TaxRank=1
    (species); compiled with EBWT=1 (LiMEeBWT, reads da and ebwt) or EBWT=0 (LiMEDA,
    reads only da)
  repo: https://github.com/veronicaguerrini/LiME
- name: ClusterBWT_DA
  role: Produces the similarity values (beta-thresholded) over the four read/genome
    collections that the ambiguous-read re-examination consumes
  repo: https://github.com/veronicaguerrini/LiME
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:c18e94e09b451f06ebcee01d1c8968e9c0c0be221bcf485918986a3d97d8bea1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - CLARK-S
  - Centrifuge
  - ClusterBWT_DA
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGAP
  - eGSA
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ambiguous-read-re-examination

## Summary

Second-pass classification step of LiME that resolves reads flagged Ambiguous ('A') by re-examining them with paired-end mate information, using maximum mate-summed similarity (M' and M'') to assign each read pair to a single taxon (e.g. species) instead of leaving it ambiguous.

## When to use

Use when LiME's Classify step labels paired-end reads as Ambiguous (status 'A') in the OutputFile and you need to convert those entries into single-taxon assignments (C/U) before computing species-level SEN/PREC/F1/specificity on datasets such as setA2/setB2.

## When NOT to use

- The read collection is single-end — the re-examination relies on mate-summed similarity (M' and M'') from paired-end reads, so no mate evidence exists to disambiguate the entry
- No reads were flagged Ambiguous ('A') by Classify — the second phase only considers ambiguous entries, so there is nothing to re-examine
- You want a fast first-pass taxonomic screen rather than final per-read assignments — the standard single-pass classification output (with 'A' entries retained) is sufficient

## Inputs

- Similarity-matrix files produced by ClusterBWT_DA for the four merged collections Reads1F+Refs, Reads1RC+Refs, Reads2F+Refs and Reads2RC+Refs (built from setA2/setB2 paired-end reads and the 930-genome Refs.fasta via eGAP with --lcp and --da)
- LineageFile: semicolon-separated taxonomy file with format Accession_number;Species_TaxID;Genus_TaxID;Family_TaxID;Order_TaxID;Class_TaxID;Phylum_TaxID (derived from Reference_database.csv)
- Classify parameters: number of ClusterBWT_DA output files, total reads in S, number of genomes (930), TaxRank (1 = species), number of threads

## Outputs

- Comma-separated OutputFile with one entry per read in the format C/U/A/H,IdSeqRead,TaxID,maxSim (C=Classified, U=not classified, A=Ambiguous, H=classified at higher ranks if HIGHER=1)
- Final per-read-pair status labels usable to compute SEN, PREC, F1 and specificity and counts of ambiguous/unclassified reads at species level (accuracy_table.csv, supplementary Table S1 layout)

## How to apply

Run Classify with the similarity files produced by ClusterBWT_DA for all four collections (Reads1F+Refs, Reads1RC+Refs, Reads2F+Refs, Reads2RC+Refs, since strand direction is unknown and both mates' forward and reverse-complement structures are needed), passing the LineageFile in the format Accession_number;Species_TaxID;Genus_TaxID;Family_TaxID;Order_TaxID;Class_TaxID;Phylum_TaxID and TaxRank=1 for species. In the second phase, consider only the entries with status 'A' and compute the maximum mate-summed similarities M' and M'' for the read pair. Select the genomes that approximately reach, with the entry R_i, a similarity value equal to M_i with a tolerance of 0.02, and use their taxonomic lineage to re-assign the read to a unique taxon at the requested rank. Parse the resulting comma-separated OutputFile entries 'C/U/A/H,IdSeqRead,TaxID,maxSim' — after successful re-examination each read pair should carry exactly one final status label instead of 'A'. The rationale is that combining evidence from both mates (and both strands) supplies the discriminating signal that a single read's similarity vector lacks, which is what underpins LiME's very high precision and specificity (99.9% of positive control reads correctly assigned, <0.01% of negative control reads classified).

## Related tools

- **LiME** (The Classify executable (step 3 of the LiME method) performs the re-examination of ambiguous reads using paired-end mate similarity and the LineageFile at TaxRank=1 (species); compiled with EBWT=1 (LiMEeBWT, reads da and ebwt) or EBWT=0 (LiMEDA, reads only da)) — https://github.com/veronicaguerrini/LiME
- **ClusterBWT_DA** (Produces the similarity values (beta-thresholded) over the four read/genome collections that the ambiguous-read re-examination consumes) — https://github.com/veronicaguerrini/LiME

## Examples

```
./Classify 4 ReadsF+Refs.out1 ReadsF+Refs.out2 ReadsF+Refs.out3 ReadsF+Refs.out4 numReads 930 OutputFile LineageFile 1 4
```

## Evaluation signals

- After re-examination, every previously-'A' read pair carries exactly one final status label (C or U) in the OutputFile — one label per read pair, format 'C/U/A/H,IdSeqRead,TaxID,maxSim'
- Genomes retained for the re-assigned read are those within tolerance 0.02 of the maximum mate-summed similarity (M' or M'')
- Reproduced SEN/PREC/F1/specificity on setA2 and setB2 are consistent with Table S1: LiME shows very high precision and specificity, with 99.9% of positive control reads correctly assigned and less than 0.01% of negative control reads classified
- Results are invariant to the construction algorithm used for ebwt/da/lcp (BCR vs eGSA) and consistent across both LiME variants (LiMEeBWT with EBWT=1 and LiMEDA with EBWT=0)
- TaxIDs emitted in the OutputFile resolve correctly against the LineageFile at the chosen TaxRank (species TaxIDs for TaxRank=1)

## Limitations

- The tolerance of 0.02 on the maximum similarity and the beta threshold (0.25 used with ClusterBWT_DA) are fixed choices from the paper; reads whose top similarities are near-ties across many genomes may remain ambiguous or be misassigned
- The pairing evidence only exists for paired-end collections; for single-end data the disambiguation signal is unavailable
- If compiled with HIGHER=1 the output includes 'H' (classified at higher ranks) entries, so downstream metric parsing must handle the extra status code

## Evidence

- [methods] In this second phase, we only consider ambiguous entries.: "Re-examination of ambiguous reads– In this second phase, we only consider ambiguous entries."
- [methods] Genomes within tolerance 0.02 of the maximum similarity are selected.: [evidence span withheld — anchor status: not_found]
- [readme] Output format encodes C/U/A/H statuses per read.: "where C=Classified, U=not classified, A=Ambiguous, H=classified at higher ranks (if HIGHER=1)."
- [results] Species-level rank used for the benchmark experiments.: "For the experiments reported in Figs. 2 and 3, we set the taxonomic rank of classification to species"
