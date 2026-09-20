---
name: homology-search-against-viral-refseq
description: Use when viral contigs (>10 kbp, already identified as viral) need reference-backed
  taxonomy, and you want homology-based calls against NCBI viral RefSeq to complement
  clustering-based assignment.
when_to_use_negative:
- Input contigs are shorter than 10 kbp — ViroProfiler retains only sequences longer
  than 10 kbp for taxonomy assignment
- The goal is classifying viruses in samples with isolated viruses, which is outside
  ViroProfiler's stated design scope
- Sequences are not yet confirmed viral — this skill assumes viral identification
  (e.g., VirSorter2/VIBRANT/DeepVirFinder) has already been performed
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: MMseqs2
  role: taxonomy module searched against the NCBI viral RefSeq database to obtain
    homology-based taxonomy calls per contig
  repo: https://github.com/soedinglab/MMseqs2
- name: ViroProfiler
  role: containerized Nextflow pipeline that orchestrates the MMseqs2 homology search
    and merges it with vConTACT2 cluster-based taxonomy
  repo: https://github.com/deng-lab/viroprofiler
- name: TaxonKit
  role: normalises the resulting lineage names to a standardised format before writing
    the contig-to-taxonomy table
- name: vConTACT2
  role: provides the complementary cluster-based (roughly genus-level viral cluster)
    assignments reconciled with the homology calls via LCA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:2a8ab2f7df9951dd641417c4583f30ddeadab029c394587aa19d462dc3e39afa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/soedinglab/MMseqs2
  tools:
  - MMseqs2
  - TaxonKit
  - ViroProfiler
  - vConTACT2
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# homology-search-against-viral-refseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign taxonomy to filtered viral contigs by running the MMseqs2 taxonomy module against the NCBI viral RefSeq database, producing homology-based taxonomic calls that are later reconciled with vConTACT2 cluster-based assignments via lowest-common-ancestor voting within ViroProfiler.

## When to use

Use when viral contigs (>10 kbp, already identified as viral) need reference-backed taxonomy, and you want homology-based calls against NCBI viral RefSeq to complement clustering-based assignment.

## When NOT to use

- Input contigs are shorter than 10 kbp — ViroProfiler retains only sequences longer than 10 kbp for taxonomy assignment
- The goal is classifying viruses in samples with isolated viruses, which is outside ViroProfiler's stated design scope
- Sequences are not yet confirmed viral — this skill assumes viral identification (e.g., VirSorter2/VIBRANT/DeepVirFinder) has already been performed

## Inputs

- Viral contigs longer than 10 kbp (pre-filtered viral sequences from metagenomic assembly)
- NCBI viral RefSeq database (MMseqs2-formatted reference)

## Outputs

- Per-contig homology-based taxonomy calls (MMseqs2 taxonomy output)
- Merged cluster-based + homology-based contig-to-taxonomy table (viral_taxonomy.tsv) with TaxonKit-normalised lineage names

## How to apply

Within the ViroProfiler workflow, first retain only viral contigs longer than 10 kbp for taxonomy assignment. Run the MMseqs2 taxonomy module (process TAXONOMY_MMSEQS, 4 cpus, 120 GB memory) with the NCBI viral RefSeq database as the reference to obtain a homology-based taxonomy call per contig. In parallel, run vConTACT2 (process TAXONOMY_VCONTACT, 4 cpus, 100 GB memory) to place contigs into roughly genus-level viral clusters based on shared protein content. Merge the two assignment types, resolving conflicting taxonomic calls within each viral cluster by lowest-common-ancestor reconciliation. Finally, normalise lineage names to a standardised format with TaxonKit and write the contig-to-taxonomy table to viral_taxonomy.tsv. The rationale is that homology against curated RefSeq references grounds calls in known viruses, while the clustering-based vConTACT2 assignment captures relationships among uncultivated viruses absent from reference databases.

## Related tools

- **MMseqs2** (taxonomy module searched against the NCBI viral RefSeq database to obtain homology-based taxonomy calls per contig) — https://github.com/soedinglab/MMseqs2
- **ViroProfiler** (containerized Nextflow pipeline that orchestrates the MMseqs2 homology search and merges it with vConTACT2 cluster-based taxonomy) — https://github.com/deng-lab/viroprofiler
- **TaxonKit** (normalises the resulting lineage names to a standardised format before writing the contig-to-taxonomy table)
- **vConTACT2** (provides the complementary cluster-based (roughly genus-level viral cluster) assignments reconciled with the homology calls via LCA)

## Evaluation signals

- viral_taxonomy.tsv exists, is a well-formed table, and every input contig >10 kbp receives a taxonomy entry at a standardised lineage
- TAXONOMY_MMSEQS process completed with the expected resource profile (4 cpus, 120 GB memory) against the NCBI viral RefSeq database
- Conflicting taxonomic calls within each vConTACT2 viral cluster are resolved to a single lowest-common-ancestor lineage
- All lineage names in the output are in the TaxonKit-normalised standardised format

## Limitations

- Contigs <=10 kbp are excluded from taxonomy assignment by the workflow's length filter
- Excessive environmental contaminations in metagenome sequences could increase the pipeline running time and result in lower precision
- Novel viruses with no close relatives in NCBI viral RefSeq may receive only shallow or unresolved homology-based assignments; the vConTACT2 clustering branch is intended to complement such cases

## Evidence

- [readme] MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets.: [evidence span withheld — anchor status: external_doc]
