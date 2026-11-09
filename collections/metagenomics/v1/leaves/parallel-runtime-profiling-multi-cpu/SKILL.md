---
name: parallel-runtime-profiling-multi-cpu
description: Use when you need to reproduce or compare reported compute times for
  ShoRAH's error-correction or frequency-estimation steps on deep-sequencing (e.g.
  40,000-read pyrosequencing) samples, or to document hardware requirements of a viral
  diversity workflow.
when_to_use_negative:
- Your goal is accuracy of haplotype reconstruction or SNV calling rather than computational-efficiency
  measurement of the timed steps
- Your sample is far outside the benchmark regime (e.g. Illumina tumour-dissection
  data or a few hundred reads), so the <12 h / two-days targets are not comparable
- You intend to run ShoRAH2's global haplotype reconstruction, which is disabled in
  current versions and would alter the timed pipeline
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: ShoRAH
  role: Suite whose diri_sampler error-correction and freqEst frequency-estimation
    steps are the timed stages of the benchmark
  repo: http://www.cbg.ethz.ch/software/shorah
- name: shorah.py
  role: Python wrapper that chains the four major ShoRAH steps; its overhead is checked
    to remain negligible in the benchmark
  repo: github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:6a87a2f94570a79810ade0bc0cb8520f1cb6781b055f5938253ae1e1c163f117
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ShoRAH
  - shorah.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# parallel-runtime-profiling-multi-cpu

## Summary

Benchmarks the wall-clock runtime of ShoRAH's compute-heavy stages — diri_sampler error correction and freqEst EM frequency estimation — on large pyrosequencing samples, recording the CPU count, clock speed, and read volume under which published timings (<12 h error correction; ~2 days frequency estimation) were obtained.

## When to use

Use when you need to reproduce or compare reported compute times for ShoRAH's error-correction or frequency-estimation steps on deep-sequencing (e.g. 40,000-read pyrosequencing) samples, or to document hardware requirements of a viral diversity workflow.

## When NOT to use

- Your goal is accuracy of haplotype reconstruction or SNV calling rather than computational-efficiency measurement of the timed steps
- Your sample is far outside the benchmark regime (e.g. Illumina tumour-dissection data or a few hundred reads), so the <12 h / two-days targets are not comparable
- You intend to run ShoRAH2's global haplotype reconstruction, which is disabled in current versions and would alter the timed pipeline

## Inputs

- Large pyrosequencing sample of 40,000 reads of average length ~300 bp (HIV-1 pol gene type data)
- Overlapping multiple sequence alignment (MSA) windows for diri_sampler
- Corrected reads for freqEst frequency estimation

## Outputs

- runtime_benchmark.csv with measured error-correction and freqEst timings
- Hardware configuration record (6 CPUs at 2.8 GHz, read volume 40,000)

## How to apply

Load the benchmark-sized sample (40,000 pyrosequencing reads of ~300 bp, HIV-1 pol gene type data) and confirm read count and length distribution match the paper's conditions. Run the error-correction/local haplotype reconstruction step with diri_sampler on overlapping MSA windows, parallelized across 6 CPUs at 2.8 GHz, and record wall-clock time against the reported target of less than 12 hours. Then feed corrected reads into freqEst, running the EM algorithm with convergence threshold 10^-6 on log-likelihood change and a maximum of 5000 iterations, and record its runtime (reported on the order of two days). Verify that mm.py and the shorah.py wrapper overhead remain negligible relative to these two timed steps, then write measured timings and hardware configuration to runtime_benchmark.csv.

## Related tools

- **ShoRAH** (Suite whose diri_sampler error-correction and freqEst frequency-estimation steps are the timed stages of the benchmark) — http://www.cbg.ethz.ch/software/shorah
- **shorah.py** (Python wrapper that chains the four major ShoRAH steps; its overhead is checked to remain negligible in the benchmark) — github.com/cbg-ethz/shorah

## Examples

```
shorah shotgun -b test_aln.cram
```

## Evaluation signals

- Measured error-correction wall-clock time is below 12 hours when run on 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads
- freqEst runtime is on the order of two days, matching the paper's computational-efficiency section
- EM convergence recorded at log-likelihood change threshold 10^-6 with a maximum of 5000 iterations
- Wrapper overhead from mm.py and shorah.py is verified negligible relative to the two timed steps
- runtime_benchmark.csv contains both measured timings and the hardware configuration used

## Limitations

- The <12 h benchmark is tied to specific hardware (6 CPUs at 2.8 GHz) and pyrosequencing read volume; timings will not transfer to different CPUs or Illumina data
- ShoRAH2 currently has global haplotype reconstruction disabled; the last version of ShoRAH1 with global mode is v1.1.3, so reproducibility may depend on the version installed
- ShoRAH must be compiled and run from the command line and has only been tested on Linux and Mac OS X platforms

## Evidence

- [methods] Error correction completed in less than 12 hours using 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads: "the error correction step was completed in less than 12 hours using 6 CPUs at 2.8 GHz for a large sample consisting of 40,000 pyrosequencing reads"
- [methods] Four major ShoRAH steps invoked via the shorah.py wrapper: "Full analysis with ShoRAH includes four major steps that are performed by calling the wrapper shorah.py"
- [readme] diri_sampler performs error correction via Dirichlet process mixture Gibbs sampling: "`diri_sampler` | Gibbs sampling for error correction via Dirichlet process mixture"
- [readme] Local analysis cut into windows and diri_sampler invoked: "They work by cutting window from the multiple sequence alignment, invoking `diri_sampler` on the windows and calling `shorah snv`"
