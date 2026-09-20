---
name: pangenome-triplet-coordinate-annotation
description: Use when variants detected on a colored de Bruijn graph pangenome need
  graph coordinates (position, bubid, basecolor) transformed into locus-level annotations
  in VCF format for highly similar haplotype sequences.
when_to_use_negative:
- Sample sequence similarity is insufficient (number of source nodes far less than
  the number of variants) — the variant detection model is not applicable
- Variants of interest include large indels (≥ 50 bp), inversions, or reverse tandem
  duplications — the node-based detection only addresses small indels
- Highly repetitive sequences, especially tandem repeats — sequence cutting fragments
  them, increasing computational difficulty and error
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph and color matrix from the
    haplotype sequences (run with k=22 for the mitogenome dataset)
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
- name: cSupB
  role: C++ program implementing colored superbubble decomposition, the tri-tuple
    coordinate system, NVD and RPD variant detection, and VCF output
  repo: https://github.com/eggleader/cSupB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:5bb1712522ab6d1fc13330073aa8869c8e1d6421cc8f9ab2d088e5c2fc3ca657
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - VARI-merge
  - cSupB
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pangenome-triplet-coordinate-annotation

## Summary

Assigns each variant detected on a colored de Bruijn graph-based pangenome a tri-tuple coordinate (offset value/position, bubble id, sample color) and writes calls in VCF format, enabling precise graph-to-locus variant annotation as demonstrated on 12 human mitochondrial genomes with VARI-merge and cSupB.

## When to use

Use when variants detected on a colored de Bruijn graph pangenome need graph coordinates (position, bubid, basecolor) transformed into locus-level annotations in VCF format for highly similar haplotype sequences.

## When NOT to use

- Sample sequence similarity is insufficient (number of source nodes far less than the number of variants) — the variant detection model is not applicable
- Variants of interest include large indels (≥ 50 bp), inversions, or reverse tandem duplications — the node-based detection only addresses small indels
- Highly repetitive sequences, especially tandem repeats — sequence cutting fragments them, increasing computational difficulty and error

## Inputs

- Haplotype nucleotide sequences (e.g., 12 human mitochondrial genomes: AP008459, AP010675, EF153784, EF153791, EF153794, EF153814, EF397559, EU007868, FJ493500, GQ895144, GU377085, NC_012920) in FASTA-like form
- Colored de Bruijn graph (.dbg) and color matrix (.colors.sd_vector) built by VARI-merge
- Optional reference variant set for validation (e.g., 1000 Genomes chrMT phase3 VCF, ALL.chrMT.phase3_callmom-v0_4.20130502.genotypes.vcf.gz)

## Outputs

- variant_calls.vcf — variant records annotated with the tri-tuple coordinate (position, bubid, basecolor) in VCF format
- Tri-tuple coordinate assignments for graph elements
- Variant calls classified as SNP, INS, or DEL (small indels < 50 bp)

## How to apply

Construct the colored de Bruijn graph with VARI-merge after preprocessing: replace degenerate bases with the most frequent base across sequences, choose k exceeding the maximum longest repeat segment length (k=22 for the 12 human mitogenomes) so the graph is acyclic, and append random ACGT anchor segments of length k to both ends of each sequence to guarantee unique start and end nodes. Decompose the graph via postorder-like traversal with cSupB matching (source and sink nodes matched by color intersection of adjacent edges), reorganize cSupBs into a hierarchical tree, and resolve cycles with offset-value-based cycle identification followed by sequence cutting and graph reconstruction. Apply the Nested Variant Detection (NVD) algorithm at source nodes — gap1>0 means INS, gap2>0 means DEL, gap1=gap2=0 means SNP — then use Reference Position Determination (RPD) to transform node variants into locus variants, restricted to small indels (<50 bp). Annotate each variant with the tri-tuple graph coordinate and output the calls as a VCF (variant_calls.vcf).

## Related tools

- **VARI-merge** (Constructs the succinct colored de Bruijn graph and color matrix from the haplotype sequences (run with k=22 for the mitogenome dataset)) — https://github.com/cosmo-team/cosmo/tree/VARI-merge
- **cSupB** (C++ program implementing colored superbubble decomposition, the tri-tuple coordinate system, NVD and RPD variant detection, and VCF output) — https://github.com/eggleader/cSupB

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- Each variant record carries a complete tri-tuple coordinate (position, bubid, basecolor)
- No called indel exceeds 50 bp
- Calls match the reference variant set reported in the article's Additional file 3
- For the 12 human mitogenomes, a majority of detected SNVs (89 of 110) map to the 1000 Genomes mtDNA annotation, and most unmapped SNVs (20 of 21) verify as real variants (e.g., position 449 T>C in EF397559)
- The constructed graph is directed, acyclic, nondegenerate, with unique start and end nodes before variant detection begins

## Limitations

- Choice of k value impacts variant detection accuracy
- Gene elements shorter than k may map to multiple nodes, making analysis complicated and ambiguous
- Degeneracy cannot be handled efficiently by the colored de Bruijn graph construction
- Precision decreases when sample similarity decreases and variant types become complicated; applicable only to high-similarity samples with simple variant types
- VARI-merge and other colored de Bruijn graph tools lack mapping functionality, limiting data selection and model promotion

## Evidence

- [methods] k parameter for acyclic graph: "we set k = 22 for VARI-merge to construct an acyclic graph."
- [methods] NVD gap rules: [evidence span withheld — anchor status: not_found]
- [abstract] tri-tuple system definition: "we propose a tri-tuple coordinate system that combines an offset value, topological structure and sample information"
- [readme] repo usage: "cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt"
- [abstract] cSupB repo availability: "available at https://github.com/eggleader/cSupB"
