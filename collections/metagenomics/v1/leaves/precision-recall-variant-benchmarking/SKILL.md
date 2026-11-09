---
name: precision-recall-variant-benchmarking
description: Use when you need to quantify detection accuracy (precision, recall,
  stability) of a variant-calling method on simulated datasets with known SNP/insertion/deletion
  ground truth, varying variant counts and k.
when_to_use_negative:
- Sequences have insufficient similarity (source number far less than variant number),
  since the variant detection model is not applicable
- The target variants are large indels, inversions, or reverse tandem duplications
  — only small indels (< 50 bp) are addressed
- The task is de novo variant discovery without ground truth — precision and recall
  cannot be computed without a known truth set
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: cSupB
  role: Colored superbubble decomposition of the colored de Bruijn graph producing
    the variant calls under evaluation
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph (k = 22) that cSupB decomposes
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
- name: MUSCLE
  role: Baseline alignment-based variant identification for precision/recall comparison
- name: MAFFT-GINSI
  role: Baseline alignment-based variant identification for precision/recall comparison
- name: Clustal Omega
  role: Baseline alignment-based variant identification for precision/recall comparison
- name: Mugsy
  role: Baseline alignment-based variant identification for precision/recall comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:d603c338ef869e59d4a993a828a207574d1e708a8e1be706173b9c342adacf5c
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# precision-recall-variant-benchmarking

## Summary

Benchmark small-indel (< 50 bp) variant detection from colored de Bruijn graph-based pipelines by computing precision and recall against simulation ground truth, and compare the graph/node-based method against alignment-based callers (MUSCLE, MAFFT-GINSI, Clustal Omega, Mugsy).

## When to use

Use when you need to quantify detection accuracy (precision, recall, stability) of a variant-calling method on simulated datasets with known SNP/insertion/deletion ground truth, varying variant counts and k.

## When NOT to use

- Sequences have insufficient similarity (source number far less than variant number), since the variant detection model is not applicable
- The target variants are large indels, inversions, or reverse tandem duplications — only small indels (< 50 bp) are addressed
- The task is de novo variant discovery without ground truth — precision and recall cannot be computed without a known truth set

## Inputs

- Simulated sequence datasets with known variant ground truth (numbers of SNPs, deletions, insertions varied per parameter set)
- Reference genome (e.g. human mitochondrial genome NC_012920)
- Variant calls from the method under test (node-based, cSupB/NVD/RPD)
- Variant calls from alignment-based methods (MUSCLE, MAFFT-GINSI, Clustal Omega, Mugsy)

## Outputs

- Per-dataset precision and recall values for indel detection
- Aggregated precision/recall table across simulation parameter sets (indel_precision_recall.tsv)
- Comparative accuracy/stability assessment across methods and k values

## How to apply

Simulate datasets with known ground truth (here, 310 simulated mitogenome sets from 12 human mitochondrial genomes including NC_012920, varying the numbers of SNPs, deletions, insertions, and k, with k = 22 set for VARI-merge so it exceeds the maximum longest repeat segment length). Run the method under test end-to-end: preprocess sequences (replace degenerate bases, add random k-length ACGT anchor segments), build the colored de Bruijn graph with VARI-merge, decompose with cSupB, call variants via the NVD algorithm (gap1 > 0 = INS, gap2 > 0 = DEL, gap1 = gap2 = 0 = SNP) and RPD, and keep only small indels (< 50 bp). Independently identify variants from a multiple sequence alignment produced by MUSCLE, MAFFT-GINSI, Clustal Omega, or Mugsy. Compare detected variants against simulation ground truth per dataset and compute precision and recall per run, then aggregate across parameter sets (e.g. into indel_precision_recall.tsv). Judge success by high precision/recall values comparable to the published accuracy results and by stability across parameter sets.

## Related tools

- **cSupB** (Colored superbubble decomposition of the colored de Bruijn graph producing the variant calls under evaluation) — https://github.com/eggleader/cSupB
- **VARI-merge** (Constructs the succinct colored de Bruijn graph (k = 22) that cSupB decomposes) — https://github.com/cosmo-team/cosmo/tree/VARI-merge
- **MUSCLE** (Baseline alignment-based variant identification for precision/recall comparison)
- **MAFFT-GINSI** (Baseline alignment-based variant identification for precision/recall comparison)
- **Clustal Omega** (Baseline alignment-based variant identification for precision/recall comparison)
- **Mugsy** (Baseline alignment-based variant identification for precision/recall comparison)

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- Detected variant counts and types match the simulated ground truth per dataset (SNP/INS/DEL assigned via gap1/gap2 rules)
- Per-dataset precision and recall fall in the high-accuracy range reported for the simulation parameter sets
- Graph is directed, acyclic, nondegenerate, with unique start and end nodes before variant calling
- Results aggregated into indel_precision_recall.tsv are comparable to the Additional file 1 accuracy results
- Method shows stability (low variance in precision/recall) across varied SNP/INS/DEL counts and k values relative to alignment-based baselines

## Limitations

- Precision decreases when sample similarity decreases and variant types become complicated; the approach suits high-similarity samples with simple variant types
- Only small indels (< 50 bp) are addressed; no effective solution for large indels, inversions, or reverse tandem duplications
- Choice of k value impacts accuracy; gene elements shorter than k may map to multiple nodes, complicating analysis
- Sequence cutting fragments highly repetitive sequences (especially tandem repeats), increasing computational difficulty and error

## Evidence

- [abstract] efficiently detects small indels (< 50 bp) for highly similar samples, which can be validated by simulated datasets: "efficiently detects small indels (< 50 bp) for highly similar samples, which can be validated by simulated datasets"
- [methods] gap rules determine variant type: [evidence span withheld — anchor status: not_found]
- [results] Clustal Omega [45], and Mugsy [46], to identify SNVs, calculate the precision and recall rate: "Clustal Omega [45], and Mugsy [46], to identify SNVs, calculate the precision and recall rate"
- [discussion] when the similarity of the sample decreases, the variant type becomes complicated, and the precision of the algorithm is reduced: "when the similarity of the sample decreases, the variant type becomes complicated, and the precision of the algorithm is reduced"
- [readme] cosmo-cSupB invocation from README: "cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt"
