---
name: ortholog-marker-retrieval-from-genomic-assemblies
description: Use when you need to mine (near-)universal single-copy ortholog markers
  from a de novo short-read assembly (e.g. SPAdes contigs) or raw WGS reads for phylogenomics,
  with DNA-to-protein similarity search against a curated OrthoDB-based USCO reference
  set.
when_to_use_negative:
- Reference sequences are not near-USCOs — the article recommends limiting references
  to near-USCOs to avoid chimeric sequences
- You need gene annotation or full gene-model prediction rather than alignment-based
  marker mining (gene-prediction tools like GeMoMa target a different task)
- The query is already a fully annotated genome from which BUSCO could directly extract
  reference sequences for downstream use
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0160
tools:
- name: Patchwork
  role: Alignment-based retrieval, hit stitching, trimming, concatenation, and accuracy
    assessment of USCO markers from the query assembly
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: BLASTX sequence similarity search of translated assembly contigs against protein
    references with --ultra-sensitive --frameshift 15 --masking 0
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Programming language in which Patchwork is written and run (v1.8.2 in the
    benchmark)
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:2c0635600b0f1bf27360d29e5ce07e3fa81cb943ab7817362bac8e39062c8d3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# ortholog-marker-retrieval-from-genomic-assemblies

## Summary

Retrieve universal single-copy orthologs (USCOs) from fragmented or high-quality genome assemblies by searching translated nucleotide contigs against amino acid reference markers with Patchwork (DIAMOND BLASTX), then stitching, trimming, and evaluating retrieval accuracy against same-species or divergent references.

## When to use

Use when you need to mine (near-)universal single-copy ortholog markers from a de novo short-read assembly (e.g. SPAdes contigs) or raw WGS reads for phylogenomics, with DNA-to-protein similarity search against a curated OrthoDB-based USCO reference set.

## When NOT to use

- Reference sequences are not near-USCOs — the article recommends limiting references to near-USCOs to avoid chimeric sequences
- You need gene annotation or full gene-model prediction rather than alignment-based marker mining (gene-prediction tools like GeMoMa target a different task)
- The query is already a fully annotated genome from which BUSCO could directly extract reference sequences for downstream use

## Inputs

- De novo genome assembly in FASTA format (e.g. SPAdes v3.15.3 assembly of D. gyrociliatus Illumina reads, K-mer size 55)
- Amino acid reference FASTA of USCO/near-USCO protein sequences (e.g. 815 D. gyrociliatus USCOs from GenBank GCA_904063045.1 or 957 H. robusta near-USCOs from GenBank GCA_000326865.1)
- DIAMOND-formatted protein reference database

## Outputs

- Recovered and concatenated marker sequences, one continuous sequence per reference USCO
- Benchmark metrics table (e.g. patchwork_h_robusta_benchmark_metrics.tsv) with per-marker percent identical positions and reference coverage
- Aggregate statistics: retrieval count (e.g. 943/957), uniquely aligned markers (e.g. 769/778), mean percent identical aligned positions, mean query coverage, and combined pidentical,cov (e.g. 67.2%)

## How to apply

Build a DIAMOND-formatted protein database from pooled reference USCO FASTA sequences (e.g. 815 D. gyrociliatus USCOs from GCA_904063045.1, or 957 H. robusta near-USCOs from GCA_000326865.1), then run Patchwork v0.5.1 (Julia v1.8.2, DIAMOND v2.0.13) with the assembly as query using --ultra-sensitive --frameshift 15 --masking 0 (tantan masking disabled), keeping default BLOSUM62 scoring with gap open 11 and gap extension 1. Patchwork performs hit stitching (retaining only the highest-scoring segment pair per region), removes unaligned residues, ambiguous amino acids and stop codons, trims noncoding stretches via the sliding-window algorithm, and concatenates and realigns nonoverlapping regions. Evaluate accuracy by searching recovered markers back to the reference USCOs in DIAMOND --ultra-sensitive mode, keeping the highest bit-score marker per reference, and compute pidentical,cov = nmatch/naligned x cov. For divergent references, restrict evaluation to 'recoverable' query USCOs that produce a hit under --ultra-sensitive (778 of the D. gyrociliatus set in the benchmark). Validation thresholds are benchmark-specific: 943/957 retrieved and 769/778 uniquely aligned markers with ~89% identity and ~74.5% coverage against distant H. robusta; all 815 retrieved with 95.9% identity and 92.2% coverage against same-species references.

## Related tools

- **Patchwork** (Alignment-based retrieval, hit stitching, trimming, concatenation, and accuracy assessment of USCO markers from the query assembly) — https://github.com/fethalen/Patchwork
- **DIAMOND** (BLASTX sequence similarity search of translated assembly contigs against protein references with --ultra-sensitive --frameshift 15 --masking 0) — https://github.com/bbuchfink/diamond
- **Julia** (Programming language in which Patchwork is written and run (v1.8.2 in the benchmark)) — https://github.com/fethalen/Patchwork

## Examples

```
diamond blastx -d h_robusta_near_usco.dmnd -q dgyrociliatus_spades_k55.fasta --ultra-sensitive --frameshift 15 --masking 0 -o patchwork_hits.tsv
```

## Evaluation signals

- Retrieval count matches the benchmark: 943 of 957 H. robusta references retrieved; 815/815 against same-species D. gyrociliatus references
- Uniquely aligned markers: 769 markers aligning to exactly 1 of the 778 recoverable D. gyrociliatus USCOs in the divergent-reference benchmark
- Mean percent identical aligned positions ~89% (divergent) or 95.9% (same-species); mean query coverage ~74.5% or 92.2%
- Combined metric pidentical,cov = nmatch/naligned x cov ≈ 67.2% (divergent) or 88.4% (same-species) over all reference positions
- Most recovered markers fall above the 90% aligned-position trimming threshold applied by the sliding-window trimming step

## Limitations

- Against highly divergent references (D. gyrociliatus vs. the distant leech H. robusta), 14 of 957 references were not retrieved and only 769 of 778 considered homologous USCOs aligned uniquely
- Noncoding or artifactual regions can persist in alignments; the article suggests PREQUAL could be used to detect and remove such regions downstream
- Lower-complexity masking trade-off: disabling tantan masking (--masking 0) improves query coverage especially for read data, but was chosen deliberately for this benchmark

## Evidence

- [abstract] Alignment-based marker mining rationale: "We here present a new software tool called Patchwork for mining phylogenetic markers from highly fragmented short-read assemblies"
- [readme] Program description: "Patchwork is an alignment-based program for retrieving and concatenating
phylogenetic markers from whole-genome sequencing (WGS) data."
- [results] Divergent-reference benchmark result: "We retrieved 943 out of the 957 H. robusta reference sequences. Of these, 769 successfully aligned back to 1 and only 1 of the 778 D. gyrociliatus USCOs"
- [results] Masking filter step: "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [discussion] Reference-choice caveat: "we recommend that the user limits the use of reference sequences to near-USCOs"
