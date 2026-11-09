---
name: metagenomic-all-vs-all-clustering
description: Use when you need to cluster many large whole-genome shotgun metagenomes
  (billions of reads, e.g., FASTQ/FASTA) all-vs-all by k-mer composition and abundance,
  where sketching, data reduction, or presence/absence metrics would diminish resolution.
when_to_use_negative:
- Datasets small enough for single-node all-vs-all tools (Compareads, Commet, MetaFast,
  Mash/Sourmash, Simka) — the Hadoop overhead is unnecessary
- The analysis requires presence/absence or MinHash-sketch distances — Libra intentionally
  uses all reads and k-mer abundance instead, so it addresses a different resolution
  trade-off
- Input is already a distance matrix or feature table rather than raw sequencing reads
  — the k-mer indexing stage is not applicable
edam_operation: http://edamontology.org/operation_0291
tools:
- name: Libra
  role: 'Performs the all-vs-all k-mer-based metagenome comparison: preprocess/indexing
    and cosine-similarity scoring via Hadoop MapReduce'
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework on which Libra's indexing and scoring stages
    are executed
- name: iMicrobe
  role: Web-based implementation where users can run Libra on data in the iMicrobe
    Data Commons
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:abf9043cb7795335a104aca0cbf78396f4b724a9d241426255189588197f5e41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - iMicrobe
  edam_operation: http://edamontology.org/operation_0291
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metagenomic-all-vs-all-clustering

## Summary

Perform a scalable all-vs-all comparison of metagenomes based on k-mer content using Libra's Hadoop MapReduce implementation with Cosine Similarity distance, enabling precise clustering of massive datasets (e.g., 43 Tara Ocean Viromes, ~4.2 billion reads).

## When to use

Use when you need to cluster many large whole-genome shotgun metagenomes (billions of reads, e.g., FASTQ/FASTA) all-vs-all by k-mer composition and abundance, where sketching, data reduction, or presence/absence metrics would diminish resolution.

## When NOT to use

- Datasets small enough for single-node all-vs-all tools (Compareads, Commet, MetaFast, Mash/Sourmash, Simka) — the Hadoop overhead is unnecessary
- The analysis requires presence/absence or MinHash-sketch distances — Libra intentionally uses all reads and k-mer abundance instead, so it addresses a different resolution trade-off
- Input is already a distance matrix or feature table rather than raw sequencing reads — the k-mer indexing stage is not applicable

## Inputs

- QC-filtered FASTA/FASTQ metagenome sequencing reads (e.g., 43 Tara Ocean Viromes, ~4.2 billion reads)
- Libra k-mer index directory (/index_dir) produced by the preprocess stage
- Hadoop cluster (e.g., 10 nodes) on which to run the MapReduce stages

## Outputs

- All-vs-all cosine-similarity distance matrix written to the score output directory (/score_dir)
- Clustering/dendrogram of samples (e.g., four temperature-gradient groupings of the 43 TOVs, Fig. 5)
- Execution-time profile for the k-mer matching and end-to-end comparison stages

## How to apply

Quality-filter the reads first (for the Tara Ocean Viromes, the Brum et al. protocol applied to the 43 TOVs from the 2009-2011 Expedition). Build the k-mer index by running Libra's Hadoop preprocess stage (default k-mer size 20, NOTUNIQUE k-mer filter, 10 GB group size, max 20 groups) on the QC'd reads to produce an index directory. Run the scoring stage in MAP run mode (default) with the COSINESIMILARITY algorithm (default) and LOGARITHM weighting (default), which compares sequence composition and abundance while normalizing for sequencing depth. Assemble the cosine-similarity distance matrix from the score directory and cluster the samples (e.g., 43 viromes into the four temperature-gradient groupings of Fig. 5). Judge success by recovering biologically coherent clusters and matching reported runtimes (k-mer matching ~1.5 h; end-to-end ~18 h for the TOV analysis on a 10-node Hadoop cluster).

## Related tools

- **Libra** (Performs the all-vs-all k-mer-based metagenome comparison: preprocess/indexing and cosine-similarity scoring via Hadoop MapReduce) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework on which Libra's indexing and scoring stages are executed)
- **iMicrobe** (Web-based implementation where users can run Libra on data in the iMicrobe Data Commons)

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Reproduction of the four temperature-gradient clusters of the 43 Tara Ocean Viromes (Fig. 5), with samples from different Longhurst provinces but the same temperature gradient clustering together
- K-mer matching stage completes within ~1.5 hours on the 10-node Hadoop cluster (Table 1)
- End-to-end comparison completes within ~18 hours on the 10-node Hadoop cluster (Table 1)
- Score output directory contains a complete all-vs-all (43 x 43) cosine-similarity matrix with valid COSINESIMILARITY scores

## Limitations

- Requires a Hadoop cluster; the TOV analysis was profiled on 10 nodes (~1.5 h k-mer matching, ~18 h end-to-end), so smaller deployments need a different cost model
- Alternative scoring algorithms (BRAYCURTIS, JENSENSHANNON) and weighting schemes (BOOLEAN, NATURAL) exist, but the validated default is COSINESIMILARITY with LOGARITHM weighting
- Preprocessing caps the number of index groups at 20 by default (combined if the 10 GB group size yields more), which constrains index partitioning choices

## Evidence

- [abstract] Libra performs all-vs-all k-mer clustering: "We developed a tool called Libra that performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content."
- [intro] Ocean viral communities structured by temperature gradients: "We show for the first time that viral communities in the ocean are similar across temperature gradients, irrespective of their location in the ocean."
- [methods] Scoring algorithm options: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] Scoring stage CLI invocation: [evidence span withheld — anchor status: external_doc]
