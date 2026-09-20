---
name: cross-tool-runtime-profiling-on-hpc
description: Use when benchmarking multiple phylogenomic marker-mining tools (e.g.
  Patchwork vs ALiBaSeq vs aTRAM 2) for both accuracy and runtime across a sequencing-coverage
  gradient on a Slurm-managed HPC cluster.
when_to_use_negative:
- If the goal is to retrieve markers for phylogeny from a single data set rather than
  to compare tool performance — use Patchwork directly instead of the benchmark harness
- If your data are not paired-end short reads with a computable genome-size-based
  coverage target, the 1x–40x subsampling gradient is meaningless
- If a candidate tool cannot run within the imposed wall-clock cap (aTRAM 2 failed
  to finish the 40x arm within 5 days), runtime figures will be dominated by non-completion
  rather than comparative speed
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Patchwork
  role: Benchmark subject (v.0.5.1, Julia v.1.8.2); run on reads and assemblies against
    DIAMOND near-USCO database with --ultra-sensitive --frameshift 15 --masking 0
  repo: https://github.com/fethalen/Patchwork
- name: ALiBaSeq
  role: Benchmark subject (v.1.2); assembly-based protein search via BLAST 2.11.0
    with documented settings, no reciprocal search
- name: aTRAM 2
  role: Benchmark subject (v.2.4.3); read-based iterative retrieval with BLAST 2.11.0,
    exonerate 2.2.0, and SPAdes under the 5-day cap
- name: DIAMOND
  role: Alignment engine for Patchwork searches and for scoring recovered markers
    against the 778 D. gyrociliatus USCOs with --ultra-sensitive
  repo: https://github.com/bbuchfink/diamond
- name: SPAdes
  role: Produces the per-coverage short-read-only assemblies (K-mer 33) used by the
    assembly arms and by aTRAM 2
- name: Subsample.jl
  role: Randomly selects corresponding read pairs to generate the 1x–40x coverage
    data sets
- name: Slurm
  role: Scheduler whose per-job elapsed-time reporting supplies the recorded runtime
    values
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:8279424d86318656a9015581970e069dbe5667746b1dea48f53a1f1b9ac7d705
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cross-tool-runtime-profiling-on-hpc

## Summary

A benchmarking protocol that profiles marker-retrieval accuracy and wall-clock runtime of Patchwork, ALiBaSeq v.1.2, and aTRAM 2 v.2.4.3 across D. gyrociliatus short-read data subsampled to 1x–40x coverage, with per-tool Slurm-elapsed times compiled into benchmark_comparison.tsv.

## When to use

Use when benchmarking multiple phylogenomic marker-mining tools (e.g. Patchwork vs ALiBaSeq vs aTRAM 2) for both accuracy and runtime across a sequencing-coverage gradient on a Slurm-managed HPC cluster.

## When NOT to use

- If the goal is to retrieve markers for phylogeny from a single data set rather than to compare tool performance — use Patchwork directly instead of the benchmark harness
- If your data are not paired-end short reads with a computable genome-size-based coverage target, the 1x–40x subsampling gradient is meaningless
- If a candidate tool cannot run within the imposed wall-clock cap (aTRAM 2 failed to finish the 40x arm within 5 days), runtime figures will be dominated by non-completion rather than comparative speed

## Inputs

- Trimmed paired-end Illumina short reads from D. gyrociliatus (ENA PRJEB37657)
- Coverage-subsampled read pairs (1x, 3x, 5x, 10x, 20x, 40x) generated with Subsample.jl
- Per-coverage SPAdes assemblies (K-mer 33) with QUAST quality reports
- DIAMOND database built from 957 H. robusta near-USCO reference sequences (GenBank GCA_000326865.1)
- Evaluation reference set: 778 recoverable D. gyrociliatus USCOs (from GenBank GCA_904063045.1)

## Outputs

- Recovered marker sequences per tool, per coverage arm (reads- and assembly-based)
- benchmark_comparison.tsv with pidentical,cov accuracy values and Slurm-reported elapsed runtimes per tool and coverage
- Per-tool completeness flags (e.g. aTRAM 2 40x run recorded as non-completing after the 5-day cap)

## How to apply

