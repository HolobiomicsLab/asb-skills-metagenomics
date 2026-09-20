---
name: classification-runtime-measurement
description: Use when comparing the RAM and runtime footprint of metagenomic classifiers
  (LiME vs Magic-BLAST, TaxMaps, CLARK-S, Centrifuge, Kraken 2) on the same read sample,
  e.g. the simulated setA2 dataset.
when_to_use_negative:
- The goal is classification accuracy evaluation (sensitivity, precision, F1, specificity)
  rather than computational cost — use the classification-quality benchmarking workflow
  instead
- The input is a real metagenome such as SRR1804065 where concordance is assessed
  via Jaccard-based agreement rates (rid/ras) rather than a controlled runtime/memory
  comparison on simulated data
- Reference database or read sample differs across tools — timing comparisons are
  only meaningful when all tools classify the same fixed sample (setA2)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: 'Classifier under benchmarking: run ClusterLCP, ClusterBWT_DA, and Classify
    on setA2 while measuring wall-time and internal memory'
  repo: https://github.com/veronicaguerrini/LiME
- name: eGSA
  role: Builds ebwt, lcp, and da data structures for the read set and genome set separately
    before merging
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges the two ebwts of read set R and genome set G with --lcp and --da options;
    peak construction memory recorded
  repo: https://github.com/felipelouza/egap
- name: MagicBlast
  role: State-of-the-art comparison tool run on setA2 for memory and wall-time measurement
- name: CLARK-S
  role: State-of-the-art comparison tool whose database-construction memory is contrasted
    with FM-index-based tools
- name: Centrifuge
  role: FM-index-based comparison tool run on setA2 for resource measurement
- name: Kraken 2
  role: Comparison tool run on setA2 for resource measurement
- name: TaxMaps
  role: Comparison tool run on setA2 for resource measurement
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:869392d018018198c907332a5c403a3c566110f77c52a6fc5c8e655569c3981b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGSA
  - eGap
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# classification-runtime-measurement

## Summary

Benchmark the computational cost (peak database-construction memory and classification wall-time) of LiME against state-of-the-art taxonomic classifiers on a fixed simulated metagenome sample, producing a per-tool resource comparison table mirroring Additional file 1 Table S3.

## When to use

Use when comparing the RAM and runtime footprint of metagenomic classifiers (LiME vs Magic-BLAST, TaxMaps, CLARK-S, Centrifuge, Kraken 2) on the same read sample, e.g. the simulated setA2 dataset.

## When NOT to use

- The goal is classification accuracy evaluation (sensitivity, precision, F1, specificity) rather than computational cost — use the classification-quality benchmarking workflow instead
- The input is a real metagenome such as SRR1804065 where concordance is assessed via Jaccard-based agreement rates (rid/ras) rather than a controlled runtime/memory comparison on simulated data
- Reference database or read sample differs across tools — timing comparisons are only meaningful when all tools classify the same fixed sample (setA2)

## Inputs

- Simulated paired-end metagenome sample setA2 (100 bp reads, e.g. setA2_1.fasta / setA2_2.fasta)
- Reference database of 930 genomes (Refs.fasta) with taxonomic information (Reference_database.csv)
- eBWT data structures ebwt, lcp, and da for the collection S (built with eGSA and merged with eGap --lcp --da)

## Outputs

- Aggregated per-tool resource comparison table (memory and wall-time per tool on setA2), e.g. table_s3_resource_comparison.csv mirroring Additional file 1 Table S3
- Peak memory measurements for LiME preprocessing (eGSA/eGap) and the three classification steps (ClusterLCP, ClusterBWT_DA, Classify)
- Database-construction memory and classification wall-time measurements for each comparison tool (Magic-BLAST, CLARK-S, Centrifuge, Kraken 2, TaxMaps)

## How to apply

