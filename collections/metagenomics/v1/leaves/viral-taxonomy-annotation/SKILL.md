---
name: viral-taxonomy-annotation
description: Use when viral contigs predicted from shotgun viral metagenomes (VirSorter2/VIBRANT/DeepVirFinder
  output) need genus- or family-level taxonomic labels in a ViroProfiler run.
when_to_use_negative:
- Samples contain isolated viruses rather than metagenomic data — ViroProfiler is
  specifically designed for classifying viruses in samples with isolated viruses is
  not its target use
- Sequences are heavily contaminated with non-viral material — excessive environmental
  contaminations in metagenome sequences could increase running time and result in
  lower precision
- Input contigs are shorter than 10 kbp — they are filtered out before taxonomy assignment
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline that orchestrates the taxonomy assignment
    workflow (TAXONOMY_VCONTACT, TAXONOMY_MMSEQS processes) and writes viral_taxonomy.tsv
  repo: https://github.com/deng-lab/viroprofiler
- name: vConTACT2
  role: Clusters viral contigs into roughly genus-level viral clusters based on shared
    protein content
- name: MMseqs2
  role: Taxonomy module; homology-based per-contig taxonomy calls against the NCBI
    viral RefSeq database
  repo: https://github.com/soedinglab/MMseqs2
- name: TaxonKit
  role: Normalises resulting lineage names to a standardised format
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:b6180ad614be4bac10f0a1b3a9f2c48e18c31910b07aaefe703e2a9559ace463
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
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# viral-taxonomy-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign taxonomy to viral contigs/bins in ViroProfiler by combining vConTACT2 clustering (genus-level viral clusters) with MMseqs2 homology-based calls against NCBI viral RefSeq, reconciled by LCA voting and normalised with TaxonKit.

## When to use

Use when viral contigs predicted from shotgun viral metagenomes (VirSorter2/VIBRANT/DeepVirFinder output) need genus- or family-level taxonomic labels in a ViroProfiler run.

## When NOT to use

- Samples contain isolated viruses rather than metagenomic data — ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses is not its target use
- Sequences are heavily contaminated with non-viral material — excessive environmental contaminations in metagenome sequences could increase running time and result in lower precision
- Input contigs are shorter than 10 kbp — they are filtered out before taxonomy assignment

## Inputs

- Viral contigs (predicted by VirSorter2/VIBRANT/DeepVirFinder/VIP), retaining only sequences >10 kbp for taxonomy assignment
- NCBI viral RefSeq database (reference for MMseqs2 taxonomy)

## Outputs

- viral_taxonomy.tsv — contig-to-taxonomy table with standardised lineage names
- vConTACT2 viral cluster assignments (genus-level clusters)

## How to apply

In ViroProfiler, filter viral contigs to sequences longer than 10 kbp before taxonomy assignment. Run vConTACT2 (process TAXONOMY_VCONTACT, 4 cpus, 100 GB memory) to place contigs into roughly genus-level viral clusters based on shared protein content, then run the MMseqs2 taxonomy module (process TAXONOMY_MMSEQS, 4 cpus, 120 GB memory) against the NCBI viral RefSeq database for per-contig homology calls. Merge the cluster-based and homology-based assignments, resolving conflicting taxonomic calls within each viral cluster by lowest-common-ancestor (LCA) reconciliation. Finally, normalise lineage names to a standardised format using TaxonKit and write the contig-to-taxonomy table to viral_taxonomy.tsv.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline that orchestrates the taxonomy assignment workflow (TAXONOMY_VCONTACT, TAXONOMY_MMSEQS processes) and writes viral_taxonomy.tsv) — https://github.com/deng-lab/viroprofiler
- **vConTACT2** (Clusters viral contigs into roughly genus-level viral clusters based on shared protein content)
- **MMseqs2** (Taxonomy module; homology-based per-contig taxonomy calls against the NCBI viral RefSeq database) — https://github.com/soedinglab/MMseqs2
- **TaxonKit** (Normalises resulting lineage names to a standardised format)

## Evaluation signals

- viral_taxonomy.tsv exists and is a well-formed table
- Every input contig >10 kbp receives a taxonomy entry at a standardised lineage
- Lineage names in the output conform to the TaxonKit-normalised standardised format
- Conflicting taxonomic calls within each vConTACT2 viral cluster are resolved to a single LCA call
- Both TAXONOMY_VCONTACT (vConTACT2) and TAXONOMY_MMSEQS processes completed successfully for the input contigs

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses
- Excessive environmental contaminations in metagenome sequences could increase the running time of the pipeline and result in lower precision
- Isolating the viral fractions before sequencing is recommended for an accurate estimation of viruses in the samples

## Evidence

- [readme] MMseqs2 description: [evidence span withheld — anchor status: external_doc]
