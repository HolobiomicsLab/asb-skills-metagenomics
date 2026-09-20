---
name: memory-footprint-profiling
description: Use when comparing computational cost (peak RAM during database/index
  construction and classification runtime) of taxonomic classifiers on an identical
  simulated dataset such as setA2, mirroring Additional file 1 Table S3.
when_to_use_negative:
- The goal is classification accuracy (sensitivity/precision/F1) rather than resource
  comparison — use the accuracy-evaluation workflow instead
- Real metagenome data (e.g. SRR1804065) is used where full resource numbers were
  deferred to Additional file 1 and concordance (rid/ras Jaccard) is the primary metric
- 'Evaluating thread scaling of LiME — this is covered separately by Additional file
  1: Table S4'
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: LiME
  role: subject classifier whose memory and wall-time are profiled via ClusterLCP,
    ClusterBWT_DA and Classify
  repo: https://github.com/veronicaguerrini/LiME
- name: eGSA
  role: builds ebwt, lcp, da for read set and genome set separately during preprocessing
    whose peak memory is recorded
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: merges the two ebwts of R and G with --lcp and --da options; its memory is
    capped (e.g. 4096 MB) and construction memory recorded
  repo: https://github.com/felipelouza/egap
- name: MagicBlast
  role: 'comparison tool: database memory and classification wall-time measured on
    setA2'
- name: CLARK-S
  role: 'comparison tool: database memory and classification wall-time measured on
    setA2'
- name: Centrifuge
  role: 'comparison tool: database memory and classification wall-time measured on
    setA2'
- name: Kraken 2
  role: 'comparison tool: database memory and classification wall-time measured on
    setA2'
- name: TaxMaps
  role: 'comparison tool: database memory and classification wall-time measured on
    setA2'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:cebd0a910261ebb7145181e4db5082fd147fd8e881cfddd237c845be49baea3d
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# memory-footprint-profiling

## Summary

Profiles the peak internal/external memory and wall-time of LiME (eBWT-based) against state-of-the-art classifiers (Magic-BLAST, TaxMaps, CLARK-S, Centrifuge, Kraken 2) on the same metagenome sample, to substantiate LiME's lightweight claim.

## When to use

Use when comparing computational cost (peak RAM during database/index construction and classification runtime) of taxonomic classifiers on an identical simulated dataset such as setA2, mirroring Additional file 1 Table S3.

## When NOT to use

- The goal is classification accuracy (sensitivity/precision/F1) rather than resource comparison — use the accuracy-evaluation workflow instead
- Real metagenome data (e.g. SRR1804065) is used where full resource numbers were deferred to Additional file 1 and concordance (rid/ras Jaccard) is the primary metric
- Evaluating thread scaling of LiME — this is covered separately by Additional file 1: Table S4

## Inputs

- Simulated paired-end metagenome dataset setA2 (100 bp reads) from the Lindgreen et al. benchmarking collection
- Reference database Refs.fasta (930 reference genomes) with taxonomic info in Reference_database.csv
- LiME data structures ebwt, lcp, da for the collection S (built via eGSA and merged with eGap --lcp --da)

## Outputs

- table_s3_resource_comparison.csv with one database-construction memory and one classification wall-time entry per tool on setA2
- Per-tool memory/timing log including peak memory during eBWT/eGap construction and LiME internal-memory usage of the similarity matrix

## How to apply

Build the eBWT data structures (ebwt, lcp, da) for the read set R and reference genome set G separately with eGSA (converting the .gesa output with EGSAtoBCR), then merge them with eGap using the --lcp and --da options, recording peak memory during construction (eGap memory is capped via -m, e.g. 4096 MB). Run the three LiME steps on setA2 — ClusterLCP with alpha=16, ClusterBWT_DA with beta=0.25, and Classify with species-level TaxRank in the 100 bp paired-end configuration — measuring wall-time and the internal memory held by the similarity matrix and auxiliary structures. Run Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps on the same setA2 sample, recording each tool's database-construction memory and classification wall-time. Aggregate the per-tool memory and timing measurements into a single table mirroring Additional file 1: Table S3. The rationale is that LiME avoids keeping ad-hoc in-memory reference database structures and only performs sequential scans, so memory and time should favor LiME over tools like CLARK-S or the FM-index-based tools while accuracy stays comparable.

## Related tools

- **LiME** (subject classifier whose memory and wall-time are profiled via ClusterLCP, ClusterBWT_DA and Classify) — https://github.com/veronicaguerrini/LiME
- **eGSA** (builds ebwt, lcp, da for read set and genome set separately during preprocessing whose peak memory is recorded) — https://github.com/felipelouza/egsa
- **eGap** (merges the two ebwts of R and G with --lcp and --da options; its memory is capped (e.g. 4096 MB) and construction memory recorded) — https://github.com/felipelouza/egap
- **MagicBlast** (comparison tool: database memory and classification wall-time measured on setA2)
- **CLARK-S** (comparison tool: database memory and classification wall-time measured on setA2)
- **Centrifuge** (comparison tool: database memory and classification wall-time measured on setA2)
- **Kraken 2** (comparison tool: database memory and classification wall-time measured on setA2)
- **TaxMaps** (comparison tool: database memory and classification wall-time measured on setA2)

## Examples

```
./eGap --lcp -m 4096 ReadsF+Refs.fasta && ./ClusterLCP ReadsF+Refs.fasta numReads 930 16 threads && ./ClusterBWT_DA ReadsF+Refs.fasta 100 0.25 threads
```

## Evaluation signals

- The comparison table contains exactly one memory entry and one wall-time entry per tool on setA2
- Relative ordering is consistent with Additional file 1: Table S3: CLARK-S database memory exceeds that of the FM-index-based tools, and LiME is fastest/lowest-memory among the compared classifiers
- eGap preprocessing respects the configured internal-memory limit (e.g. INTMEM/4096 MB) and peak memory is logged during construction
- LiME classification accuracy on setA2 remains comparable to Magic-BLAST, TaxMaps and CLARK-S, so resource savings do not come at accuracy cost
- Wall-time and memory are measured under the same configuration (100 bp paired-end, species-level TaxRank) across all tools

## Limitations

- Full computational performance results for setA2 are deferred to supplementary Additional file 1: Table S3 and not present in the main text
- Version numbers of the compared tools (Magic-BLAST, CLARK-S, Centrifuge, Kraken 2, TaxMaps) are not stated in the article
- Finding the best method for building the eBWT data structures is out of scope of the paper; classification does not depend on the construction algorithm used
- Thread scaling results are deferred to Additional file 1: Table S4 and not covered by this skill

## Evidence

- [discussion] Full results for the dataset setA2 are available in Additional file 1: Table S3.: "Full results for the dataset setA2 are available in Additional file 1: Table S3."
- [intro] LiME does not need ad-hoc in-memory database: "LiME does not need to build ad-hoc and keep in internal memory the data structures relating to the database of the reference genomes, as it only needs to run sequential scans"
- [readme] eGap merge options: "To merge the two ebwts associated with the sets *R* and *G*, one could use eGap [https://github.com/felipelouza/egap] and set both options *--lcp* and *--da*"
- [readme] eGap memory setting: "one can specify the maximum available internal memory to be used by egsa and eGap (in MB); in the above two scripts we set 4096 MB"
- [discussion] Compared tools list: "Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps, which have recently been introduced."
- [results] Resource comparison finding: [evidence span withheld — anchor status: not_found]
