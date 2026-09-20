---
name: usco-retrieval-benchmark-evaluation
description: Use when you need to quantify how accurately a fragmented or assembled
  genome-skimming query recovers a known set of universal single-copy ortholog (USCO)
  references, producing reproducible accuracy statistics.
when_to_use_negative:
- The query data are already an annotated feature table or gene set rather than unassembled
  contigs or WGS reads — Patchwork's DIAMOND BLASTX and stitching steps are not needed
- You need whole-genome structural comparison rather than retrieval of near-universal
  single-copy ortholog markers
- Reference sequences are not near-USCOs; the article recommends limiting reference
  sequences to near-USCOs to avoid chimeric sequences
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_0622
tools:
- name: Patchwork
  role: Retrieves, stitches, trims, concatenates, and realigns markers; provides the
    accuracy and completeness assessment used for benchmarking
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Performs the translated nucleotide (BLASTX) similarity search of query contigs
    against the USCO protein reference database with --ultra-sensitive --frameshift
    15 --masking 0
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Programming language in which Patchwork is written and run (v1.8.2 in the
    benchmark)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:61eca0f4499affefab5799b5d2b77332dd1770adf9f523ef52fc5fe446fdadf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# usco-retrieval-benchmark-evaluation

## Summary

Benchmark the accuracy and completeness of Patchwork marker retrieval by searching a high-quality query assembly against same-species USCO reference sequences (e.g. 815 D. gyrociliatus USCOs from GCA_904063045.1) and computing percent identical aligned positions, query coverage, and combined (pidentical, cov) measures.

## When to use

Use when you need to quantify how accurately a fragmented or assembled genome-skimming query recovers a known set of universal single-copy ortholog (USCO) references, producing reproducible accuracy statistics.

## When NOT to use

- The query data are already an annotated feature table or gene set rather than unassembled contigs or WGS reads — Patchwork's DIAMOND BLASTX and stitching steps are not needed
- You need whole-genome structural comparison rather than retrieval of near-universal single-copy ortholog markers
- Reference sequences are not near-USCOs; the article recommends limiting reference sequences to near-USCOs to avoid chimeric sequences

## Inputs

- De novo query assembly (e.g. D. gyrociliatus SPAdes v3.15.3, K-mer 55, from Illumina short reads, PRJEB37657)
- Reference protein FASTA of same-species USCOs (e.g. 815 D. gyrociliatus USCOs from GenBank GCA_904063045.1)
- DIAMOND database built from the pooled reference protein sequences

## Outputs

- Retrieved marker sequences (stitched, trimmed, concatenated, and realigned) for each reference USCO
- Accuracy statistics: percent identical aligned positions (e.g. 95.9% mean)
- Completeness statistics: mean query coverage (e.g. 92.2%) and combined (pidentical, cov) measure (e.g. 88.4% identical matches over all reference positions)
- Count of successfully retrieved markers relative to the reference set (e.g. 815/815)

## How to apply

Build a DIAMOND database from the pooled reference protein sequences (e.g. 815 D. gyrociliatus USCOs from GenBank GCA_904063045.1) and run a DIAMOND BLASTX search of the translated nucleotide query assembly (SPAdes v3.15.3, K-mer 55) with --ultra-sensitive --frameshift 15 --masking 0, keeping default BLOSUM62 scoring (gap open 11, gap extension 1). Let Patchwork v0.5.1 (under Julia v1.8.2, DIAMOND v2.0.13) perform hit stitching — retaining only the highest-scoring segment pair per region — remove unaligned residues, ambiguous amino acids, and stop codons, trim noncoding stretches with the sliding-window algorithm, then concatenate and realign remaining nonoverlapping regions. Assess recovered markers with Patchwork's accuracy and completeness assessment, computing percent identical aligned positions, mean query coverage, and identical matches over all reference positions. The benchmark passes when results reproduce reference values: all 815 markers retrieved, 95.9% mean identical aligned positions, 92.2% mean query coverage, and 88.4% identical matches over all reference positions.

## Related tools

- **Patchwork** (Retrieves, stitches, trims, concatenates, and realigns markers; provides the accuracy and completeness assessment used for benchmarking) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Performs the translated nucleotide (BLASTX) similarity search of query contigs against the USCO protein reference database with --ultra-sensitive --frameshift 15 --masking 0) — https://github.com/bbuchfink/diamond
- **Julia** (Programming language in which Patchwork is written and run (v1.8.2 in the benchmark))

## Examples

```
diamond blastx --db usco_815.dmnd --query dyro_assembly.fasta --ultra-sensitive --frameshift 15 --masking 0
```

## Evaluation signals

- All reference markers are retrieved (815/815 for the D. gyrociliatus same-species benchmark)
- Mean identical aligned positions ≈ 95.9% across retrieved markers
- Mean query coverage ≈ 92.2% across retrieved markers
- Combined (pidentical, cov) measure ≈ 88.4% identical matches over all reference positions
- Most individual markers sit well above the 90% aligned position trimming threshold

## Limitations

- Retrieval quality degrades for distant references: against H. robusta references only 943 of 957 sequences were retrieved, 769 aligned back uniquely, with 89% identity and 74.5% coverage
- At very low coverage (1x), read-based data may outperform assembly-based marker retrieval
- Disabling tantan masking (--masking 0) improves query coverage especially for read data, so results depend on masking settings
- Using reference sequences beyond near-USCOs risks chimeric sequences

## Evidence

- [results] same-species benchmark values: "We retrieved all of the initial 815 markers. On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%"
- [methods] reference set provenance: "815 D. gyrociliatus USCOs retrieved from the published high-quality genome assembly (GenBank, accession: GCA_904063045.1)"
- [methods] stitching mechanism: "all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained"
- [methods] trimming algorithm: "we have implemented a sliding window–based alignment trimming approach to rid the alignments from these unwanted regions"
- [readme] tool description: "Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data."
- [results] masking setting rationale: "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
