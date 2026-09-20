---
name: small-indel-variant-detection
description: Use when highly similar sample sequences (e.g., human mitogenomes) have
  been assembled into an acyclic colored de Bruijn graph and you need to call small
  indels (< 50 bp) and SNPs with graph-coordinate output.
when_to_use_negative:
- Sequence similarity is insufficient (source number far less than variant number)
  — the variant detection model is not applicable
- Variants are large indels, inversions, or reverse tandem duplications — the node-based
  method only addresses small indels (< 50 bp)
- Samples are highly repetitive (e.g., tandem repeats) or highly divergent — sequence
  cutting fragments sequences and detection precision decreases as similarity decreases
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: cSupB
  role: C++ program implementing graph decomposition into colored superbubbles and
    the tri-tuple coordinate system; NVD and RPD variant detection are run through
    cosmo-cSupB
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph (with k=22 for the mitogenome
    datasets) that supplies the acyclic graph with unique start/end nodes on which
    NVD/RPD operate
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
- name: MUSCLE
  role: Alignment-based variant identification used as a comparator for computing
    precision and recall of detected indels
- name: MAFFT-GINSI
  role: Alignment-based variant identification comparator for precision/recall evaluation
- name: Clustal Omega
  role: Alignment-based comparator used to identify variants and calculate precision
    and recall rate
- name: Mugsy
  role: Alignment-based comparator used to identify variants and calculate precision
    and recall rate
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:1169008be46cc12fe5dfb7b01c47f1668db7593af2dd478eceb2f87cea163af6
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# small-indel-variant-detection

## Summary

Node-based detection of small insertions and deletions (< 50 bp) plus SNPs from a colored de Bruijn graph, using the Nested Variant Detection (NVD) algorithm over cSupB source nodes followed by Reference Position Determination (RPD) to convert node variants into locus variants on the reference genome.

## When to use

Use when highly similar sample sequences (e.g., human mitogenomes) have been assembled into an acyclic colored de Bruijn graph and you need to call small indels (< 50 bp) and SNPs with graph-coordinate output.

## When NOT to use

- Sequence similarity is insufficient (source number far less than variant number) — the variant detection model is not applicable
- Variants are large indels, inversions, or reverse tandem duplications — the node-based method only addresses small indels (< 50 bp)
- Samples are highly repetitive (e.g., tandem repeats) or highly divergent — sequence cutting fragments sequences and detection precision decreases as similarity decreases

## Inputs

- Set of highly similar sample nucleotide sequences (e.g., 12 human mitochondrial genomes: AP008459, AP010675, EF153784, EF153791, EF153794, EF153814, EF397559, EU007868, FJ493500, GQ895144, GU377085, NC_012920)
- Simulated mitogenome datasets with ground-truth variant lists (Additional file 1 simulation protocol, 310 datasets varying SNP/deletion/insertion counts and k)
- Colored de Bruijn graph (.dbg) and color matrix (.colors.sd_vector) constructed by VARI-merge
- Reference variant set for validation (e.g., 1000 Genomes chrMT phase3 VCF)

## Outputs

- Variant calls in VCF format (variant_calls.vcf) where each record carries the tri-tuple coordinate (position, bubid, basecolor)
- Detected small indels (< 50 bp) and SNPs typed as INS (gap1 > 0), DEL (gap2 > 0), or SNP (gap1 = gap2 = 0)
- Per-dataset precision and recall values for indel detection (e.g., indel_precision_recall.tsv) comparable to the Additional file 1 accuracy results

## How to apply

