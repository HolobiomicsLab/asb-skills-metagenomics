---
name: short-read-coverage-subsampling
description: Use when you must benchmark a retrieval/assembly pipeline across sequencing
  depths and only full-coverage paired-end Illumina reads (e.g. D. gyrociliatus PRJEB37657)
  are available — subsample to simulate lower-coverage genome-skimming arms.
when_to_use_negative:
- Your analysis question does not involve sequencing-depth response (e.g. full-coverage
  assemblies are already available and only a single-depth retrieval run is needed).
- Input reads are not paired-end, or the FASTQ naming structure does not allow selecting
  corresponding read pairs — random independent subsampling would break read-pair
  correspondence and bias assemblies.
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Subsample.jl
  role: randomly selects corresponding read pairs from the paired-end FASTQ data to
    generate coverage arms
- name: SPAdes
  role: produces the short-read-only de novo assembly for each coverage arm (K-mer
    size 33) used as assembly-based input
- name: QUAST
  role: assesses quality of each per-coverage-arm assembly to verify depth-dependent
    assembly improvement
- name: Patchwork
  role: retrieves near-USCO markers from the subsampled reads and assemblies at each
    depth for benchmarking
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:d80a6a596829d16c437aa8cb021c339ff4c0f5ce5e9a1c8ddb246ab833666638
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - ALiBaSeq
  - BLAST
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# short-read-coverage-subsampling

## Summary

Generates a sequencing-depth gradient by randomly selecting corresponding read pairs from paired-end short-read FASTQ data (Subsample.jl), producing coverage arms (e.g. 1×, 3×, 5×, 10×, 20×, 40×) used to benchmark marker-retrieval accuracy and runtime across depths.

## When to use

Use when you must benchmark a retrieval/assembly pipeline across sequencing depths and only full-coverage paired-end Illumina reads (e.g. D. gyrociliatus PRJEB37657) are available — subsample to simulate lower-coverage genome-skimming arms.

## When NOT to use

- Your analysis question does not involve sequencing-depth response (e.g. full-coverage assemblies are already available and only a single-depth retrieval run is needed).
- Input reads are not paired-end, or the FASTQ naming structure does not allow selecting corresponding read pairs — random independent subsampling would break read-pair correspondence and bias assemblies.

## Inputs

- trimmed paired-end Illumina short-read FASTQ files (D. gyrociliatus, ENA accession PRJEB37657)
- target coverage levels (1×, 3×, 5×, 10×, 20×, 40×) relative to genome size (73.8 Mb)
- Subsample.jl for read-pair selection
- reference for downstream scoring: 778 recoverable D. gyrociliatus USCOs

## Outputs

- one FASTQ read-pair set per coverage arm (1×–40×), with paired-end synchronization preserved
- per-arm SPAdes (K=33) short-read-only de novo assembly (contigs) with QUAST quality report
- per-arm retrieval results and Slurm elapsed times for each program
- benchmark_comparison.tsv with accuracy (pidentical,cov) and runtime per program per coverage

## How to apply

Subsample the trimmed paired-end Illumina reads with Subsample.jl by randomly selecting corresponding read pairs, so read1/read2 pairing is preserved and the fraction of pairs maps to the target depth (here 1×, 3×, 5×, 10×, 20×, and 40× of the ~73.8-Mb D. gyrociliatus genome). For each coverage arm, produce a short-read-only de novo assembly with SPAdes (K-mer size 33) and assess assembly quality with QUAST, giving both read-based and assembly-based inputs per depth. Run the comparison programs (Patchwork v.0.5.1 on reads and assemblies, ALiBaSeq v.1.2 on assemblies, aTRAM 2 v.2.4.3 on reads) on each arm under identical settings, recording per-program Slurm-reported elapsed times. Compile accuracy (joint metric pidentical,cov against the 778 recoverable D. gyrociliatus USCO set) and runtime values into benchmark_comparison.tsv to reproduce the depth-response pattern.

## Related tools

- **Subsample.jl** (randomly selects corresponding read pairs from the paired-end FASTQ data to generate coverage arms)
- **SPAdes** (produces the short-read-only de novo assembly for each coverage arm (K-mer size 33) used as assembly-based input)
- **QUAST** (assesses quality of each per-coverage-arm assembly to verify depth-dependent assembly improvement)
- **Patchwork** (retrieves near-USCO markers from the subsampled reads and assemblies at each depth for benchmarking) — https://github.com/fethalen/Patchwork

## Evaluation signals

- Each coverage arm contains the expected linear fraction of read pairs (arm sizes follow 1:3:5:10:20:40 relative to the 40× arm) with read1/read2 pairs still matched.
- QUAST metrics on the per-arm SPAdes (K=33) assemblies should improve monotonically with coverage.
- The retrieved-marker scores (pidentical,cov against the 778 D. gyrociliatus USCO set) should show the reported depth trend: Patchwork with assemblies reaching up to 62% total percent identity at ≥10×, ALiBaSeq ~7% lower, aTRAM 2 max ~22%.
- The 40× aTRAM 2 run should be recordable as non-completing within the 5-day limit.
- benchmark_comparison.tsv contains one accuracy and one runtime entry per program per coverage arm, with times matching Slurm-reported elapsed times.

## Limitations

- At 1× coverage, assembly-based retrieval is inadequate — read-based data performed better, so the lowest arm may favor read-direct tools.
- aTRAM 2 failed to complete the 40× data set within 5 days, so runtime capping/timeouts must be pre-defined to keep the benchmark bounded.
- Coverage arms are simulated by read-pair subsampling, not independent sequencing runs, so per-arm library biases of real low-coverage experiments are not modeled.

## Evidence

- [results] subsampled short reads to simulate coverages: "we also subsampled the aforementioned D. gyrociliatus short reads in order to simulate various sequencing coverages"
- [results] read data source and accession: "Illumina short-read nucleotide sequences from the marine annelid Dimorphilus gyrociliatus (accession PRJEB37657 in the European Nucleotide Archive)"
- [results] 1× coverage favors read-based retrieval: "Only with a coverage of 1×, D. gyrociliatus read-based data seem to be better suited for marker retrieval"
- [results] per-arm assembly step: "We assembled these sequence reads using SPAdes and subsequently used Patchwork to search for near-USCOs"
- [full_text] subsampling procedure and coverage arms: [evidence span withheld — anchor status: not_found]
