---
name: cluster-representative-sequence-selection
description: Use when a reference gene set (FASTA protein sequences with matching
  nucleotides) must be reduced to non-redundant representative clusters before aligning
  to metagenomes or designing primers, and a similarity threshold must be chosen.
when_to_use_negative:
- Input sequences are already non-redundant representatives or the goal is primer
  scoring rather than de-replication
- The sequences are unassembled reads or metagenomic contigs rather than a curated
  reference gene set
- Nucleotide-only clustering is needed at DNA identity levels where CD-HIT amino acid
  thresholds are not meaningful
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CD-HIT
  role: Clusters the input amino acid sequences at 80–100% similarity (1% increments)
    and emits cluster representatives and memberships
- name: MetaFunPrimer
  role: Pipeline embedding the clustering step; consumes the representative sequences
    for MAFFT alignment, NGPhylogeny.fr tree construction, and R-score ranking against
    metagenomes
  repo: github.com/jialiu232/MetaFunPrimer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:ab2e4a8b60dc190e507196a6aa778fcfe81347b28d882abb4411a2d32c1b886e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cluster-representative-sequence-selection

## Summary

Clusters a curated set of functional-gene protein sequences (e.g., 1,205 FunGene amoA-AOB sequences) at a chosen amino acid identity threshold with CD-HIT to select representative sequences for each cluster, yielding one representative per gene cluster for downstream ranking and primer design.

## When to use

Use when a reference gene set (FASTA protein sequences with matching nucleotides) must be reduced to non-redundant representative clusters before aligning to metagenomes or designing primers, and a similarity threshold must be chosen.

## When NOT to use

- Input sequences are already non-redundant representatives or the goal is primer scoring rather than de-replication
- The sequences are unassembled reads or metagenomic contigs rather than a curated reference gene set
- Nucleotide-only clustering is needed at DNA identity levels where CD-HIT amino acid thresholds are not meaningful

## Inputs

- Curated functional-gene protein sequences in FASTA format (e.g., 1,205 amoA-AOB sequences from FunGene v9.6)
- Corresponding nucleotide sequences with nucleotide-to-protein ID mapping
- Chosen amino acid similarity threshold (e.g., 96%)

## Outputs

- FASTA of representative sequences (60 clusters for the 1,205-sequence amoA-AOB set at 96%)
- Cluster membership CSV mapping every input sequence to its cluster/representative
- Cluster count summary (number of distinct representative clusters at each threshold tested)

## How to apply

Load the curated protein FASTA (e.g., 1,205 amoA-AOB sequences from Fungene v9.6, pre-filtered for HMM score >400 and HMM coverage >70.2%) together with the nucleotide-to-protein ID mapping. Run CD-HIT at the selected amino acid similarity threshold — the MetaFunPrimer case study scanned 80–100% in 1% increments and chose 96% because clustering above 96% similarity gave the largest increase in total unique clusters. Record each cluster's representative sequence and assign every input sequence to its cluster to build a membership table (CSV, one row per input sequence). For the amoA-AOB input set at 96%, verify that exactly 60 distinct representative clusters are produced; those 60 representatives are then aligned (e.g., MAFFT) and ranked by R-score against reference metagenomes.

## Related tools

- **CD-HIT** (Clusters the input amino acid sequences at 80–100% similarity (1% increments) and emits cluster representatives and memberships)
- **MetaFunPrimer** (Pipeline embedding the clustering step; consumes the representative sequences for MAFFT alignment, NGPhylogeny.fr tree construction, and R-score ranking against metagenomes) — github.com/jialiu232/MetaFunPrimer

## Examples

```
cd-hit -i amoA_AOB_proteins.fasta -o amoA_AOB_reps96.fasta -c 0.96 -d 0
```

## Evaluation signals

- Number of distinct representative clusters equals the reported value (60 for 1,205 amoA-AOB sequences at 96% amino acid similarity)
- Output CSV exists with exactly one row per input sequence, each carrying a cluster/representative assignment
- Every cluster contains exactly one designated representative sequence
- Threshold sensitivity check: clustering above 96% similarity should show the largest increase in total unique clusters (first-order difference criterion)
- All input sequence IDs are accounted for in the membership table (no sequences dropped or duplicated)

## Limitations

- Highly diverse functional genes will fragment into many clusters at high identity thresholds, inflating the number of downstream primer pairs and qPCR runs
- The optimal similarity threshold depends on the gene set; 96% is not universal and should be re-derived via the first-order-difference criterion for other genes
- Representatives are only as good as the upstream curation (HMM score >400 and HMM coverage >70.2% filtering of the Fungene source)

## Evidence

- [methods] CD-HIT clusters amoA-AOB at 96% into 60 clusters: "CD-HIT (38, 39) is used to cluster sequences in the range of 80% to 100% (with 1% increments) similarity"
- [results] 96% threshold yields largest increase in unique clusters: "clustering sequences at greater than 96% similarity resulted in the largest increase in the resulting total unique clusters"
- [results] 60 representatives aligned with MAFFT: "The representatives of the 60 gene clusters found after clustering the input amoA-AOB sequences at 96% amino acid sequence similarity were aligned using MAFFT"
- [methods] Fungene curation filter precedes clustering: "filtering for a Hidden Markov Model (HMM) search score of >400 and HMM coverage over 70.2% amino acid similarity"
- [results] Representatives selected by amino acid similarity clustering: "Representative sequences of reference genes were first selected by clustering sequences based on their amino acid sequence similarity"
