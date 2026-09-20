---
name: cd-hit-similarity-threshold-parameterization
description: Use when building a nonredundant representative set from a curated functional-gene
  protein FASTA (e.g., amoA-AOB) and you need a data-driven similarity threshold rather
  than an arbitrary one.
when_to_use_negative:
- Input is nucleotide rather than protein sequence — the threshold optimization and
  the 96% cutoff were established on amino acid sequence similarity
- Input sequences are not yet curated (e.g., lack the FunGene HMM score >400 and HMM
  coverage over 70.2% filtering), so clustering would be applied to low-quality or
  truncated hits
- The goal is direct primer design rather than representative-sequence selection —
  use the downstream MetaFunPrimer/EcoFunPrimer steps instead
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: CD-HIT
  role: Clusters the curated amoA-AOB protein sequences at candidate similarity thresholds
    (80%–100%, 1% increments) to produce representative sequences per cluster
- name: MetaFunPrimer
  role: Pipeline that consumes the 96%-identity cluster representatives for downstream
    gene ranking and primer design; tutorial at https://metafunprimer.readthedocs.io/en/latest/
  repo: github.com/jialiu232/MetaFunPrimer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:3cfdd8a48cbef89d78d2ab6e8069386d6516ba6d54ccfbf78873091463de9239
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cd-hit-similarity-threshold-parameterization

## Summary

Systematically parameterizes CD-HIT identity thresholds (80%–100% in 1% increments) to select the clustering cutoff that maximizes resolution of functional-gene reference sets, as done for 1,205 curated FunGene amoA-AOB protein sequences in the MetaFunPrimer pipeline.

## When to use

Use when building a nonredundant representative set from a curated functional-gene protein FASTA (e.g., amoA-AOB) and you need a data-driven similarity threshold rather than an arbitrary one.

## When NOT to use

- Input is nucleotide rather than protein sequence — the threshold optimization and the 96% cutoff were established on amino acid sequence similarity
- Input sequences are not yet curated (e.g., lack the FunGene HMM score >400 and HMM coverage over 70.2% filtering), so clustering would be applied to low-quality or truncated hits
- The goal is direct primer design rather than representative-sequence selection — use the downstream MetaFunPrimer/EcoFunPrimer steps instead

## Inputs

- Curated protein sequences in FASTA format (e.g., 1,205 amoA-AOB reference proteins from Ribosomal Database Project Fungene version 9.6)
- Corresponding nucleotide sequences and nucleotide-to-protein ID mapping (for downstream cluster membership tracking)

## Outputs

- CD-HIT cluster representative FASTA (60 representatives for the amoA-AOB set at 96% identity)
- Cluster membership table (one row per input sequence with its cluster/representative assignment)
- Threshold scan table of cluster counts per similarity level (80%–100%, 1% increments) with first-order difference values

## How to apply

Run CD-HIT on the protein sequences across the range of 80% to 100% similarity in 1% increments, recording the number of unique clusters at each threshold. Compute the first-order difference (symmetric derivative) of the cluster count across thresholds and select the similarity cutoff that optimizes this criterion. In the MetaFunPrimer case study, clustering amoA-AOB sequences at greater than 96% similarity gave the largest increase in total unique clusters, so 96% amino acid sequence similarity was chosen; applying it to the 1,205 curated FunGene amoA-AOB proteins yielded 60 representative clusters. Record each cluster's representative sequence and build a membership table assigning every input sequence to its cluster.

## Related tools

- **CD-HIT** (Clusters the curated amoA-AOB protein sequences at candidate similarity thresholds (80%–100%, 1% increments) to produce representative sequences per cluster)
- **MetaFunPrimer** (Pipeline that consumes the 96%-identity cluster representatives for downstream gene ranking and primer design; tutorial at https://metafunprimer.readthedocs.io/en/latest/) — github.com/jialiu232/MetaFunPrimer

## Examples

```
cd-hit -i fungene_amoA_AOB_proteins.faa -o amoA_AOB_reps_96.faa -c 0.96 -n 5 -g 1 > cdhit_96.log
```

## Evaluation signals

- The number of distinct representative clusters at the chosen threshold equals 60 for the 1,205-sequence amoA-AOB input set
- The output CSV exists and contains exactly one row per input sequence, each with a cluster/representative assignment
- The selected threshold (96%) corresponds to the maximum first-order difference in unique cluster count across the 80%–100% scan
- Every cluster representative is a member of the original input FASTA (no synthetic or modified representative IDs)

## Limitations

- The first-order-difference criterion is a heuristic; for highly diverse functional genes the optimization may still yield numerous clusters requiring many primer pairs, which may exceed even HT-qPCR platform capacity
- Clustering threshold choice is context-specific — the 96% optimum was derived for amoA-AOB and may differ for other genes
- The scan is bounded to 80%–100% similarity with 1% increments; thresholds outside this range were not evaluated in the study

## Evidence

- [methods] CD-HIT scan range and increments: "CD-HIT (38, 39) is used to cluster sequences in the range of 80% to 100% (with 1% increments) similarity"
- [methods] Threshold optimization criterion: "a similarity threshold that optimizes the first-order difference, a criterion based on the symmetric derivative"
- [results] 96% threshold rationale: "clustering sequences at greater than 96% similarity resulted in the largest increase in the resulting total unique clusters"
- [results] 60 clusters outcome and downstream use: "The representatives of the 60 gene clusters found after clustering the input amoA-AOB sequences at 96% amino acid sequence similarity were aligned using MAFFT"
