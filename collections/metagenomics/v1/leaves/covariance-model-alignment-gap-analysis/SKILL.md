---
name: covariance-model-alignment-gap-analysis
description: Use when screening assembled or database 16S rRNA gene sequences (e.g.
  SILVA 115 records or metagenome assemblies) for large insertion sequences, introns,
  or insertion-encoded ORFs whose boundaries must be located relative to a structural
  rRNA alignment.
when_to_use_negative:
- The input is already intron-removed 16S sequence intended for alignment/phylogeny
  (insertions ≥10 bp should already have been removed before multiple sequence alignment)
- The query is metatranscriptomic (RNA) rather than gene sequence — insertions are
  not retained in transcribed RNAs, so gap analysis of transcripts will miss introns
- The goal is general 16S taxonomic classification rather than detection of introns
  or insertion-encoded ORFs
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Infernal (cmsearch)
  role: Aligns each SILVA/assembly 16S rRNA sequence against the 16S rRNA covariance
    model in hmmonly mode with score threshold T 1; alignment gaps ≥10 bp define insertion
    boundaries
- name: SSUAlign
  role: Provides the manually curated structural alignment of the 16S rRNA used as
    the search target/reference model for identifying 16S sequences and insertion
    positions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:d694f5d60c2c44b133fcd9d3b238c82b195eae96ec45c4957d2bda9eb35e58b7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Infernal (cmsearch)
  - SSUAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# covariance-model-alignment-gap-analysis

## Summary

Detects large (≥10 bp) insertions in bacterial 16S rRNA genes by aligning sequences to a 16S rRNA covariance model with Infernal/SSUAlign and calling insertion boundaries from alignment gaps, enabling quantification of insertion prevalence (e.g. 31% in CPR vs rarity outside CPR) in databases such as non-redundant SILVA 115.

## When to use

Use when screening assembled or database 16S rRNA gene sequences (e.g. SILVA 115 records or metagenome assemblies) for large insertion sequences, introns, or insertion-encoded ORFs whose boundaries must be located relative to a structural rRNA alignment.

## When NOT to use

- The input is already intron-removed 16S sequence intended for alignment/phylogeny (insertions ≥10 bp should already have been removed before multiple sequence alignment)
- The query is metatranscriptomic (RNA) rather than gene sequence — insertions are not retained in transcribed RNAs, so gap analysis of transcripts will miss introns
- The goal is general 16S taxonomic classification rather than detection of introns or insertion-encoded ORFs

## Inputs

- Non-redundant SILVA version 115 bacterial 16S rRNA gene sequences (FASTA)
- Assembled and curated 16S rRNA gene sequences (≥800 bp aligned) from metagenome assemblies
- 16S rRNA covariance model built from the Comparative RNA Web manually curated seed alignment

## Outputs

- Per-sequence list of insertion boundaries defined by ≥10 bp alignment gaps
- Insertion length distribution and prevalence statistics (e.g. % of 16S rRNA genes with ≥10 bp insertions)
- Lineage/family associations for detected insertions, including Rfam classifications of insertion sequences

## How to apply

First build a sequence-only covariance model from the manually curated Comparative RNA Web seed alignment of the 16S rRNA, then search every record (e.g. non-redundant SILVA version 115 bacterial sequences) against the model using Infernal cmsearch in hmmonly mode with a score threshold of T 1. Scan the resulting covariance-model alignments for gaps of ≥10 bp between the query sequence and the model; each such gap defines an insertion boundary. Record each insertion's length and the lineage/family association of the source sequence, then compute the prevalence of ≥10 bp insertions across the database and the length distribution. Interpret hits carefully: insertions should be classified against Rfam using the ≥25% alignment-overlap criterion, and encoded ORFs require ≥90% overlap with the insertion to count as insertion-encoded. Validate the result by comparing the computed prevalence and length statistics against the known benchmark (31% of CPR 16S rRNA genes carry ≥10 bp insertions, mean 519 bp, s.d. 372 bp, max 2,004 bp) and confirming rarity outside the CPR.

## Related tools

- **Infernal (cmsearch)** (Aligns each SILVA/assembly 16S rRNA sequence against the 16S rRNA covariance model in hmmonly mode with score threshold T 1; alignment gaps ≥10 bp define insertion boundaries)
- **SSUAlign** (Provides the manually curated structural alignment of the 16S rRNA used as the search target/reference model for identifying 16S sequences and insertion positions)

## Examples

```
cmsearch --hmmonly -T 1 16S_rRNA.cm silva115_bacterial_16s.fasta > silva115_cmsearch.out
```

## Evaluation signals

- All detected insertions are ≥10 bp gaps in the covariance-model alignment; no boundaries are called from gaps below this threshold
- Computed SILVA-wide insertion prevalence is far below the 31% rate observed within CPR, consistent with reported rarity outside the CPR
- Insertion length distribution matches the reported statistics: maximum 2,004 bp, mean 519 bp, standard deviation 372 bp
- Every Rfam-positive insertion hit has ≥25% of its alignment overlapping the insertion; insertion-encoded ORFs have ≥90% overlap with the insertion
- Removal of detected insertions before multiple sequence alignment yields 16S rRNA sequences that align cleanly for phylogenetic analysis

## Limitations

- Divergent 16S rRNA gene sequences mean 50–100% of organisms in some CPR phyla evade detection in typical cultivation-independent surveys, so database-based prevalence estimates can undercount insertion-bearing lineages
- The method applies to gene sequences, not transcripts: mapping metatranscriptomic reads showed insertions are not retained in transcribed RNAs and are probably rapidly degraded
- Assembly errors can masquerade as insertions in metagenome assemblies, so 16S rRNA genes must first be curated to eliminate assembly errors before gap calling

## Evidence

- [methods] SILVA 115 non-redundant analysed with the same gap-based methods: [evidence span withheld — anchor status: not_found]