Subsample trimmed D. gyrociliatus Illumina reads (PRJEB37657) into paired coverage arms (1x, 3x, 5x, 10x, 20x, 40x) with Subsample.jl, then build a short-read-only SPAdes assembly (K-mer 33) per arm and check quality with QUAST. Run each tool on identical inputs: ALiBaSeq with BLAST 2.11.0 using the protein-search settings '-x a -f S -e 1e-10 -c 1 --amalgamate-hits --is -ac aa-tdna' without reciprocal search; aTRAM 2 v.2.4.3 with '--evalue 1e-10 --file-filter "*.filtered contigs.fasta" --overlap N', capping the 40x run at 5 days and recording it as non-completing if unfinished; and Patchwork v.0.5.1 (Julia v.1.8.2) in a single one-step command with '--ultra-sensitive --frameshift 15 --masking 0' on both reads and assemblies against a DIAMOND database of the 957 H. robusta near-USCOs. Score accuracy uniformly by stripping ambiguous N characters, searching recovered markers against the 778 recoverable D. gyrociliatus USCOs with DIAMOND --ultra-sensitive, keeping the highest-bit-score marker per reference, and computing the joint metric pidentical,cov (percent identical aligned positions times reference coverage). Record per-program elapsed times as reported by Slurm and tabulate all accuracy and runtime values into benchmark_comparison.tsv to expose the coverage/accuracy/runtime trade-offs.

## Related tools

- **Patchwork** (Benchmark subject (v.0.5.1, Julia v.1.8.2); run on reads and assemblies against DIAMOND near-USCO database with --ultra-sensitive --frameshift 15 --masking 0) — https://github.com/fethalen/Patchwork
- **ALiBaSeq** (Benchmark subject (v.1.2); assembly-based protein search via BLAST 2.11.0 with documented settings, no reciprocal search)
- **aTRAM 2** (Benchmark subject (v.2.4.3); read-based iterative retrieval with BLAST 2.11.0, exonerate 2.2.0, and SPAdes under the 5-day cap)
- **DIAMOND** (Alignment engine for Patchwork searches and for scoring recovered markers against the 778 D. gyrociliatus USCOs with --ultra-sensitive) — https://github.com/bbuchfink/diamond
- **SPAdes** (Produces the per-coverage short-read-only assemblies (K-mer 33) used by the assembly arms and by aTRAM 2)
- **Subsample.jl** (Randomly selects corresponding read pairs to generate the 1x–40x coverage data sets)
- **Slurm** (Scheduler whose per-job elapsed-time reporting supplies the recorded runtime values)

## Examples

```
diamond blastx --ultra-sensitive -q recovered_markers.fasta -d dg_usco_778.dmnd --out hits.tsv
```

## Evaluation signals

- benchmark_comparison.tsv contains a pidentical,cov value and a Slurm-reported elapsed time for every tool x coverage-arm combination
- Reproduces the headline pattern: Patchwork with D. gyrociliatus assembly data reaches up to 62% total percent identity at >=10x coverage, with ALiBaSeq trailing by approximately 7%
- The 40x aTRAM 2 run is explicitly recorded as non-completing after the 5-day cutoff rather than silently dropped
- At the 1x arm, the read-based data set outperforms the assembly-based one for marker retrieval, matching the reported inversion of the coverage trend
- Marker scoring respects the stated evaluation chain: N characters removed, highest-bit-score marker kept per reference, and DIAMOND --ultra-sensitive against the 778-USCO set

## Limitations

- aTRAM 2 failed to complete the 40x coverage data set within 5 days, so its runtime is right-censored at the cap
- Patchwork with assembly data underperforms read-based data at only 1x coverage, so conclusions from the high-coverage arms do not extrapolate to the lowest coverage
- Disabling tantan masking (--masking 0) was needed to improve query coverage for read data, so parameters are not tool-default and must be reported alongside runtimes

## Evidence

- [results] Patchwork with assembly data reaches up to 62% total percent identity at >=10x coverage, outperforming ALiBaSeq and aTRAM 2 over almost all data sets: "Patchwork with D. gyrociliatus assembly data performs best over almost all data sets (Fig. 4), reaching as much as 62% total percent identity for data with at least 10× coverage"
- [results] ALiBaSeq trails Patchwork assembly runs by roughly 7 percentage points across all data sets: "ALiBaSeq achieves approximately 7% less total percent identity than Patchwork with assemblies for all data sets"
- [results] The coverage gradient is simulated by subsampling the D. gyrociliatus short reads: "we also subsampled the aforementioned D. gyrociliatus short reads in order to simulate various sequencing coverages"
- [results] At the lowest coverage, read-based retrieval beats the assembly-based arm: "Only with a coverage of 1×, D. gyrociliatus read-based data seem to be better suited for marker retrieval"
- [other] The 40x aTRAM 2 arm is capped at 5 days and recorded as non-completing if unfinished: [evidence span withheld — anchor status: not_found]
- [results] Disabling tantan masking in DIAMOND improves query coverage for read data, motivating --masking 0: "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
