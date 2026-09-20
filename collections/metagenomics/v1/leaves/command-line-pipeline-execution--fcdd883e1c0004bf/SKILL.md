---
name: command-line-pipeline-execution
description: Use when a task requires running the STEMSim simulator end-to-end from
  the command line, or reproducing its reported throughput benchmark (~1 million reads
  in ~3 minutes on a 2.40 GHz server CPU).
when_to_use_negative:
- The goal is de novo community abundance simulation — use CAMISIM itself; STEMSim
  consumes CAMISIM output rather than generating communities.
- The input is real metagenomic data from species with low depth or poorly characterized
  genomes — the README warns STEMSim 'only works well on species with high depth and
  well-characterized genomes'.
- The task requires within-strain mutation simulation but not pipeline execution/benchmarking,
  e.g. interpreting SnpEff-annotated mutation output.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: STEMSim
  role: Main simulator invoked as stemsim.py from the command line in -m reads or
    -m camisim mode
  repo: github.com/BoyanZhou/STEMSim
- name: Bowtie2
  role: Internal alignment of longitudinal reads against the reference genome during
    mutation simulation
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: File handling (tested with 1.9) within the STEMSim pipeline
  repo: http://www.htslib.org/doc/
- name: CAMISIM
  role: Optional upstream generator of the input directory for 'camisim' mode (tested
    with 1.3)
  repo: https://github.com/CAMI-challenge/CAMISIM
- name: ART
  role: Optional upstream read simulator producing FASTQ input for 'reads' mode (tested
    with art/2016-06-05)
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm
- name: SnpEff
  role: Optional annotation of the VCF output of true mutations (tested with 5.1)
  repo: http://pcingola.github.io/SnpEff/
- name: pysam
  role: Python (3.6.5) package used by STEMSim for read/file handling (tested with
    0.16.0.1)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:22db19c45300c7c542bd0f66db9337bcbcf96aa9b69eb1f0f86d0b0bc3a0cce2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - Bowtie2
  - CAMISIM
  - Python
  - STEMSim
  - SnpEff
  - numpy
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# command-line-pipeline-execution

## Summary

Execute and time the STEMSim command-line pipeline (stemsim.py in 'reads' or 'camisim' mode) to generate simulated longitudinal metagenomic reads carrying within-strain short-term evolutionary mutations, and verify wall-clock runtime on benchmark-scale inputs (1 million reads).

## When to use

Use when a task requires running the STEMSim simulator end-to-end from the command line, or reproducing its reported throughput benchmark (~1 million reads in ~3 minutes on a 2.40 GHz server CPU).

## When NOT to use

- The goal is de novo community abundance simulation — use CAMISIM itself; STEMSim consumes CAMISIM output rather than generating communities.
- The input is real metagenomic data from species with low depth or poorly characterized genomes — the README warns STEMSim 'only works well on species with high depth and well-characterized genomes'.
- The task requires within-strain mutation simulation but not pipeline execution/benchmarking, e.g. interpreting SnpEff-annotated mutation output.

## Inputs

- Simulated raw longitudinal reads (FASTQ) from a reference genome, e.g. from ART, for 'reads' mode
- CAMISIM output directory plus genome_to_id.tsv for 'camisim' mode
- Reference genome FASTA (.fas) and its genome ID
- config.txt parameter template (kept at tutorial defaults for benchmarking)
- Optional non-mutated region file (.gtf, .gff, or .bed)

## Outputs

- Modified reads carrying within-strain evolutionary mutations in the output directory (-o)
- Mutation information / VCF of true mutations (optionally annotated with SnpEff)
- Run log file (e.g. stemsim_test1_log)
- Measured elapsed wall-clock time for the benchmark (e.g. runtime_report.txt)

## How to apply