After preprocessing sequences (replace degenerate bases with the most frequent base of other sequences, choose k exceeding the maximum longest repeat segment length K, e.g., k=22 for VARI-merge, and add random ACGT anchor segments of length k to both ends), build a directed, acyclic, nondegenerate colored de Bruijn graph with VARI-merge and decompose it into a hierarchical cSupB tree via postorder-like traversal with color-intersection matching. Run the Nested Variant Detection (NVD) algorithm over source nodes, reading gap information near each source node: gap1 > 0 means INS, gap2 > 0 means DEL, and gap1 = gap2 = 0 means SNP. Then apply Reference Position Determination (RPD) to transform node variants into locus variants, keeping only small indels (< 50 bp). Annotate each detected variant with the tri-tuple coordinate (position, bubid, basecolor) combining offset value, topological structure and sample information, and write calls in VCF format (e.g., variant_calls.vcf). The approach assumes a one-to-one correspondence between variant generation and source node, so validate calls against ground truth (e.g., simulation truth sets or the 1000 Genomes chrMT annotation) by computing precision and recall, optionally comparing against alignment-based identification with MUSCLE, MAFFT-GINSI, Clustal Omega, or Mugsy.

## Related tools

- **cSupB** (C++ program implementing graph decomposition into colored superbubbles and the tri-tuple coordinate system; NVD and RPD variant detection are run through cosmo-cSupB) — https://github.com/eggleader/cSupB
- **VARI-merge** (Constructs the succinct colored de Bruijn graph (with k=22 for the mitogenome datasets) that supplies the acyclic graph with unique start/end nodes on which NVD/RPD operate) — https://github.com/cosmo-team/cosmo/tree/VARI-merge
- **MUSCLE** (Alignment-based variant identification used as a comparator for computing precision and recall of detected indels)
- **MAFFT-GINSI** (Alignment-based variant identification comparator for precision/recall evaluation)
- **Clustal Omega** (Alignment-based comparator used to identify variants and calculate precision and recall rate)
- **Mugsy** (Alignment-based comparator used to identify variants and calculate precision and recall rate)

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- Every variant record carries a complete tri-tuple coordinate (position, bubid, basecolor)
- No called indel exceeds 50 bp
- High precision and recall across simulated datasets (310 mitogenome simulations), with the node-based algorithm achieving best precision, recall, and stability in some parameter sets
- Variant type from gap values is consistent with the expected signature: gap1 > 0 = INS, gap2 > 0 = DEL, gap1 = gap2 = 0 = SNP
- Calls are consistent with the reference variant set (e.g., Additional file 3 for the 12 real mitogenomes; 89 of 110 SNVs mapped to the 1000 Genomes mtDNA annotation, 20 of 21 unmapped SNVs verified to exist, e.g., position 449 T>C in EF397559)

## Limitations

- Applicable only to high-similarity samples with simple variant types; precision decreases when sample similarity decreases and variant types become complicated
- Only small indels are addressed; no effective solution for large indels, inversions, and reverse tandem duplications
- Choice of the k value impacts variant detection accuracy; gene elements shorter than k may map to multiple nodes, making analysis complicated and ambiguous
- Sequence cutting fragments highly repetitive sequences (especially tandem repeats), increasing computational difficulty and error
- Degeneracy cannot be handled efficiently; colored de Bruijn graph construction tools including VARI-merge lack mapping functionality

## Evidence

- [methods] If gap1 > 0, var(a)=INS; if gap2 > 0, var(a)=DEL; and if gap1 = gap2 = 0, var(a)=SNP: [evidence span withheld — anchor status: not_found]
- [abstract] efficiently detects small indels (< 50 bp) for highly similar samples, which can be validated by simulated datasets: "efficiently detects small indels (< 50 bp) for highly similar samples, which can be validated by simulated datasets"
- [discussion] the choice of the k value impacts the accuracy of the algorithm: "the choice of the k value impacts the accuracy of the algorithm"
- [discussion] our variant detection method could only address small indels, but for more complicated conditions, such as large indels, inversions, and reverse tandem duplications, we do not have an effective: "our variant detection method could only address small indels, but for more complicated conditions, such as large indels, inversions, and reverse tandem duplications, we do not have an effective"
- [readme] VARI-cSupB provides a novel method that utilizes complete topological information and efficiently detects variants for highly similar samples: "VARI-cSupB provides a novel method that utilizes complete topological information and efficiently detects variants for highly similar samples"
