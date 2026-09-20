---
name: ortholog-marker-accuracy-scoring
description: Use when recovered marker FASTA sequences from genome-skimming retrieval
  runs (e.g. Patchwork, ALiBaSeq, aTRAM 2 outputs) must be quantitatively scored against
  a curated USCO reference set to compare accuracy across programs or coverages.
when_to_use_negative:
- Reference sequences are not near-universal single-copy orthologs — the authors recommend
  limiting references to near-USCOs to avoid chimeric retrieved sequences, so scoring
  against multi-copy or paralogous loci is out of scope
- The goal is de novo assembly quality assessment rather than marker-retrieval accuracy
  (that belongs to QUAST-based assembly evaluation)
- Retrieved markers are destined for direct tree inference without accuracy benchmarking
  — PREQUAL/MACSE-style cleaning and phylogenetic reconstruction are separate downstream
  steps
edam_operation: http://edamontology.org/operation_0296
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DIAMOND
  role: Performs the ultra-sensitive sequence similarity search of recovered markers
    against the reference USCO database to score identity and coverage
  repo: https://github.com/bbuchfink/diamond
- name: Patchwork
  role: Produces the marker retrieval outputs (from reads or assemblies) that are
    scored by this skill in benchmark comparisons
  repo: https://github.com/fethalen/Patchwork
- name: Slurm
  role: Reports the elapsed runtime per program run recorded alongside accuracy values
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:07795789d793c44bc335b3bf15b4a09ce29a45f3242bc9b490c2faafbc94d542
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - ALiBaSeq
  - BLAST
  - DIAMOND
  - Julia
  - Patchwork
  - QUAST
  - SPAdes
  - Slurm
  - Subsample.jl
  - aTRAM 2
  - exonerate
  edam_operation: http://edamontology.org/operation_0296
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Ortholog marker accuracy scoring

## Summary

Scores recovered phylogenetic marker sequences (e.g. USCO/near-USCO retrievals from Patchwork, ALiBaSeq, or aTRAM 2) by searching them against a validated same-species reference ortholog set with DIAMOND and computing the joint metric pidentical,cov (percent identical aligned positions × reference coverage).

## When to use

Use when recovered marker FASTA sequences from genome-skimming retrieval runs (e.g. Patchwork, ALiBaSeq, aTRAM 2 outputs) must be quantitatively scored against a curated USCO reference set to compare accuracy across programs or coverages.

## When NOT to use

- Reference sequences are not near-universal single-copy orthologs — the authors recommend limiting references to near-USCOs to avoid chimeric retrieved sequences, so scoring against multi-copy or paralogous loci is out of scope
- The goal is de novo assembly quality assessment rather than marker-retrieval accuracy (that belongs to QUAST-based assembly evaluation)
- Retrieved markers are destined for direct tree inference without accuracy benchmarking — PREQUAL/MACSE-style cleaning and phylogenetic reconstruction are separate downstream steps

## Inputs

- Recovered marker sequences (FASTA) from Patchwork, ALiBaSeq, or aTRAM 2 runs
- Reference USCO/near-USCO amino acid set (e.g. 778 recoverable D. gyrociliatus USCOs; 957 H. robusta near-USCOs from GenBank GCA_000326865.1)
- DIAMOND reference database built from the reference marker set
- Per-job Slurm elapsed-time records for runtime benchmarking

## Outputs

- Per-reference best-hit assignment table (highest-bit-score marker per USCO)
- Joint accuracy metric pidentical,cov (percent identical aligned positions × reference coverage) per program/coverage arm
- Total percent identity values compiled into benchmark_comparison.tsv
- Per-program runtime values from Slurm

## How to apply

First strip ambiguous N characters from every recovered marker sequence, since Ns inflate apparent coverage without contributing aligned identity. Search the cleaned markers against the recoverable reference USCO set (here the 778 recoverable D. gyrociliatus USCOs, originally 815 same-species USCOs from GenBank GCA_904063045.1) using DIAMOND with --ultra-sensitive to maximize homolog detection. For each reference sequence, retain only the single recovered marker with the highest bit score, preventing multi-hit inflation. Compute the joint metric pidentical,cov as the percent of aligned positions that are identical multiplied by the fraction of the reference covered; report also total percent identity over all reference positions (e.g. up to 62% for Patchwork assemblies at ≥10× coverage). Log per-program Slurm-elapsed runtimes alongside accuracy so accuracy-vs-runtime trade-offs (e.g. aTRAM 2 failing the 40× run within 5 days) can be judged jointly.

## Related tools

- **DIAMOND** (Performs the ultra-sensitive sequence similarity search of recovered markers against the reference USCO database to score identity and coverage) — https://github.com/bbuchfink/diamond
- **Patchwork** (Produces the marker retrieval outputs (from reads or assemblies) that are scored by this skill in benchmark comparisons) — https://github.com/fethalen/Patchwork
- **Slurm** (Reports the elapsed runtime per program run recorded alongside accuracy values)

## Examples

```
diamond blastx --db dg_usco.dmnd --query recovered_markers.fasta --out hits.tsv --ultra-sensitive --outfmt 6
```

## Evaluation signals

- Every recovered marker has ambiguous N characters removed before the DIAMOND search
- Exactly one marker (the highest-bit-score hit) is kept per reference USCO — no multi-hit double counting
- For the high-quality same-species test, all 815 initial markers are retrieved with ~95.9% average identical aligned positions and 92.2% mean query coverage
- Against distant H. robusta references, markers align back to 1 and only 1 homologous D. gyrociliatus USCO before being counted as accurate
- Benchmark pattern reproduced: Patchwork with assemblies reaches up to 62% total percent identity at ≥10× coverage, ALiBaSeq trails by ~7%, values tabulated in benchmark_comparison.tsv

## Limitations

- The joint metric pidentical,cov conflates identity and coverage, so a short but perfect marker can score lower than a long partial one; report both components when interpreting
- At very low coverage (1×), read-based data outperform assembly-based retrieval, so accuracy conclusions do not transfer across coverage regimes
- aTRAM 2 may fail to complete high-coverage runs (e.g. 40× unfinished after 5 days), and such non-completions must be recorded as missing rather than zero-accuracy results
- No specific cutoff thresholds are given for gap-excluded identity trimming, so threshold choice near high identity values is somewhat arbitrary

## Evidence

- [results] Patchwork with D. gyrociliatus assembly data performs best over almost all data sets (Fig. 4), reaching as much as 62% total percent identity for data with at least 10× coverage: "Patchwork with D. gyrociliatus assembly data performs best over almost all data sets (Fig. 4), reaching as much as 62% total percent identity for data with at least 10× coverage"
- [results] We retrieved all of the initial 815 markers. On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%: "We retrieved all of the initial 815 markers. On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%"
- [results] Of these, 769 successfully aligned back to 1 and only 1 of the 778 D. gyrociliatus USCOs: "Of these, 769 successfully aligned back to 1 and only 1 of the 778 D. gyrociliatus USCOs"
- [readme] Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data.: "Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data."
