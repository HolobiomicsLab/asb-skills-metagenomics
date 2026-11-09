---
name: reference-position-mapping
description: Use when variant calls or gene features exist in colored de Bruijn graph
  node coordinates (nodepos plus bubid/basecolor tri-tuples) and must be converted
  to reference-genome positions for comparison with linear annotations like GTF files
  or VCFs.
when_to_use_negative:
- Gene elements shorter than k (here k = 22), which may map to multiple graph nodes
  making the mapping ambiguous
- Detection of large indels, inversions, or reverse tandem duplications — the variant
  model only addresses small indels (< 50 bp)
- Sample sets with insufficient sequence similarity, where the variant detection model
  that RPD depends on is not applicable
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: cSupB
  role: Implements the cSupB decomposition, tri-tuple coordinate assignment, NVD,
    and RPD steps that produce reference-position-mapped outputs
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph (k = 22, acyclic, unique start/end
    nodes) whose nodes RPD maps to reference positions
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
- name: C++
  role: Language of the cSupB program implementing the coordinate mapping pipeline
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:9ca59fe70963d104076cb7532da3e0a88e71ba4fc9eef0288f3f110dcdd06228
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - Clustal Omega
  - MAFFT-GINSI
  - MUSCLE
  - Mugsy
  - VARI-merge
  - cSupB
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# reference-position-mapping

## Summary

Reference Position Determination (RPD) transforms variants and gene elements expressed in the cSupB tri-tuple graph coordinate system (position, bubid, basecolor) into conventional reference-genome (locus) coordinates, so graph-derived calls can be compared with linear reference annotations such as the 1000 Genomes mtDNA VCF or a GTF.

## When to use

Use when variant calls or gene features exist in colored de Bruijn graph node coordinates (nodepos plus bubid/basecolor tri-tuples) and must be converted to reference-genome positions for comparison with linear annotations like GTF files or VCFs.

## When NOT to use

- Gene elements shorter than k (here k = 22), which may map to multiple graph nodes making the mapping ambiguous
- Detection of large indels, inversions, or reverse tandem duplications — the variant model only addresses small indels (< 50 bp)
- Sample sets with insufficient sequence similarity, where the variant detection model that RPD depends on is not applicable

## Inputs

- cSupB decomposition files of the colored de Bruijn graph (CST topology, CSDI details, NNI node-nearby info, OVI offset-value info) built by VARI-merge with k = 22 from the 12 human mitogenomes
- Detected variants as node coordinates (position, bubid, basecolor) from the Nested Variant Detection (NVD) algorithm
- A reference genome among the samples (e.g., NC_012920) defining refpos
- Linear reference annotation to project, e.g., a mitochondrial GTF gene file or 1000 Genomes chrMT phase3 VCF

## Outputs

- Variant calls with reference-locus coordinates in VCF format (variant_calls.vcf), each carrying the tri-tuple (position, bubid, basecolor)
- Graph-projected gene annotation in GTF format (gtf2graph.txt / mitochondrial_gene_regions.gtf) with start/end reference coordinates
- Mapping table relating nodepos to refpos via the nearest parent source node containing the reference genome

## How to apply

First ensure the cSupB decomposition files (CST, CSDI, NNI, OVI) are loaded and the cSupB hierarchical tree is ordered from the root bubble, since a child cSupB cannot be resolved before its parent. For each node and base, record the tri-tuple (position, bubid, basecolor), where bubid is the smallest containing cSupB and basecolor is the 0-1 sample string. Then apply RPD: map each graph node position (nodepos) onto a reference-genome position (refpos) by locating the nearest parent source node that contains the reference genome (e.g., NC_012920 among the 12 human mitogenomes), using offset values computed from the source/sink node positions. Transform node variants into locus variants: the NVD algorithm classifies each source node variant by gap values (gap1 > 0 = INS, gap2 > 0 = DEL, gap1 = gap2 = 0 = SNP), and RPD assigns the reference locus. Project linear annotations by the refpos relation, e.g., emitting mitochondrial gene elements as GTF records with start/end reference coordinates, and write variant calls in VCF format. Judge success by whether each gene resolves to a unique (position, bubid, basecolor) triple and whether refpos-mapped calls match the deposited reference annotation.

## Related tools

- **cSupB** (Implements the cSupB decomposition, tri-tuple coordinate assignment, NVD, and RPD steps that produce reference-position-mapped outputs) — https://github.com/eggleader/cSupB
- **VARI-merge** (Constructs the succinct colored de Bruijn graph (k = 22, acyclic, unique start/end nodes) whose nodes RPD maps to reference positions) — https://github.com/cosmo-team/cosmo/tree/VARI-merge
- **C++** (Language of the cSupB program implementing the coordinate mapping pipeline)

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- Every emitted variant record carries a complete tri-tuple coordinate (position, bubid, basecolor)
- No called indel exceeds 50 bp (small-indel restriction)
- The projected GTF matches the deposited reference GTF (Additional file 4 of 10.1186/s12859-021-04149-w) in gene feature counts and coordinate intervals
- Variant calls are consistent with the reference variant set in the article's Additional file 3 (e.g., 89 of 110 SNVs location-mapped to the 1000 Genomes mtDNA annotation)
- Every gene region resolves to a unique (position, bubid, basecolor) triple

## Limitations

- If the length of a gene element is less than k, it may correspond to multiple nodes on the graph, making analysis complicated and ambiguous
- For highly repetitive sequences, especially tandem repeats, the sequence cutting method fragments sequences excessively, increasing computational difficulty and error
- Variant detection precision decreases when sample similarity decreases and variant types become complicated; the approach is applicable only to high-similarity samples with simple variant types
- Degeneracy cannot be handled efficiently by the underlying graph model

## Evidence

- [methods] RPD definition: "Determine the relations between the nodes' position on the reference genome and graph (reference position determination (RPD)); then, transform the node variant to a locus variant"
- [methods] hierarchical ordering precondition: "a child cSupB cannot be obtained before its parent cSupB (Additional file 1: Corollary 6)"
- [readme] tri-tuple coordinate definition: "Based on this model, a tri-tuple coordinate system that combines an offset value, topological structure and sample information."
- [abstract] small-indel scope: "efficiently detects small indels (< 50 bp) for highly similar samples, which can be validated by simulated datasets"
