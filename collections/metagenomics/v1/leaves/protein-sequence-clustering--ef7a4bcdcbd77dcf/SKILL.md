---
name: protein-sequence-clustering
description: Use when a curated reference set of protein sequences for a functional
  gene (e.g., FunGene amoA-AOB) must be reduced to representative gene clusters before
  alignment to metagenomes or degenerate primer design.
when_to_use_negative:
- Input is a nucleotide rather than amino acid sequence set — the article clusters
  sequences based on amino acid sequence similarity
- Input sequences are already representatives from a prior clustering and the goal
  is primer design rather than redundancy reduction
- The gene set is too small or too conserved for threshold optimization (first-order
  difference) to be meaningful
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CD-HIT
  role: Clusters the curated amino acid sequences at 80%–100% similarity (1% increments);
    run at 96% to produce representative gene clusters
- name: MetaFunPrimer
  role: Pipeline that consumes the cluster representatives for downstream alignment,
    R-score ranking, and primer design
  repo: github.com/jialiu232/MetaFunPrimer_paper_info
- name: MAFFT
  role: Aligns the representatives of the 60 clusters after clustering for tree construction
    and downstream use
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:f868470c835796fbc69ea0092fcadd0e18362ebcc027ac4daff9965467310360
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - MAFFT
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# protein-sequence-clustering

## Summary

Cluster protein sequences of a functional gene of interest with CD-HIT at a user-optimized amino acid similarity threshold (e.g., 96% for amoA-AOB) to select representative sequences for downstream primer design in MetaFunPrimer. The threshold is chosen where the first-order difference in total unique clusters is largest, and each cluster's representative anchors subsequent alignment, ranking (R-score), and primer design.

## When to use

Use when a curated reference set of protein sequences for a functional gene (e.g., FunGene amoA-AOB) must be reduced to representative gene clusters before alignment to metagenomes or degenerate primer design.

## When NOT to use

- Input is a nucleotide rather than amino acid sequence set — the article clusters sequences based on amino acid sequence similarity
- Input sequences are already representatives from a prior clustering and the goal is primer design rather than redundancy reduction
- The gene set is too small or too conserved for threshold optimization (first-order difference) to be meaningful

## Inputs

- Curated protein FASTA of the target functional gene (e.g., 1,205 amoA-AOB sequences from FunGene v9.6, filtered for HMM score >400 and HMM coverage >70.2%)
- Corresponding nucleotide FASTA and nucleotide-to-protein ID mapping

## Outputs

- CD-HIT cluster representatives (one per cluster; 60 for amoA-AOB at 96%)
- Membership table mapping every input sequence to its cluster/representative (CSV, one row per input sequence)

## How to apply

Load the curated protein FASTA (e.g., 1,205 FunGene amoA-AOB sequences) together with the corresponding nucleotide sequences and ID mapping. Scan CD-HIT across 80%–100% similarity in 1% increments and pick the threshold via the first-order difference (symmetric derivative) criterion; for amoA-AOB this is 96%, where clustering at greater than 96% similarity gave the largest increase in total unique clusters. Run CD-HIT at the chosen threshold, record each cluster's representative sequence, and build a membership table assigning every input sequence to its cluster. Verify the expected cluster count (60 for the amoA-AOB set) and that every input row has a representative assignment. Representatives are then aligned with MAFFT and screened against 1,550 soil metagenomes (97% identity over reference length) to compute R-scores for gene ranking.

## Related tools

- **CD-HIT** (Clusters the curated amino acid sequences at 80%–100% similarity (1% increments); run at 96% to produce representative gene clusters)
- **MetaFunPrimer** (Pipeline that consumes the cluster representatives for downstream alignment, R-score ranking, and primer design) — github.com/jialiu232/MetaFunPrimer_paper_info
- **MAFFT** (Aligns the representatives of the 60 clusters after clustering for tree construction and downstream use)

## Examples

```
cd-hit -i amoA_AOB_proteins.fasta -o amoA_AOB_clusters96 -c 0.96 -n 5
```

## Evaluation signals

- Number of distinct representative clusters equals 60 when the 1,205-sequence amoA-AOB set is clustered at 96% similarity
- Output CSV exists, contains exactly one row per input sequence, and every row has a non-empty cluster/representative assignment
- Cluster count at thresholds >96% shows the largest increase in total unique clusters, confirming the threshold choice
- Each representative sequence is a member of its own cluster and matches one input protein ID
- Downstream, representatives align to the 1,550 soil metagenomes at ≥97% identity over the reference length for R-score computation

## Limitations

- A higher similarity threshold can drastically inflate cluster count, requiring more primer pairs and qPCR runs than HT-qPCR platforms can accommodate
- Highly diverse functional genes may produce numerous clusters/primer pairs, exceeding practical assay limits
- Threshold choice is input-set dependent; the 96% optimum for amoA-AOB does not transfer to other genes

## Evidence

- [methods] CD-HIT threshold and range: "CD-HIT (38, 39) is used to cluster sequences in the range of 80% to 100% (with 1% increments) similarity"
- [results] amoA-AOB 96% threshold rationale: "clustering sequences at greater than 96% similarity resulted in the largest increase in the resulting total unique clusters"
- [results] 60 clusters aligned with MAFFT: "The representatives of the 60 gene clusters found after clustering the input amoA-AOB sequences at 96% amino acid sequence similarity were aligned using MAFFT"
- [methods] Threshold optimization criterion: "a similarity threshold that optimizes the first-order difference, a criterion based on the symmetric derivative"