Install the required stack (python 3.6.5, numpy 1.19.5, pysam 0.16.0.1, Bowtie2, samtools 1.9, optionally CAMISIM 1.3, ART 2016-06-05, SnpEff 5.1) and download the STEMSim files into one directory. Choose the working mode with -m: 'reads' when starting from raw simulated FASTQ reads of a single reference genome (e.g. from ART), or 'camisim' when starting from a CAMISIM output directory. Supply the config file template via -f, the subject ID via -s, longitudinal read files separated by ':' (paired files within a sample by ','), the reference .fas genome and --genome_id, an output directory via -o, and a log path via -l (e.g. stemsim_test1_log). Wrap the invocation in a wall-clock timer on a comparable server CPU, keeping config parameters at tutorial defaults so input size is the only variable, then confirm the run completes and writes modified reads, mutation information, and the log.

## Related tools

- **STEMSim** (Main simulator invoked as stemsim.py from the command line in -m reads or -m camisim mode) — github.com/BoyanZhou/STEMSim
- **Bowtie2** (Internal alignment of longitudinal reads against the reference genome during mutation simulation) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (File handling (tested with 1.9) within the STEMSim pipeline) — http://www.htslib.org/doc/
- **CAMISIM** (Optional upstream generator of the input directory for 'camisim' mode (tested with 1.3)) — https://github.com/CAMI-challenge/CAMISIM
- **ART** (Optional upstream read simulator producing FASTQ input for 'reads' mode (tested with art/2016-06-05)) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm
- **SnpEff** (Optional annotation of the VCF output of true mutations (tested with 5.1)) — http://pcingola.github.io/SnpEff/
- **pysam** (Python (3.6.5) package used by STEMSim for read/file handling (tested with 0.16.0.1))

## Examples

```
python /path_to_stemsim_directory/stemsim.py -m reads -f PATH/config.txt -s test_subject1 --input_reads PATH/sample0_R1.fq,PATH/sample0_R2.fq:PATH/sample1.fq:PATH/sample2.fq -o PATH_to_output_directory_of_STEMSim/ --reference_genome PATH/species0_strain0.fas --genome_id species0_strain0 -l Path/stemsim_test1_log
```

## Evaluation signals

- runtime_report.txt exists and states a single numeric elapsed wall-clock time (~3 minutes for 1 million reads on a 2.40 GHz server CPU).
- The stemsim_test1_log file confirms the run processed the fixed one-million-read input without errors.
- Output directory contains modified reads with within-strain evolutionary mutations plus the relevant mutation information.
- If SnpEff was supplied via -p, the VCF of true mutations is annotated; nonmutated regions listed in a .gtf/.gff/.bed file contain no generated mutations.
- All required software versions fall within the tested set: python 3.6.5, numpy 1.19.5, pysam 0.16.0.1, samtools 1.9.

## Limitations

- STEMSim is designed to work on longitudinal or concurrent simulated raw sequencing reads; although it can run on real metagenomic data, it only works well on species with high depth and well-characterized genomes.
- The reported runtime (~1 million reads in ~3 minutes) was measured on a server with a 2.40 GHz CPU; different hardware will change elapsed time.
- No changelog is available for the BoyanZhou/STEMSim repository, complicating version pinning.
- Python 3.6.5 is an old runtime; the code was written and tested specifically on that version with pysam 0.16.0.1.

## Evidence

- [other] STEMSIM is fast enough for benchmark-scale use, processing one million reads in approximately three minutes when run on a server with a 2.40 GHz CPU.: [evidence span withheld — anchor status: not_found]
- [readme] This code was written and tested on python 3.6.5, and requires the following packages: "This code was written and tested on python 3.6.5, and requires the following packages"
- [methods] stemsim takes the simulated raw reads from other software (such as ART) as input, and generate longitudinal mutations: "stemsim takes the simulated raw reads from other software (such as ART) as input, and generate longitudinal mutations on the simulated raw sequencing data"
- [abstract] The output is the modified reads with within-strain evolutionary mutations and the relevant information of these mutations.: "The output is the modified reads with within-strain evolutionary mutations and the relevant information of these mutations."
- [readme] it only works well on species with high depth and well-characterized genomes.: "Although it can also works on real metagenomic data, it only works well on species with high depth and well-characterized genomes."
