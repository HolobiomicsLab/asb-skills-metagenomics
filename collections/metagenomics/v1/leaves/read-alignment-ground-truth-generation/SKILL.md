---
name: read-alignment-ground-truth-generation
description: Use when a simulated metagenomic dataset with known reference genomes
  (e.g. Sim50, Sim100, CAMI2 UG) has been assembled into contigs and you need per-contig
  truth labels to evaluate binning tools with AMBER.
when_to_use_negative:
- The dataset is a real-world metagenome with no reference genomes available, so contigs
  cannot be assigned a genome of origin (e.g. a fully de novo environmental sample
  without simulated truth).
- You need to evaluate the binner on all contigs regardless of mapping confidence
  and cannot afford discarding multi-mapping contigs from the benchmark.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Minimap2
  role: Aligns assembled contigs to the reference genomes to determine each contig's
    genome of origin
  repo: https://github.com/wwood/CoverM
- name: SAMTools
  role: Processes Minimap2 alignments and supports retaining only contigs with the
    highest mapping quality
- name: AMBER
  role: Consumes the generated ground truth to compute purity, completeness, F1, accuracy,
    and percentage binned for each binner's output
- name: CoverM
  role: Produces the sample-wise contig coverage TSV used alongside the ground truth
    in binning workflows on the same contigs
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.compbiolchem.2022.107734
    title: 'CH-Bin: A convex hull based approach for binning metagenomic contigs'
  source_content_hash: sha256:6e074278ca791668e2d877ec9c50cabfd73e3711ace36dbfd0407bfb9546f536
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kdsuneraavinash/CH-Bin
  repo_url: https://github.com/wwood/CoverM
  tools:
  - AMBER
  - Busy Bee Web
  - CH-Bin
  - CoverM
  - FragGeneScan
  - HMMER3
  - MaxBin2
  - MetaBAT2
  - Minimap2
  - SAMTools
  - SolidBin
  - metaSPAdes
  - seq2vec
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# read-alignment-ground-truth-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a binning ground-truth label set for AMBER evaluation by aligning assembled contigs to known reference genomes with Minimap2 and SAMTools, retaining only contigs whose best mapping has the highest mapping quality. This converts reference genome assignments into the per-contig bin labels AMBER needs to score purity, completeness, and F1.

## When to use

Use when a simulated metagenomic dataset with known reference genomes (e.g. Sim50, Sim100, CAMI2 UG) has been assembled into contigs and you need per-contig truth labels to evaluate binning tools with AMBER.

## When NOT to use

- The dataset is a real-world metagenome with no reference genomes available, so contigs cannot be assigned a genome of origin (e.g. a fully de novo environmental sample without simulated truth).
- You need to evaluate the binner on all contigs regardless of mapping confidence and cannot afford discarding multi-mapping contigs from the benchmark.

## Inputs

- Assembled contigs in multi-FASTA format (e.g. metaSPAdes assembly of Sim50, Sim100, CAMI2 UG, or Sharon datasets)
- Reference genome FASTA files for the species used in simulation (e.g. simMC+ / simHC+ species, CAMI UG genomes)

## Outputs

- AMBER ground-truth file mapping each filtered contig to its reference genome of origin
- Per-dataset AMBER evaluation metrics table (average purity bp, average completeness bp, F1 bp, accuracy bp, percentage binned bp, number of bins), aggregated e.g. into amber_evaluation_results.csv

## How to apply

For each simulated dataset (Sim5, Sim10, Sim20, Sim50, Sim100, CAMI2 UG), align the metaSPAdes-assembled multi-FASTA contigs against the reference genomes used in simulation using Minimap2, then process the alignments with SAMTools. Keep only contigs whose highest-scoring alignment has the highest mapping quality, so each contig receives exactly one unambiguous genome-of-origin label; discard ambiguously mapped contigs rather than assigning them. Write the resulting contig-to-genome mapping as the AMBER ground-truth file, then run AMBER against candidate binner outputs (CH-Bin, MaxBin2, MetaBAT2 -m 1500, SolidBin 'naive' mode, BusyBee Web) to extract average purity (bp), average completeness (bp), F1 score (bp), accuracy (bp), percentage binned (bp), and number of bins. The rationale is that mapping quality filtering prevents multi-mapping contigs from introducing label noise into the binning benchmark.

## Related tools

- **Minimap2** (Aligns assembled contigs to the reference genomes to determine each contig's genome of origin) — https://github.com/wwood/CoverM
- **SAMTools** (Processes Minimap2 alignments and supports retaining only contigs with the highest mapping quality)
- **AMBER** (Consumes the generated ground truth to compute purity, completeness, F1, accuracy, and percentage binned for each binner's output)
- **CoverM** (Produces the sample-wise contig coverage TSV used alongside the ground truth in binning workflows on the same contigs) — https://github.com/wwood/CoverM

## Examples

```
minimap2 -ax sr reference_genomes.fna contigs.fna | samtools view -b -q 60 > filtered.bam  # keep highest-MQ contig alignments as AMBER ground truth
```

## Evaluation signals

- Every retained contig appears in the ground truth exactly once with a single reference genome label (no multi-assignment after mapping-quality filtering).
- Reproducing the Sim50 benchmark should yield CH-Bin (k=5,6) at F1 = 96.91%, average purity 95.28%, average completeness 98.60%, accuracy 93.10%, and 46 bins.
- Aggregated metrics across datasets and tools assemble into a single amber_evaluation_results.csv with one row per (dataset, tool) pair.
- CH-Bin (k=5,6) attains the highest F1 score among all evaluated tools on Sim50, Sim100, and CAMI2 UG, matching the reported comparison.
- Contigs filtered out should be those without a unique highest-mapping-quality hit; spot-checking several multi-mapping contigs confirms they are excluded.

## Limitations

- The ground truth is only as good as the reference set; it is applicable to simulated datasets or datasets with known reference genomes, not purely novel communities.
- Discarding contigs without the highest mapping quality reduces the evaluated contig pool, so benchmark metrics (e.g. percentage binned bp) reflect only uniquely mappable contigs.
- The CH-Bin repository provides no examples and no Zenodo/archival DOI beyond the GitHub source link, so reproducibility of the exact ground-truth commands depends on the paper's workflow description.

## Evidence

- [results] Build the AMBER ground truth for each dataset by aligning contigs to the respective reference genomes with Minimap2 and SAMTools, retaining only contigs with the highest mapping quality.: [evidence span withheld — anchor status: not_found]
- [results] Evaluate every tool's binning with AMBER, extracting average purity (bp), average completeness (bp), F1 score (bp), accuracy (bp), percentage binned (bp) and number of bins identified.: [evidence span withheld — anchor status: not_found]
