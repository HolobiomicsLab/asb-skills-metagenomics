---
name: cluster-count-verification-against-reported-results
description: Use when a published pipeline reports a specific number of sequence clusters
  at a stated identity threshold and you must independently reproduce and validate
  that cluster count from the same curated input set.
when_to_use_negative:
- Input sequences are not the curated amoA-AOB reference set (e.g., raw unfiltered
  FunGene hits or metagenome reads), since the 60-cluster count was reported only
  for the filtered 1,205-sequence input
- The goal is de novo threshold selection rather than reproducing a published count
  — use the first-order difference optimization scan over 80–100% instead
- Clustering nucleotide rather than protein sequences — the 96% threshold and 60-cluster
  result apply to amino acid similarity
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: CD-HIT
  role: Clusters the curated amoA-AOB protein sequences at 96% similarity to produce
    representative gene clusters
- name: MetaFunPrimer
  role: Pipeline whose reference-gene clustering step (representative selection) this
    skill reproduces and verifies
  repo: github.com/jialiu232/MetaFunPrimer
- name: MAFFT
  role: Downstream alignment of the 60 verified cluster representatives in the MetaFunPrimer
    workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:bf78986493f12e2efec109a5ef3de78cf3d7327ace86ea87106a3069439dde61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - MAFFT
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Cluster count verification against reported results

## Summary

Reproduces a reported sequence-clustering result — clustering 1,205 curated FunGene amoA-AOB protein sequences with CD-HIT at 96% amino acid similarity and verifying that exactly 60 representative gene clusters are obtained, as described in the MetaFunPrimer pipeline.

## When to use

Use when a published pipeline reports a specific number of sequence clusters at a stated identity threshold and you must independently reproduce and validate that cluster count from the same curated input set.

## When NOT to use

- Input sequences are not the curated amoA-AOB reference set (e.g., raw unfiltered FunGene hits or metagenome reads), since the 60-cluster count was reported only for the filtered 1,205-sequence input
- The goal is de novo threshold selection rather than reproducing a published count — use the first-order difference optimization scan over 80–100% instead
- Clustering nucleotide rather than protein sequences — the 96% threshold and 60-cluster result apply to amino acid similarity

## Inputs

- 1,205 curated amoA-AOB protein sequences from FunGene v9.6 (HMM score >400, HMM coverage >70.2%), in FASTA format
- Corresponding amoA-AOB nucleotide sequences
- Nucleotide-to-protein sequence ID mapping table

## Outputs

- CD-HIT cluster representative sequences (60 representatives expected)
- Membership table (CSV) assigning each of the 1,205 input sequences to its cluster representative

## How to apply

Load the 1,205 curated amoA-AOB protein sequences obtained from the Ribosomal Database Project Fungene (version 9.6), which were filtered for HMM search score >400 and HMM coverage over 70.2% amino acid similarity, together with their nucleotide sequences and nucleotide-to-protein ID mapping. Run CD-HIT on the protein sequences at the 96% amino acid sequence similarity threshold — the threshold the study identified via the first-order difference criterion as giving the largest increase in total unique clusters. Record the CD-HIT representative sequence for each cluster and assign every input sequence to its cluster to build a membership table. Validate by confirming the output contains one row per input sequence with a cluster/representative assignment and that the number of distinct representative clusters equals 60, matching the reported result; the 60 cluster representatives were subsequently aligned with MAFFT in the MetaFunPrimer workflow.

## Related tools

- **CD-HIT** (Clusters the curated amoA-AOB protein sequences at 96% similarity to produce representative gene clusters)
- **MetaFunPrimer** (Pipeline whose reference-gene clustering step (representative selection) this skill reproduces and verifies) — github.com/jialiu232/MetaFunPrimer
- **MAFFT** (Downstream alignment of the 60 verified cluster representatives in the MetaFunPrimer workflow)

## Examples

```
cd-hit -i amoA_AOB_proteins.fasta -o amoA_AOB_clusters_96.fasta -c 0.96 -n 5 -d 0
```

## Evaluation signals

- The output CSV exists and contains exactly one row per input sequence (1,205 rows) with a cluster/representative assignment
- The number of distinct representative clusters equals 60, matching the value reported in the article
- Every input sequence ID in the membership table maps to exactly one cluster representative
- The CD-HIT run used a 96% amino acid similarity threshold (the identified optimum for largest increase in unique clusters)
- Representative sequence IDs in the output correspond to members of the curated input FASTA (no invented representatives)

## Limitations

- The expected count of 60 clusters holds only for the specific curated FunGene v9.6 amoA-AOB input (HMM score >400, coverage >70.2%) and the 96% protein similarity threshold; other inputs or thresholds yield different counts
- CD-HIT is used in the study across 80–100% similarity in 1% increments, so a different increment setting changes the number of resulting clusters

## Evidence

- [results] 60 gene clusters from 96% clustering of amoA-AOB input: "The representatives of the 60 gene clusters found after clustering the input amoA-AOB sequences at 96% amino acid sequence similarity were aligned using MAFFT"
- [methods] CD-HIT similarity range used: "CD-HIT (38, 39) is used to cluster sequences in the range of 80% to 100% (with 1% increments) similarity"
- [results] Rationale for the 96% threshold: "clustering sequences at greater than 96% similarity resulted in the largest increase in the resulting total unique clusters"
- [methods] FunGene filtering criteria for the input set: "were obtained from the Fungene database (32) by filtering for a Hidden Markov Model (HMM) search score of >400 and HMM coverage over 70.2% amino acid similarity"
- [results] Reference-gene representative selection step: "Representative sequences of reference genes were first selected by clustering sequences based on their amino acid sequence similarity"
