---
name: kmer-abundance-distance-computation
description: Use when you need all-vs-all pairwise distance matrices between many
  metagenomic (or viromic) FASTA/FASTQ samples based on k-mer composition and abundance,
  especially when datasets are too large for sketch-based or normalization-dependent
  tools.
when_to_use_negative:
- Inputs are assembled contigs rather than reads — the paper's HMP WGS assembled-contig
  comparison (Fig. S3) shows Libra in REDUCE mode with reduced resolution and Simka
  collapsing to maximum distances when abundance cannot be measured from reads
- You need an exact reference-based taxonomic assignment rather than an unsupervised
  sample-to-sample distance matrix
- Datasets are small enough that a non-distributed tool suffices — Libra's Hadoop
  preprocessing/scoring stack adds setup overhead not justified for a handful of samples
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Libra
  role: 'Primary tool: two-stage Hadoop MapReduce pipeline (preprocess k-mer indexing,
    core all-vs-all Cosine Similarity scoring with selectable weighting/scoring)'
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework on which Libra's indexing and scoring stages
    are executed for massive all-vs-all comparisons
- name: Mash
  role: 'Comparison tool: MinHash sketch-based distances used as a baseline; compresses
    distances relative to Libra''s logarithmic weighting'
- name: Simka
  role: 'Comparison tool: computes Jaccard-ab and Bray-Curtis distance matrices; flat
    or saturated distances in the mock and assembled-contig benchmarks'
- name: iMicrobe
  role: Web-based implementation of Libra exposing the analysis over iMicrobe Data
    Commons datasets
- name: CyVerse
  role: Cyberinfrastructure on which the iMicrobe web tool runs Libra under user accounts
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:ecf984c10ed80e9c118e902eadbbeb281c26d9e9420ca69276a49e006bc1ee51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - CyVerse
  - Hadoop
  - Libra
  - Mash
  - Simka
  - iMicrobe
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# kmer-abundance-distance-computation

## Summary

Compute all-vs-all metagenomic pairwise distances from k-mer abundance counts using Libra, a Hadoop-based tool that applies Cosine Similarity over full-read k-mer profiles with selectable weighting (LOGARITHM, NATURAL, BOOLEAN) and scoring (COSINESIMILARITY, BRAYCURTIS, JENSENSHANNON) algorithms. This skill covers the two-stage Libra pipeline (preprocess indexing followed by core scoring) that avoids the resolution loss caused by sketching, read-count normalization, or presence/absence metrics.

## When to use

Use when you need all-vs-all pairwise distance matrices between many metagenomic (or viromic) FASTA/FASTQ samples based on k-mer composition and abundance, especially when datasets are too large for sketch-based or normalization-dependent tools.

## When NOT to use

- Inputs are assembled contigs rather than reads — the paper's HMP WGS assembled-contig comparison (Fig. S3) shows Libra in REDUCE mode with reduced resolution and Simka collapsing to maximum distances when abundance cannot be measured from reads
- You need an exact reference-based taxonomic assignment rather than an unsupervised sample-to-sample distance matrix
- Datasets are small enough that a non-distributed tool suffices — Libra's Hadoop preprocessing/scoring stack adds setup overhead not justified for a handful of samples

## Inputs

- FASTA or FASTQ metagenomic read files for each sample (e.g., simulated 454 HM-277D mock-community reads, HMP WGS reads, Tara Ocean Virome reads)
- HDFS-accessible source directory containing the read files (/source_dir)
- Libra binary (libra-all.jar) run via `hadoop jar`
- User-selected parameters: k-mer size, weighting algorithm (LOGARITHM | BOOLEAN | NATURAL), scoring algorithm (COSINESIMILARITY | BRAYCURTIS | JENSENSHANNON), run mode (MAP | REDUCE)

## Outputs

- K-mer index directory (/index_dir) from the preprocess stage, grouped by file-group size and reducer task count
- All-vs-all pairwise similarity/distance matrix in /score_dir (one similarity value per sample pair, e.g., hmp_wgs_distance_matrices.tsv-style output)
- Clustered sample groupings derived from the distance matrix (e.g., four temperature-gradient clusters of 43 Tara Ocean Viromes)

## How to apply

