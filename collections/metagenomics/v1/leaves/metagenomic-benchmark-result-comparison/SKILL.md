---
name: metagenomic-benchmark-result-comparison
description: Reference for evaluating whether a new k-mer/minhash-based read classifier
  beats established metagenomics tools (CLARK, Kraken2, Kraken2+Bracken) on FP rate
  and abundance estimation accuracy over the same reference genome database.
when_to_use_negative:
- Input reads have no ground truth (unknown composition) — FP rate and quantification
  accuracy cannot be scored; only runtime benchmarking applies
- The task is long-read or contig classification benchmarking where the compared tools
  (CLARK, Kraken2 short-read defaults) are out of scope for the comparison protocol
- The reference database differs between tools — comparisons of FP rate/accuracy would
  be confounded by database content rather than algorithm
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: AFS-MetaCache
  role: C++ workstation implementation of the minhashing classifier under evaluation;
    substitutes the alignment tool in the AFS pipeline
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Apache Spark cluster implementation used to classify reads (query mode, 16
    executors, 4 threads per executor) in the big-data benchmark arm
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: CLARK
  role: Established baseline classifier compared on false-positive rate and quantification
    accuracy
- name: Kraken2
  role: Established baseline classifier compared on false-positive rate and quantification
    accuracy
- name: Kraken2+Bracken
  role: Baseline classifier with abundance re-estimation compared on FP rate and quantification
    accuracy
- name: MegaBLAST
  role: 'Alignment-based speed reference: read classification accuracy comparable
    but around three orders-of-magnitude slower'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:64d239ec84830ebdf7f532be505cc50e84671b46003086d4472fee1d073c984a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MegaBLAST
  - MetaCacheSpark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metagenomic-benchmark-result-comparison

## Summary

Method for benchmarking alignment-free metagenomic classifiers (AFS-MetaCache, MetaCacheSpark) against established tools (CLARK, Kraken2, Kraken2+Bracken, MegaBLAST) by comparing classification outputs on shared reference databases, focusing on false-positive rate and quantification accuracy.

## When to use

Reference for evaluating whether a new k-mer/minhash-based read classifier beats established metagenomics tools (CLARK, Kraken2, Kraken2+Bracken) on FP rate and abundance estimation accuracy over the same reference genome database.

## When NOT to use

- Input reads have no ground truth (unknown composition) — FP rate and quantification accuracy cannot be scored; only runtime benchmarking applies
- The task is long-read or contig classification benchmarking where the compared tools (CLARK, Kraken2 short-read defaults) are out of scope for the comparison protocol
- The reference database differs between tools — comparisons of FP rate/accuracy would be confounded by database content rather than algorithm

## Inputs

- FASTA/FASTQ metagenomic read files (paired reads supported via -r/-pairfiles)
- Reference genome collections (e.g. NCBI RefSeq bacterial, viral, archaea genomes) used to build each tool's database
- Classification output files from each benchmarked tool (CLARK, Kraken2, Kraken2+Bracken, AFS-MetaCache/MetaCacheSpark)

## Outputs

- Per-tool false-positive rate estimates on reads with known origin
- Per-tool quantification/abundance-accuracy results at a selectable taxonomic level (species)
- Runtime/throughput comparison against alignment-based baselines (BWA, Bowtie2, CUSHAW, MegaBLAST)

## How to apply

Run each classifier (AFS-MetaCache on a workstation, MetaCacheSpark on a Spark cluster with 16 executors, plus CLARK, Kraken2 and Kraken2+Bracken baselines) over identical reference genome collections and identical query reads, e.g. paired food-sample reads classified with abundance estimation enabled at species level (-a species) and remove-overpopulated-features (-o) during build. Record per-tool classification outputs and abundance estimates, then quantify false-positive rate (reads assigned to absent taxa, e.g. Actinoalloteichus flagged by the coverage filter) and quantification accuracy against known admixture compositions (chicken, turkey, pork, beef, horse, sheep). The article's key decision point is that a new tool must not merely be fast — it must be superior in FP rate and quantification accuracy, as reported in Figs. 5-6 and Table 11. As a speed baseline, compare wall-clock classification throughput against the alignment-based AFS pipeline (BWA/Bowtie2/CUSHAW per reference genome) and MegaBLAST, where the k-mer method is orders-of-magnitude faster. Validate by confirming the published outcome: AFS-MetaCache/MetaCacheSpark superior on FP rates while retaining classification accuracy.

## Related tools

- **AFS-MetaCache** (C++ workstation implementation of the minhashing classifier under evaluation; substitutes the alignment tool in the AFS pipeline) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Apache Spark cluster implementation used to classify reads (query mode, 16 executors, 4 threads per executor) in the big-data benchmark arm) — https://github.com/jmabuin/MetaCacheSpark
- **CLARK** (Established baseline classifier compared on false-positive rate and quantification accuracy)
- **Kraken2** (Established baseline classifier compared on false-positive rate and quantification accuracy)
- **Kraken2+Bracken** (Baseline classifier with abundance re-estimation compared on FP rate and quantification accuracy)
- **MegaBLAST** (Alignment-based speed reference: read classification accuracy comparable but around three orders-of-magnitude slower)

## Examples

```
./metacache query refseq myReads1.fa myReads2.fa -pairfiles -out results.txt
```

## Evaluation signals

- AFS-MetaCache/MetaCacheSpark must show lower false-positive rates than CLARK, Kraken2, and Kraken2+Bracken on the same database and reads
- Quantification accuracy at species level must not degrade relative to baselines (superior in terms of false-positive rate and quantification accuracy)
- Classification runtime should be orders-of-magnitude faster than the alignment-based AFS pipeline (BWA/Bowtie2/CUSHAW per reference genome)
- Known-admixture benchmarks (chicken, turkey, pork, beef, horse, sheep) must be recovered without spurious taxa such as Actinoalloteichus, which the coverage filter removes
- Both implementations (C++ workstation and Spark cluster) should produce substitutable classification results as documented

## Limitations

- No software version numbers reported for benchmarked tools (Kraken2, CLARK, Bracken, Spark), limiting reproducibility
- Hardware specifications of the big data cluster used for benchmarks are not provided
- No availability statement for the raw sequencing data or reference sample datasets used for the admixture benchmarks
- No specific versions or identifiers of the reference genomes used for database construction are stated

## Evidence

- [abstract] Superiority over baselines: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [discussion] Speed baseline rationale: "The comparison to the established metagenomics tools Kraken2, CLARK, and Kraken2+Bracken shows that AFS-MetaCache and MetaCacheSpark are superior in terms of false positive (FP) rates."
- [intro] Alignment-based baseline pipeline: "the AFS pipeline relies on applying a read alignment tool (such as BWA [7–9], Bowtie2 [10], or CUSHAW [11]) for each considered reference genome"
- [readme] Classifier scope: [evidence span withheld — anchor status: external_doc]
