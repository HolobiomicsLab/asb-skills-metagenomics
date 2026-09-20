---
name: lowest-common-ancestor-taxonomy-resolution
description: Decision support for assigning taxonomy to filtered viral contigs (>10
  kbp) from ViroProfiler when combining clustering-based (vConTACT2) and homology-based
  (MMseqs2) evidence.
when_to_use_negative:
- The task is classifying viruses in samples without prior viral isolation — ViroProfiler
  is specifically designed for classifying viruses in samples with isolated viruses
- Input contigs are shorter than 10 kbp, which are filtered out before taxonomy assignment
- The goal is viral identification/discovery (VirSorter2, VIBRANT, DeepVirFinder)
  rather than taxonomic placement of already-identified viral contigs
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0781
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline that orchestrates the taxonomy assignment
    workflow and merges cluster- and homology-based calls
  repo: https://github.com/deng-lab/viroprofiler
- name: vConTACT2
  role: Clusters contigs into roughly genus-level viral clusters based on shared protein
    content (TAXONOMY_VCONTACT process, 4 cpus, 100 GB memory)
- name: MMseqs2
  role: Taxonomy module searched against NCBI viral RefSeq to obtain homology-based
    taxonomy calls per contig (TAXONOMY_MMSEQS process, 4 cpus, 120 GB memory)
  repo: https://github.com/soedinglab/MMseqs2
- name: TaxonKit
  role: Normalises resulting lineage names to a standardised format before writing
    viral_taxonomy.tsv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:4a2014e63bf887b9b7f7e6883c014deca1acb41629fe04ce7b88aeaa36c28dfa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/deng-lab/viroprofiler
  tools:
  - MMseqs2
  - TaxonKit
  - ViroProfiler
  - vConTACT2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# lowest-common-ancestor-taxonomy-resolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns taxonomy to viral contigs from shotgun viral metagenomic data by reconciling vConTACT2 clustering-based genus-level viral clusters with MMseqs2 homology-based calls against NCBI viral RefSeq, using lowest-common-ancestor voting to resolve conflicts, then normalising lineages with TaxonKit.

## When to use

Decision support for assigning taxonomy to filtered viral contigs (>10 kbp) from ViroProfiler when combining clustering-based (vConTACT2) and homology-based (MMseqs2) evidence.

## When NOT to use

- The task is classifying viruses in samples without prior viral isolation — ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses
- Input contigs are shorter than 10 kbp, which are filtered out before taxonomy assignment
- The goal is viral identification/discovery (VirSorter2, VIBRANT, DeepVirFinder) rather than taxonomic placement of already-identified viral contigs

## Inputs

- Filtered viral contigs or bins from metagenomic data (retaining sequences longer than 10 kbp)
- NCBI viral RefSeq database (for MMseqs2 taxonomy search)

## Outputs

- viral_taxonomy.tsv: contig-to-taxonomy table with standardised lineage names

## How to apply

Filter viral contigs to sequences longer than 10 kbp, since only those are retained for taxonomy assignment. Run vConTACT2 (process TAXONOMY_VCONTACT, 4 cpus, 100 GB memory) to place contigs into roughly genus-level viral clusters based on shared protein content, and in parallel run the MMseqs2 taxonomy module (process TAXONOMY_MMSEQS, 4 cpus, 120 GB memory) against the NCBI viral RefSeq database for per-contig homology calls. Merge both assignment types and resolve conflicting taxonomic calls within each viral cluster by lowest-common-ancestor reconciliation, so that a contig inherits the most specific rank supported by both clustering and homology evidence. Finally, normalise lineage names to a standardised format with TaxonKit and emit the contig-to-taxonomy table as viral_taxonomy.tsv.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline that orchestrates the taxonomy assignment workflow and merges cluster- and homology-based calls) — https://github.com/deng-lab/viroprofiler
- **vConTACT2** (Clusters contigs into roughly genus-level viral clusters based on shared protein content (TAXONOMY_VCONTACT process, 4 cpus, 100 GB memory))
- **MMseqs2** (Taxonomy module searched against NCBI viral RefSeq to obtain homology-based taxonomy calls per contig (TAXONOMY_MMSEQS process, 4 cpus, 120 GB memory)) — https://github.com/soedinglab/MMseqs2
- **TaxonKit** (Normalises resulting lineage names to a standardised format before writing viral_taxonomy.tsv)

## Evaluation signals

- viral_taxonomy.tsv exists and is a well-formed table
- Every input contig longer than 10 kbp receives a taxonomy entry at a standardised lineage
- Lineage names in viral_taxonomy.tsv are in TaxonKit-normalised, standardised format
- Each contig's assigned rank is consistent with lowest-common-ancestor reconciliation of its vConTACT2 viral cluster and its MMseqs2 RefSeq hit
- Both TAXONOMY_VCONTACT and TAXONOMY_MMSEQS processes completed with their documented resource allocations (4 cpus; 100 GB and 120 GB memory)

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses
- Excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision
- Only contigs longer than 10 kbp are retained for taxonomy assignment, so short viral contigs receive no taxonomy entry

## Evidence

- [readme] MMseqs2 is the homology-search engine used for per-contig taxonomy calls: [evidence span withheld — anchor status: external_doc]