First run Libra's preprocessing stage (`preprocess`) on FASTA/FASTQ inputs with a chosen k-mer size (-k 20 in the README example), number of reducer tasks (-t), group sizing options (-s, -g), and an optional k-mer filter (-f NONE | STDDEV | STDDEV2 | NOTUNIQUE, NOTUNIQUE by default) to produce a k-mer count index directory. Then run the scoring stage (`core`) on that index, choosing a weighting algorithm (-w LOGARITHM default, NATURAL for raw abundance, or BOOLEAN), a scoring algorithm (-s COSINESIMILARITY default, BRAYCURTIS, or JENSENSHANNON), and a run mode (-m MAP default or REDUCE for large index sets), writing the pairwise similarity matrix to the output directory. The rationale is that Cosine Similarity over logarithmically weighted k-mer abundance vectors considers genetic distance and microbial abundance simultaneously while normalizing for sequencing depth, which the paper shows resolves composition series (e.g., simulated 454 mock communities mock1–mock4 with stepwise distance increases) that compress or saturate under Mash sketches, Simka's Jaccard-ab/Bray-Curtis, or Libra's own NATURAL weighting. For very large collections such as the 43 Tara Ocean Viromes (~4.2 billion reads), run on a Hadoop cluster; k-mer matching completes within ~1.5 hours and the full all-vs-all comparison within ~18 hours on a 10-node cluster. Avoid read duplication (no measurable distance change) and do not substitute assembled contigs, where minimum-count normalization collapses resolution.

## Related tools

- **Libra** (Primary tool: two-stage Hadoop MapReduce pipeline (preprocess k-mer indexing, core all-vs-all Cosine Similarity scoring with selectable weighting/scoring)) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework on which Libra's indexing and scoring stages are executed for massive all-vs-all comparisons)
- **Mash** (Comparison tool: MinHash sketch-based distances used as a baseline; compresses distances relative to Libra's logarithmic weighting)
- **Simka** (Comparison tool: computes Jaccard-ab and Bray-Curtis distance matrices; flat or saturated distances in the mock and assembled-contig benchmarks)
- **iMicrobe** (Web-based implementation of Libra exposing the analysis over iMicrobe Data Commons datasets)
- **CyVerse** (Cyberinfrastructure on which the iMicrobe web tool runs Libra under user accounts)

## Examples

```
hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir; hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- The preprocess stage produces a populated /index_dir and the core stage produces /score_dir containing one similarity value for every sample pair (a complete all-vs-all matrix)
- On the simulated 454 mock-community composition series (mock1–mock4), LOGARITHM weighting yields a stepwise increase in pairwise distance, whereas NATURAL weighting, Mash, and Simka compress or saturate (e.g., maximum difference between mocks 3 and 4)
- Read duplication produces no measurable change in distances (depth-invariance invariant)
- On the 43 Tara Ocean Viromes, clustering the Cosine Similarity matrix reproduces the four temperature-gradient groupings of Fig. 5, with same-gradient samples from different Longhurst provinces clustering together
- Runtime on a 10-node Hadoop cluster falls within the reported profile: k-mer matching ~1.5 hours, end-to-end all-vs-all comparison ~18 hours (Table 1)

## Limitations

- Presence/absence-style reductions, read-count normalization, and data reduction diminish resolution — BOOLEAN weighting and min-count normalized metrics lose the abundance signal that LOGARITHM Cosine Similarity preserves
- Requires a Hadoop cluster for massive datasets; on assembled contigs (no read counts) Libra shows discrepancies and reduced resolution vs. Mash's body-site clustering
- K-mer filter choice (NONE | STDDEV | STDDEV2 | NOTUNIQUE) and group sizing (-s/-g, 10GB default group size, 20 max groups) affect index structure and must be chosen per dataset

## Evidence

- [abstract] Cosine Similarity on sequence composition and abundance, normalized for sequencing depth: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [abstract] Data reduction diminishes resolution: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence"
- [abstract] Libra uses all reads in Hadoop: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses"
- [methods] Scoring options: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] README scoring command: [evidence span withheld — anchor status: external_doc]
- [readme] README weighting options: "w : weighting algorithm. LOGARITHM (default) | BOOLEAN | NATURAL"
