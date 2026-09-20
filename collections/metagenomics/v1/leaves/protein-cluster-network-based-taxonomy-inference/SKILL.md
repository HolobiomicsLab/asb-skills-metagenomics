---
name: protein-cluster-network-based-taxonomy-inference
description: Use when viral contigs (>10 kbp) identified from shotgun viral metagenomes
  need taxonomy, and both clustering-based (viral cluster) and homology-based (reference
  database) evidence should be combined into one lineage per contig.
when_to_use_negative:
- Input contigs are shorter than 10 kbp, where shared-protein clustering is unreliable
  for cluster-level placement
- The goal is to classify viruses in samples with isolated viruses rather than metagenomic
  viral contigs — ViroProfiler is specifically designed for classifying viruses in
  metagenomic samples
- Metagenome sequences carry excessive environmental contamination, which increases
  running time and lowers precision of taxonomy calls
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0781
tools:
- name: ViroProfiler
  role: Containerized pipeline that orchestrates the taxonomy assignment subworkflow
    and writes viral_taxonomy.tsv
  repo: https://github.com/deng-lab/viroprofiler
- name: vConTACT2
  role: Clusters viral contigs into roughly genus-level viral clusters based on shared
    protein content
- name: MMseqs2
  role: Taxonomy module searched against NCBI viral RefSeq to produce homology-based
    per-contig taxonomy calls
  repo: https://github.com/soedinglab/MMseqs2
- name: TaxonKit
  role: Normalises resulting lineage names to a standardised format in the contig-to-taxonomy
    table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:d33235ffbc599e95aa13dec06b334f2cf46197a00ce30195ce47404157f709a5
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# protein-cluster-network-based-taxonomy-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign taxonomy to viral contigs by combining vConTACT2 protein-content clustering (viral-cluster-level placement) with MMseqs2 homology searches against NCBI viral RefSeq, reconciling conflicts by lowest-common-ancestor voting and normalising lineages with TaxonKit.

## When to use

Use when viral contigs (>10 kbp) identified from shotgun viral metagenomes need taxonomy, and both clustering-based (viral cluster) and homology-based (reference database) evidence should be combined into one lineage per contig.

## When NOT to use

- Input contigs are shorter than 10 kbp, where shared-protein clustering is unreliable for cluster-level placement
- The goal is to classify viruses in samples with isolated viruses rather than metagenomic viral contigs — ViroProfiler is specifically designed for classifying viruses in metagenomic samples
- Metagenome sequences carry excessive environmental contamination, which increases running time and lowers precision of taxonomy calls

## Inputs

- Filtered viral contig FASTA (retaining only sequences longer than 10 kbp)
- NCBI viral RefSeq database (for MMseqs2 taxonomy searches)
- Protein-cluster network reference used by vConTACT2

## Outputs

- viral_taxonomy.tsv: contig-to-taxonomy table with standardised lineage names
- Per-contig homology-based taxonomy calls from MMseqs2
- vConTACT2 viral cluster assignments for each contig

## How to apply

First filter the viral contig set to sequences longer than 10 kbp, since shorter fragments lack reliable shared protein content for cluster placement. Run vConTACT2 (process TAXONOMY_VCONTACT, 4 cpus, 100 GB memory) to place contigs into roughly genus-level viral clusters based on shared protein content. In parallel, run the MMseqs2 taxonomy module (process TAXONOMY_MMSEQS, 4 cpus, 120 GB memory) against the NCBI viral RefSeq database to obtain a homology-based call per contig. Merge the two evidence streams, resolving conflicting taxonomic calls within each viral cluster by lowest-common-ancestor (LCA) reconciliation rather than picking a single best hit. Finally, normalise lineage names with TaxonKit and write the contig-to-taxonomy table to viral_taxonomy.tsv.

## Related tools

- **ViroProfiler** (Containerized pipeline that orchestrates the taxonomy assignment subworkflow and writes viral_taxonomy.tsv) — https://github.com/deng-lab/viroprofiler
- **vConTACT2** (Clusters viral contigs into roughly genus-level viral clusters based on shared protein content)
- **MMseqs2** (Taxonomy module searched against NCBI viral RefSeq to produce homology-based per-contig taxonomy calls) — https://github.com/soedinglab/MMseqs2
- **TaxonKit** (Normalises resulting lineage names to a standardised format in the contig-to-taxonomy table)

## Examples

```
nextflow run deng-lab/viroprofiler -profile docker
```

## Evaluation signals

- viral_taxonomy.tsv exists and is a well-formed table
- Every input contig >10 kbp receives a taxonomy entry at a standardised lineage
- Lineage names in viral_taxonomy.tsv are in the TaxonKit-normalised standardised format
- Contigs placed in the same vConTACT2 viral cluster do not receive discordant high-rank taxonomy calls unless reconciled to the LCA
- Both TAXONOMY_VCONTACT (4 cpus, 100 GB) and TAXONOMY_MMSEQS (4 cpus, 120 GB) processes complete successfully in the workflow log

## Limitations

- Taxonomy assignment is restricted to viral contigs longer than 10 kbp; shorter contigs receive no cluster-supported placement
- Excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision
- The two evidence streams (cluster-based and homology-based) can conflict, and LCA reconciliation may yield more conservative, higher-rank lineages

## Evidence

- [readme] MMseqs2 searches and clusters large sequence sets, supporting the homology-based taxonomy module of the workflow.: [evidence span withheld — anchor status: external_doc]
