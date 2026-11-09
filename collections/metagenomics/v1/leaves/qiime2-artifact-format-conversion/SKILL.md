---
name: qiime2-artifact-format-conversion
description: Use when a reconstructed SSU sequences FASTA (e.g. metagenome_SSU_sequences.fasta
  from RiboTaxa's EMIRGE/MetaRib assembly) must be converted into a QIIME 2 .qza artifact
  before classify-sklearn classification.
when_to_use_negative:
- The input sequences are already a QIIME 2 artifact (.qza) — re-importing a FASTA
  conversion step is unnecessary
- 'The input is unassembled short Illumina reads: the article found that ''inferring
  taxa directly to short-length Illumina reads resulted in a high rate of false positives
  while reconstructed sequences were correctly assigned'', so reads should first be
  filtered and reconstructed before this conversion/classification step'
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2 Sklearn classifier
  role: The classify-sklearn plugin that consumes the converted .qza artifact and
    assigns taxonomy (domain to species) with confidence values at threshold 0.7
  repo: https://docs.qiime2.org/2020.8/plugins/available/feature-classifier/classify-sklearn/
- name: RiboTaxa
  role: Pipeline providing the reconstructed SSU sequences input FASTA and the RiboTaxa_py36
    environment that runs the QIIME 2 sklearn classifier on them
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:8ea182e8ba41d6f776b251b25062c06df625a93d2a4882f4234f404ae490a3dd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  tools:
  - QIIME 2 Sklearn classifier
  - RiboTaxa
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# qiime2-artifact-format-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts RiboTaxa-reconstructed full-length/nearly full-length SSU rRNA sequences from plain FASTA into a QIIME 2 feature-data artifact (.qza) so they can be taxonomically classified with QIIME 2's sklearn feature-classifier using the SILVA 138 trained classifier.

## When to use

Use when a reconstructed SSU sequences FASTA (e.g. metagenome_SSU_sequences.fasta from RiboTaxa's EMIRGE/MetaRib assembly) must be converted into a QIIME 2 .qza artifact before classify-sklearn classification.

## When NOT to use

- The input sequences are already a QIIME 2 artifact (.qza) — re-importing a FASTA conversion step is unnecessary
- The input is unassembled short Illumina reads: the article found that 'inferring taxa directly to short-length Illumina reads resulted in a high rate of false positives while reconstructed sequences were correctly assigned', so reads should first be filtered and reconstructed before this conversion/classification step

## Inputs

- metagenome_SSU_sequences.fasta — 97%-clustered reconstructed full-length/nearly full-length SSU rRNA gene sequences from RiboTaxa (EMIRGE + MetaRib assembly)
- silva-138-99-nb-classifier.qza — SILVA 138 reference classifier trained and downloaded from the Data resources of QIIME 2

## Outputs

- metagenome_SSU_sequences_qiime2.qza — QIIME 2 FeatureData[Sequence] artifact containing the reconstructed SSU sequences
- metagenome_renamed_16S18S_recons_qiime2_taxonomy.qza — taxonomy classification artifact from classify-sklearn
- metagenome_SSU_taxonomy_abundance.tsv — parsed output with columns Sequence_ID, Domain, Phylum, Class, Order, Family, Genus, Species, Confidence, Length(bp), Assigned reads, Relative_Abundance(%)

## How to apply

Take the reconstructed full-length SSU sequences file (metagenome_SSU_sequences.fasta, the 97%-clustered output of RiboTaxa's SSU reconstruction) and import it into QIIME 2 as a 'FeatureData[Sequence]' artifact, producing metagenome_SSU_sequences_qiime2.qza. This conversion is the prerequisite for running the sklearn feature-classifier plugin with the pre-trained SILVA 138 reference classifier (silva-138-99-nb-classifier.qza) at a confidence threshold of 0.7 for limiting taxonomic depth. RiboTaxa dedicates a dedicated python 3.6 environment (RiboTaxa_py36) to running the QIIME 2 sklearn classifier on the reconstructed rRNA sequences, separate from the python 2.7 environment used for quality control, filtering and reconstruction. Use batch mode 'auto' (20,000 sequences per batch); set batch=1 if less than 16 GB RAM is available. Export the resulting taxonomy artifact (metagenome_renamed_16S18S_recons_qiime2_taxonomy.qza) and parse the taxonomy ranks (domain to species) with the confidence value per sequence.

## Related tools

- **QIIME 2 Sklearn classifier** (The classify-sklearn plugin that consumes the converted .qza artifact and assigns taxonomy (domain to species) with confidence values at threshold 0.7) — https://docs.qiime2.org/2020.8/plugins/available/feature-classifier/classify-sklearn/
- **RiboTaxa** (Pipeline providing the reconstructed SSU sequences input FASTA and the RiboTaxa_py36 environment that runs the QIIME 2 sklearn classifier on them) — https://github.com/oschakoory/RiboTaxa

## Examples

```
qiime tools import --input-path metagenome_SSU_sequences.fasta --type 'FeatureData[Sequence]' --output-path metagenome_SSU_sequences_qiime2.qza
```

## Evaluation signals

- The output metagenome_SSU_sequences_qiime2.qza is a valid QIIME 2 artifact importable/inspectable with 'qiime tools' and contains the same sequence IDs as the input reconstructed FASTA
- The exported taxonomy artifact yields a lineage for every reconstructed sequence with a Confidence value at or above the 0.7 threshold
- The combined TSV contains all twelve named columns (Sequence_ID through Relative_Abundance(%))
- Relative abundances are percentages in the range 0–100 and sum to ~100 across all reconstructed sequences
- Batch mode resolves to 'auto' (20,000 sequences per batch) on machines with ≥16 GB RAM, or batch=1 on smaller-memory machines without failure

## Limitations

- The confidence threshold of 0.7 limits taxonomic depth; RiboTaxa could not assign certain sequences at species level and stopped at genus level where 16S rRNA genes are near-identical (e.g. certain Enterobacteriaceae, Lachnospiraceae genera)
- Batch mode 'auto' loads 20,000 sequences per batch and requires substantial memory — set batch=1 if less than 16 GB RAM is available
- Occasional small indel errors in reconstructed sequences can occur, though in practice these rare indels have little effect on taxonomic classification; no chimera detection is included in the reconstruction workflow

## Evidence

- [methods] Take the reconstructed full-length/nearly full-length SSU sequences file metagenome_SSU_sequences.fasta and convert it to a QIIME 2 artifact (metagenome_SSU_sequences_qiime2.qza).: [evidence span withheld — anchor status: not_found]
- [methods] Run the sklearn feature-classifier plugin of QIIME 2 with the trained SILVA 138 reference classifier (silva-138-99-nb-classifier.qza) at a confidence threshold of 0.7: [evidence span withheld — anchor status: not_found]
- [methods] the database used is the trained classifier SILVA 138 reference sequence downloaded from the Data resources of Qiime2: [evidence span withheld — anchor status: not_found]
