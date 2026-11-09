---
name: hmm-based-rrna-gene-identification
description: Use when you need to detect SSU rRNA genes within large sequence collections
  (e.g. non-redundant SILVA releases or metagenome assemblies) so that structural
  features such as ≥10 bp insertion boundaries can be located from alignment gaps.
when_to_use_negative:
- Input sequences are already structurally aligned and curated — apply the gap-based
  insertion boundary detection directly instead of re-running database search
- The goal is ribosomal protein gene (rp16) phylogeny, which requires ≥1,000 aligned
  residues rather than SSU rRNA model search
- Sequences contain introns that have not been removed; all insertions ≥10 bp must
  be removed before multiple sequence alignment and phylogenetic analysis of 16S rRNA
  gene sequences
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Infernal (cmsearch, hmmonly mode)
  role: Scans every SILVA 115 record against the SSU rRNA covariance model at score
    threshold T 1 to call 16S rRNA gene sequences
- name: SSUAlign
  role: Provides the manually curated structural alignment of the 16S rRNA used as
    the search model
- name: Infernal (cmscan)
  role: Used to search full rRNA sequences against Rfam as part of rRNA identification
    and classification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:525611ed17d3847144360bb3542f2e5a6614ee4265467b35dc574e3931dd04a7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Infernal (cmscan)
  - Infernal (cmsearch)
  - Infernal (cmsearch, hmmonly mode)
  - SSUAlign
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# hmm-based-rrna-gene-identification

## Summary

Identifies 16S/SSU rRNA gene sequences in a nucleotide database by searching all records against a manually curated structural alignment model using Infernal cmsearch in hmmonly mode, enabling reproducible rRNA detection and downstream gap-based insertion analysis.

## When to use

Use when you need to detect SSU rRNA genes within large sequence collections (e.g. non-redundant SILVA releases or metagenome assemblies) so that structural features such as ≥10 bp insertion boundaries can be located from alignment gaps.

## When NOT to use

- Input sequences are already structurally aligned and curated — apply the gap-based insertion boundary detection directly instead of re-running database search
- The goal is ribosomal protein gene (rp16) phylogeny, which requires ≥1,000 aligned residues rather than SSU rRNA model search
- Sequences contain introns that have not been removed; all insertions ≥10 bp must be removed before multiple sequence alignment and phylogenetic analysis of 16S rRNA gene sequences

## Inputs

- Non-redundant SILVA version 115 bacterial 16S rRNA gene sequence records
- Manually curated structural alignment of the 16S rRNA distributed with SSUAlign
- Assembled and curated 16S rRNA gene sequences from metagenomes (≥800 bp aligned sequence)

## Outputs

- Set of records classified as 16S rRNA genes with cmsearch scores (T 1 threshold)
- Insertion coordinates, lengths, and lineage/family associations for each ≥10 bp gap relative to the covariance model alignment
- Database-wide prevalence of large insertions in 16S rRNA genes (e.g. Extended Data Fig. 4 / Supplementary Table 6 style survey)

## How to apply

Download the target sequence set — for the SILVA survey, version 115 of non-redundant SILVA bacterial 16S rRNA genes — and search every record against the manually curated structural alignment of the 16S rRNA provided with SSUAlign, using Infernal cmsearch in hmmonly mode with a score threshold of T 1. Build the model from a sequence-only version of the curated seed alignment maintained by the Comparative RNA Web so the same model applies to both study assemblies and the public database. For each hit, compare the sequence against the 16S rRNA covariance model alignment and call large gaps (≥10 bp) as insertion boundaries, mirroring the gap-based detection used on Rifle metagenome assemblies. Record each insertion's length and the lineage/family of the source sequence, then compute the prevalence of ≥10 bp insertions; correctness is judged by confirming the computed prevalence matches the reported survey (Extended Data Fig. 4, Supplementary Table 6) and contrasts with the 31% large-insertion rate in CPR organisms.

## Related tools

- **Infernal (cmsearch, hmmonly mode)** (Scans every SILVA 115 record against the SSU rRNA covariance model at score threshold T 1 to call 16S rRNA gene sequences)
- **SSUAlign** (Provides the manually curated structural alignment of the 16S rRNA used as the search model)
- **Infernal (cmscan)** (Used to search full rRNA sequences against Rfam as part of rRNA identification and classification)

## Examples

```
cmsearch --hmmonly -T 1 SSU.rRNA.cm silva_115_bacterial_16S.fasta > silva115_16s_hits.tblout
```

## Evaluation signals

- cmsearch hits are called at score threshold T 1 in hmmonly mode — re-running should reproduce the same set of 16S rRNA genes from SILVA 115
- Insertion prevalence computed from SILVA should match the reported survey in Extended Data Fig. 4 and Supplementary Table 6
- Computed database-wide insertion rate should be far below the 31% large (≥10 bp) insertion rate observed within CPR organisms, confirming rarity outside the CPR
- Gap-based insertion boundaries in SILVA sequences should mirror the boundary detection results obtained for the Rifle metagenome assemblies
- Only sequences with ≥800 bp of aligned 16S sequence should enter downstream phylogenetic analysis

## Limitations

- Owing to divergent 16S rRNA gene sequences, 50–100% of organisms sampled from specific CPR phyla would evade detection in typical cultivation-independent surveys, so model-based search can still miss highly divergent rRNA genes
- Insertion-encoded ORFs and Rfam hits require overlap criteria (≥25% of the alignment overlapped with an insertion for Rfam; ≥90% of the ORF overlapping the insertion) — hits below these cutoffs are not counted
- The Comparative RNA Web-derived model is a sequence-only model in this application; hmmonly mode bypasses full covariance model search, which may reduce sensitivity for structurally atypical rRNAs

## Evidence

- [methods] version 115 of nonredundant SILVA was analysed using the same methods: [evidence span withheld — anchor status: not_found]
- [methods] Rfam cmscan search: [evidence span withheld — anchor status: not_found]
