---
name: marker-recovery-accuracy-evaluation
description: Use when you need to quantify marker-retrieval accuracy of a phylogenomic
  mining tool (e.g. Patchwork) against a divergent reference species, using recovery
  counts, unique reciprocal alignments, percent identity, and query coverage as validation
  metrics.
when_to_use_negative:
- The reference set contains non-near-USCO sequences — the article recommends limiting
  reference sequences to near-USCOs to avoid chimeric sequences.
- Query and reference are the same species and high quality — for that scenario the
  article's own-species benchmark (815 same-species USCOs, 95.9% identity, 92.2% coverage)
  is the appropriate comparison, not the divergent-reference benchmark.
- You want to compare against other tools on low-coverage read data rather than evaluate
  a single assembly-based retrieval accuracy profile.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0194
tools:
- name: Patchwork
  role: Retrieves and concatenates phylogenetic markers from the query assembly against
    DIAMOND-formatted reference USCOs; run with benchmark options --ultra-sensitive
    --frameshift 15 --masking 0
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Underlying sequence aligner used by Patchwork (BLASTX against the near-USCO
    protein database) and used directly in --ultra-sensitive mode to define the recoverable
    USCO set and score reciprocal alignments
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Implementation language of Patchwork (v.1.8.2 used for the benchmark)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:ff96f570ec7b9d30e65dd22a3ac9b0c9feb939ea6731aa0be9e267c0a8b6ffda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# marker-recovery-accuracy-evaluation

## Summary

Benchmarking skill that quantifies how well a marker-retrieval tool recovers phylogenetic markers when query and reference species are divergent, by searching a de novo query assembly against distant near-USCO references and scoring retrieval, uniqueness, identity, and coverage metrics.

## When to use

Use when you need to quantify marker-retrieval accuracy of a phylogenomic mining tool (e.g. Patchwork) against a divergent reference species, using recovery counts, unique reciprocal alignments, percent identity, and query coverage as validation metrics.

## When NOT to use

- The reference set contains non-near-USCO sequences — the article recommends limiting reference sequences to near-USCOs to avoid chimeric sequences.
- Query and reference are the same species and high quality — for that scenario the article's own-species benchmark (815 same-species USCOs, 95.9% identity, 92.2% coverage) is the appropriate comparison, not the divergent-reference benchmark.
- You want to compare against other tools on low-coverage read data rather than evaluate a single assembly-based retrieval accuracy profile.

## Inputs

- 957 near-USCO reference protein sequences from the annotated H. robusta genome (GenBank GCA_000326865.1), formatted as a DIAMOND database
- D. gyrociliatus SPAdes de novo assembly (K-mer size 55) as query contigs (FASTA DNA)
- 778 'recoverable' D. gyrociliatus USCOs (GenBank GCA_904063045.1) as the reciprocal-alignment reference set

## Outputs

- patchwork_h_robusta_benchmark_metrics.tsv with per-marker percent identical positions and reference coverage
- Aggregated benchmark statistics: retrieval count (n/N), uniquely aligned marker count, mean percent identity, mean query coverage, and combined pidentical,cov metric

## How to apply

Build a DIAMOND-formatted database from the distant reference marker set (e.g. 957 H. robusta near-USCO protein sequences from GenBank GCA_000326865.1) and run Patchwork (v.0.5.1, Julia v.1.8.2, DIAMOND v.2.0.13) with the query assembly — such as the D. gyrociliatus SPAdes de novo assembly with K-mer size 55 — using --ultra-sensitive --frameshift 15 --masking 0 (tantan masking disabled), which uses DIAMOND BLASTX with the default BLOSUM62 matrix, gap open penalty 11, and gap extension penalty 1. Restrict evaluation to 'recoverable' query markers (e.g. the 778 D. gyrociliatus USCOs from GCA_904063045.1 that produced a hit under DIAMOND --ultra-sensitive), then search recovered markers back against this reference set, retaining only the highest bit-score marker per reference sequence. Compute the combined metric pidentical,cov = nmatch/naligned x cov (total recovered marker length / total reference USCO length), and record per-marker percent identical positions and reference coverage. Aggregate into a metrics table and verify reported values reproduce (here: 943/957 retrieved, 769/778 uniquely aligned, ~89% mean identity, ~74.5% mean query coverage, ~67.2% combined pidentical,cov).

## Related tools

- **Patchwork** (Retrieves and concatenates phylogenetic markers from the query assembly against DIAMOND-formatted reference USCOs; run with benchmark options --ultra-sensitive --frameshift 15 --masking 0) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Underlying sequence aligner used by Patchwork (BLASTX against the near-USCO protein database) and used directly in --ultra-sensitive mode to define the recoverable USCO set and score reciprocal alignments) — https://github.com/bbuchfink/diamond
- **Julia** (Implementation language of Patchwork (v.1.8.2 used for the benchmark))

## Examples

```
diamond blastx --db h_robusta_nearUSCO.dmnd --query Dgyro_spades_k55.fasta --ultra-sensitive --frameshift 15 --masking 0
```

## Evaluation signals

- Retrieval count must equal 943 of 957 H. robusta reference sequences when using the D. gyrociliatus assembly against distant references.
- Exactly 769 of the 778 recoverable D. gyrociliatus USCOs should have a recovered marker aligning back to 1 and only 1 homologous USCO (highest bit-score marker retained per reference).
- Mean percent identical aligned positions should be approximately 89% and mean query coverage approximately 74.5%.
- Combined metric pidentical,cov should be approximately 67.2%, matching the article's reference-divergence benchmark.
- patchwork_h_robusta_benchmark_metrics.tsv must contain per-marker percent identical positions and reference coverage for all recovered markers.

## Limitations

- Even with --ultra-sensitive, only 778 D. gyrociliatus USCOs were 'recoverable' against the distant H. robusta references, so evaluation is restricted to this homologous subset; 14 of 957 references were not retrieved at all.
- Alignment trimming applies a 90% aligned-positions threshold, so markers below that threshold are affected.
- Tantan masking in DIAMOND reduces query coverage, especially for read data, hence --masking 0 must be used for fair coverage comparison.
- Disabling tantan masking is required for this benchmark's reproducibility; other masking settings will not reproduce the reported values.

## Evidence

- [results] retrieval count and unique-alignment validation: "We retrieved 943 out of the 957 H. robusta reference sequences. Of these, 769 successfully aligned back to 1 and only 1 of the 778 D. gyrociliatus USCOs"
- [results] subsampled-coverage benchmark context: "we also subsampled the aforementioned D. gyrociliatus short reads in order to simulate various sequencing coverages"
- [results] masking caveat affecting coverage: "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [readme] Patchwork purpose and scope: "Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data."
- [methods] reference dataset provenance: "815 D. gyrociliatus USCOs retrieved from the published high-quality genome assembly (GenBank, accession: GCA_904063045.1)"