First build the eBWT data structures (ebwt, lcp, da) for the setA2 collection by running eGSA separately on the read set and the 930-genome reference set, then merging with eGap using --lcp and --da (optionally --trlcp k), recording peak memory during construction. Run the three LiME steps on setA2 — ClusterLCP with alpha=16, ClusterBWT_DA with beta=0.25, and Classify at the species-level TaxRank as in the 100 bp paired-end configuration — measuring wall-time and internal memory held by the similarity matrix and auxiliary structures. Run Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps on the same setA2 sample, recording each tool's database-construction memory and classification wall-time. Aggregate per-tool memory and timing measurements into a single comparison table mirroring Additional file 1 Table S3 (e.g. table_s3_resource_comparison.csv). Rationale: LiME requires only sequential scans of ebwt/lcp/da rather than keeping ad-hoc in-memory reference database structures, so the expected outcome is comparable accuracy at lower computational cost.

## Related tools

- **LiME** (Classifier under benchmarking: run ClusterLCP, ClusterBWT_DA, and Classify on setA2 while measuring wall-time and internal memory) — https://github.com/veronicaguerrini/LiME
- **eGSA** (Builds ebwt, lcp, and da data structures for the read set and genome set separately before merging) — https://github.com/felipelouza/egsa
- **eGap** (Merges the two ebwts of read set R and genome set G with --lcp and --da options; peak construction memory recorded) — https://github.com/felipelouza/egap
- **MagicBlast** (State-of-the-art comparison tool run on setA2 for memory and wall-time measurement)
- **CLARK-S** (State-of-the-art comparison tool whose database-construction memory is contrasted with FM-index-based tools)
- **Centrifuge** (FM-index-based comparison tool run on setA2 for resource measurement)
- **Kraken 2** (Comparison tool run on setA2 for resource measurement)
- **TaxMaps** (Comparison tool run on setA2 for resource measurement)

## Examples

```
Preprocessing.sh   # then: ./ClusterLCP ReadsF+Refs.fasta numReads numRefs 16 threads ; ./ClusterBWT_DA ReadsF+Refs.fasta length 0.25 threads
```

## Evaluation signals

- The output table contains exactly one memory entry and one wall-time entry per tool on setA2
- Database memory of CLARK-S exceeds that of the FM-index-based tools (LiME, Centrifuge, TaxMaps), consistent with Additional file 1 Table S3
- Classification speed ranking across tools is consistent with the ordering reported in Additional file 1 Table S3
- LiME preprocessing and classification memory reflects sequential scans of ebwt/lcp/da (no in-memory reference database), matching the paper's lightweight claim
- All tools were run on the identical fixed setA2 sample with the same reference database of 930 genomes

## Limitations

- Full computational performance results for setA2 are deferred to supplementary Table S3 and are not included in the main text
- Version numbers of the compared tools (Magic-BLAST, CLARK-S, Centrifuge, Kraken 2, TaxMaps) are not stated, complicating exact reproduction
- Finding the best method for building the ebwt/lcp/da data structures is explicitly out of scope of the paper, so preprocessing cost depends on the chosen construction tools
- The best construction method is not established; classification does not depend on the construction algorithm used for ebwt, da, and lcp

## Evidence

- [intro] Resource comparison motivation: "LiME does not need to build ad-hoc and keep in internal memory the data structures relating to the database of the reference genomes, as it only needs to run sequential scans"
- [methods] Parameter settings for LiME steps: [evidence span withheld — anchor status: not_found]
- [discussion] Resource results location: "Full results for the dataset setA2 are available in Additional file 1: Table S3."
- [discussion] Comparison tool set: "Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps, which have recently been introduced."
- [methods] Reference database composition: "The database Refs.fasta is the fasta file of reference genomes (number of genomes: 930)"
- [methods] eGap merge usage: "To merge the two ebwts associated with the sets R and G, one could use eGap"
- [readme] Merge options in README: "one could use eGap [https://github.com/felipelouza/egap] and set both options --lcp and --da to have the lcp and da computed"
