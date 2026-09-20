---
name: cd-hit-nucleotide-sequence-clustering
description: Use when you have a large set of extracted marker-gene or coding nucleotide
  sequences (FASTA) from many genomes and need a non-redundant reference catalog that
  keeps strain-level resolution (high identity cutoff like 99%).
when_to_use_negative:
- Building a functional protein database where broader redundancy collapse is desired
  — the paper used 95% identity clustering of proteins (FunOMIC-P) rather than 99%
  nucleotide clustering
- Input reads are short NGS reads to be mapped against an existing reference — clustering
  is for reference construction, not read mapping (use bowtie2 with >80% alignment
  length and q-score >30 filters for that)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: CD-HIT
  role: Clusters extracted nucleotide marker-gene sequences at a 99% identity threshold
    to remove redundancy and produce the final FunOMIC-T catalog
- name: BUSCO 5.0.0
  role: 'Upstream step: identifies single-copy marker genes in each genome using Fungi
    OrthoDB v10.1 HMMs before sequences are pooled for clustering'
- name: AUGUSTUS
  role: Gene predictor used by BUSCO in default mode to call marker-gene sequences
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:588359a6d1ebcec248f32a9bbf4c98e438fb7205b2a7995d73160ff89e03d850
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - AUGUSTUS
  - BUSCO
  - BUSCO 5.0.0
  - CD-HIT
  - FunOMIC
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cd-hit-nucleotide-sequence-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cluster nucleotide sequences with CD-HIT at a 99% identity threshold to remove redundancy while preserving near-identical strain-level variants, as used to build the 1.69-million-gene FunOMIC-T fungal marker-gene catalog from BUSCO-extracted single-copy marker genes.

## When to use

Use when you have a large set of extracted marker-gene or coding nucleotide sequences (FASTA) from many genomes and need a non-redundant reference catalog that keeps strain-level resolution (high identity cutoff like 99%).

## When NOT to use

- Building a functional protein database where broader redundancy collapse is desired — the paper used 95% identity clustering of proteins (FunOMIC-P) rather than 99% nucleotide clustering
- Input reads are short NGS reads to be mapped against an existing reference — clustering is for reference construction, not read mapping (use bowtie2 with >80% alignment length and q-score >30 filters for that)

## Inputs

- Nucleotide FASTA of single-copy marker gene sequences extracted from quality-filtered fungal genomes (e.g. from 4,816 genomes passing the <30-marker-gene filter)
- Chosen identity threshold (99% for the taxonomic marker catalog; 95% for protein redundancy removal)

## Outputs

- Non-redundant marker-gene catalog FASTA (~1.69 million fungal single-copy marker genes, funomic_t_marker_genes.fasta)
- CD-HIT cluster membership file mapping redundant sequences to cluster representatives

## How to apply

After identifying single-copy marker genes (e.g. BUSCO 5.0.0 with Fungi OrthoDB v10.1 and the AUGUSTUS gene predictor) and extracting their nucleotide sequences from curated genomes, run CD-HIT on the pooled nucleotide FASTA with a 99% identity threshold to collapse near-identical duplicates while retaining strain-level variants. In the FunOMIC-T construction this step reduced marker genes extracted from 4,816 filtered genomes (of 4,839 collected from NCBI and JGI MycoCosm) to a final set of ~1.69 million genes. Note that CD-HIT is applied separately from the protein-level database construction, which used a looser 95% identity threshold; choose the threshold according to whether the catalog must discriminate closely related taxa (99%) or represent broader functional protein space (95%). The output FASTA (e.g. funomic_t_marker_genes.fasta) then serves as the mapping target for taxonomic profiling.

## Related tools

- **CD-HIT** (Clusters extracted nucleotide marker-gene sequences at a 99% identity threshold to remove redundancy and produce the final FunOMIC-T catalog)
- **BUSCO 5.0.0** (Upstream step: identifies single-copy marker genes in each genome using Fungi OrthoDB v10.1 HMMs before sequences are pooled for clustering)
- **AUGUSTUS** (Gene predictor used by BUSCO in default mode to call marker-gene sequences)

## Examples

```
cd-hit -i marker_genes_raw.fasta -o funomic_t_marker_genes.fasta -c 0.99 -aS 0.9
```

## Evaluation signals

- Final catalog size falls in the expected range (~1.6–1.69 million non-redundant marker genes from 4,816 genomes)
- Fewer than 0.01% of clustered marker genes map to the prokaryotic UHGG catalogue (bowtie2), confirming fungal specificity
- Simulated bacterial reads (903 genomes, 458 species; bacterial:fungal read ratio ~1000) show ~0 alignment to the clustered catalog (specificity 0.9999)
- Clustered catalog spans the expected taxonomic breadth: 475 genera, 1,916 species, 4,537 strains across eight fungal phyla
- All sequences in the output FASTA derive from genomes that yielded ≥30 single-copy marker genes

## Limitations

- Extraction and quality of single-copy marker genes depend on completeness of available fungal genomes, which may result in lower fungal taxonomy coverage compared with amplicon databases
- The 99% threshold preserves strain-level variants, so the catalog remains large (~1.69M genes) and mapping against it is computationally heavier than a collapsed protein database
- Genome quality filtering (discard genomes with >500 contigs or N50 <10 kbp, and <30 marker genes) must precede clustering or low-quality sequence will contaminate the catalog

## Evidence
