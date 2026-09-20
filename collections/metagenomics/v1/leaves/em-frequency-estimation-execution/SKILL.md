---
name: em-frequency-estimation-execution
description: Use when error-corrected reads and reconstructed haplotypes from ShoRAH
  windows (e.g., HIV-1 pol pyrosequencing data) must be quantified into per-haplotype
  population frequencies via the EM-based freqEst step of the shorah.py workflow.
when_to_use_negative:
- The input is raw uncorrected reads with no prior error-correction/local haplotype
  reconstruction step — freqEst expects corrected reads from diri_sampler
- Only local analysis or SNV calling is needed (shorah shotgun/amplicon or shorah
  snv suffices) and global haplotype frequencies are not the target
- Running ShoRAH2 with the expectation of global haplotype reconstruction — global
  mode is disabled there and requires ShoRAH1 v1.1.3 or V-pipe global options
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: ShoRAH
  role: Computational method whose freqEst EM algorithm performs haplotype frequency
    estimation in the mixed sample; implemented in C++, Python, and Perl
  repo: https://github.com/cbg-ethz/shorah
- name: shorah.py
  role: Python wrapper that orchestrates the four major ShoRAH steps, including frequency
    estimation, from a single call
  repo: https://github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:ae43448dd04372076fee11e3ea531203c3834d8ed8dfb7e268b715350511af6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/shorah
  tools:
  - ShoRAH
  - shorah.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# EM Frequency Estimation Execution

## Summary

Executes the ShoRAH frequency-estimation stage (freqEst), an EM algorithm that takes error-corrected reads and reconstructed global haplotypes and estimates the prevalence of each haplotype in the mixed sample, as applied to deep sequencing data such as HIV-1 pol reads.

## When to use

Use when error-corrected reads and reconstructed haplotypes from ShoRAH windows (e.g., HIV-1 pol pyrosequencing data) must be quantified into per-haplotype population frequencies via the EM-based freqEst step of the shorah.py workflow.

## When NOT to use

- The input is raw uncorrected reads with no prior error-correction/local haplotype reconstruction step — freqEst expects corrected reads from diri_sampler
- Only local analysis or SNV calling is needed (shorah shotgun/amplicon or shorah snv suffices) and global haplotype frequencies are not the target
- Running ShoRAH2 with the expectation of global haplotype reconstruction — global mode is disabled there and requires ShoRAH1 v1.1.3 or V-pipe global options

## Inputs

- Error-corrected reads output by the ShoRAH error-correction (diri_sampler) step
- Reconstructed global haplotypes from the global haplotype reconstruction step
- Sorted BAM/CRAM alignment of next-generation sequencing reads (e.g., test_aln.cram)
- ShoRAH configuration with bamfile and snvsfile entries

## Outputs

- Estimated frequencies for each global haplotype in the mixed sample
- Haplotype prevalence estimates reflecting the number of reads per cluster
- Runtime/hardware measurements for the freqEst step (on the order of two days for 40,000 pyrosequencing reads)

## How to apply

Run frequency estimation as step 4 of the full ShoRAH pipeline (alignment; error correction/local haplotype reconstruction via diri_sampler on overlapping MSA windows; global haplotype reconstruction; frequency estimation), invoked through the shorah.py wrapper. Feed the corrected reads from the error-correction step into freqEst, running the EM algorithm with a convergence threshold of 10^-6 on the log-likelihood change and a maximum of 5000 iterations. The number of reads associated with each haplotype cluster estimates the haplotype's prevalence in the population. Budget substantial compute time: on the paper's benchmark of 40,000 pyrosequencing reads (~300 bp, HIV-1 pol-type data), error correction took less than 12 hours on 6 CPUs at 2.8 GHz and freqEst took about two days, so the full workflow must be planned around multi-day runtimes. Use ShoRAH v1.1.3 if global haplotype reconstruction is required, since ShoRAH2 has global mode disabled.

## Related tools

- **ShoRAH** (Computational method whose freqEst EM algorithm performs haplotype frequency estimation in the mixed sample; implemented in C++, Python, and Perl) — https://github.com/cbg-ethz/shorah
- **shorah.py** (Python wrapper that orchestrates the four major ShoRAH steps, including frequency estimation, from a single call) — https://github.com/cbg-ethz/shorah

## Examples

```
python shorah.py -b test_aln.cram
```

## Evaluation signals

- freqEst EM run terminates with log-likelihood change below the 10^-6 convergence threshold before hitting the 5000-iteration cap
- Runtime for freqEst is on the order of two days for a large sample (e.g., 40,000 pyrosequencing reads), and error correction completes in less than 12 hours on 6 CPUs at 2.8 GHz
- Estimated haplotype frequencies are consistent with the number of reads associated with each cluster (cluster read counts reflect haplotype prevalence)
- Low-frequency haplotypes (down to ~0.1% at 5000x coverage, per the paper's sensitivity benchmark) appear in the frequency output without being swamped by sequencing-error noise
- The upstream error-correction stage reduced the error rate about five-fold (0.25% to 0.05%), confirming inputs to freqEst are properly corrected

## Limitations

- ShoRAH2 has global haplotype reconstruction disabled; the last version of ShoRAH1 with global mode is v1.1.3, so frequency estimation on global haplotypes requires that version or V-pipe's global haplotype options
- Computational cost is high: error correction took less than 12 hours on 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads and freqEst about two days, so small machines or tight deadlines are a poor fit
- The software must be compiled and run from the command line and has only been tested on Linux and Mac OS X platforms; dependencies such as SAMtools/pysam must be obtained separately in older setups

## Evidence

- [methods] These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation: "These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation"
- [intro] The number of reads associated with the cluster estimates the prevalence of the haplotype in the population: "The number of reads associated with the cluster estimates the prevalence of the haplotype in the population"
- [methods] the error correction step was completed in less than 12 hours using 6 CPUs at 2.8 GHz for a large sample consisting of 40,000 pyrosequencing reads: "the error correction step was completed in less than 12 hours using 6 CPUs at 2.8 GHz for a large sample consisting of 40,000 pyrosequencing reads"
- [intro] Longer haplotypes sequences can be reconstructed from adjacent, overlapping reads, and the frequencies of these global haplotypes can then be estimated: "Longer haplotypes sequences can be reconstructed from adjacent, overlapping reads, and the frequencies of these global haplotypes can then be estimated"
- [readme] Currently in ShoRAH2 global haplotype reconstruction is disabled. The last version of ShoRAH1 with global mode is v1.1.3: "Currently in ShoRAH2 global haplotype reconstruction is disabled. The last version of ShoRAH1 with global mode is v1.1.3"
