---
name: allelecall-scaling-analysis
description: Use when you need to quantify the computational scaling of chewBBACA
  AlleleCall (runtime vs CPU count, per-sample seconds-to-minutes behaviour) for a
  fixed set of draft genome FASTA files on a laptop or HPC cluster.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled draft/complete genome FASTA
  files (AlleleCall consumes FASTA assemblies; read-based tools like MentaLiST are
  a different approach)
- You want to benchmark schema creation (CreateSchema) or other chewBBACA modules
  rather than the AlleleCall runtime specifically
- The dataset has fewer than a few dozen genomes, so per-sample runtimes are already
  in the seconds-to-minutes range and CPU scaling cannot be meaningfully resolved
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: chewBBACA
  role: Provides the AlleleCall module whose BLAST Score Ratio-based allele calling
    is benchmarked across CPU counts
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: Underlying alignment engine used by AlleleCall to determine allelic profiles
    of bacterial strains
- name: Python 3.4
  role: Runtime environment for chewBBACA; scripting (Python) is used to parse outputs
    and compute per-sample runtimes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:d8f750ed26ab3af813e6090a2bbf4805cc06bfd7e60d5620dc4fa421f4118254
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - Python 3.4
  - chewBBACA
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# allelecall-scaling-analysis

## Summary

Benchmark how chewBBACA's BLAST-based AlleleCall module scales with CPU count by timing allele calling of 100 S. agalactiae draft assemblies across multiple CPU configurations, identifying the point at which disk/I/O replaces CPU parallelism as the bottleneck.

## When to use

Use when you need to quantify the computational scaling of chewBBACA AlleleCall (runtime vs CPU count, per-sample seconds-to-minutes behaviour) for a fixed set of draft genome FASTA files on a laptop or HPC cluster.

## When NOT to use

- Input is raw sequencing reads rather than assembled draft/complete genome FASTA files (AlleleCall consumes FASTA assemblies; read-based tools like MentaLiST are a different approach)
- You want to benchmark schema creation (CreateSchema) or other chewBBACA modules rather than the AlleleCall runtime specifically
- The dataset has fewer than a few dozen genomes, so per-sample runtimes are already in the seconds-to-minutes range and CPU scaling cannot be meaningfully resolved

## Inputs

- 100 S. agalactiae draft genome FASTA assembly files (FigShare zip)
- chewBBACA cgMLST/wgMLST schema directory with schema_seed loci (from CreateSchema)
- Prodigal training file for Streptococcus agalactiae (e.g. Streptococcus_agalactiae.trn)

## Outputs

- allelecall_runtime_scaling.tsv (per-sample runtime vs CPU count table)
- AlleleCall output directories with results_alleles.tsv allelic profile matrices per CPU configuration

## How to apply

Download the zip of S. agalactiae assemblies from FigShare (https://figshare.com/s/9cbe1d422805db54cd52) and extract the 100 draft genome FASTA files, then load the cgMLST schema directory produced by CreateSchema as the reference loci. Invoke `chewBBACA.py AlleleCall` once per tested CPU count (varying `--cpu`), recording wall-clock time for each run; the default BSR threshold of 0.6 applies to allele classification. Parse the AlleleCall outputs to extract the allelic profile matrix (results_alleles.tsv) and compute per-sample runtime for each CPU configuration. Tabulate runtime versus CPU count and confirm the reported pattern: near-linear per-sample runtime decrease up to 32 CPUs, after which disk/I/O becomes the bottleneck (~2.5 s per sample at 32 CPUs on the HPC; seconds to minutes per strain on a laptop).

## Related tools

- **chewBBACA** (Provides the AlleleCall module whose BLAST Score Ratio-based allele calling is benchmarked across CPU counts) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (Underlying alignment engine used by AlleleCall to determine allelic profiles of bacterial strains)
- **Python 3.4** (Runtime environment for chewBBACA; scripting (Python) is used to parse outputs and compute per-sample runtimes)

## Examples

```
chewBBACA.py AlleleCall -i sagalactiae_100_assemblies/ -g tutorial_schema/schema_seed -o results_100 --cpu 32 && /usr/bin/time -v echo done
```

## Evaluation signals

- Measured per-sample runtimes show near-linear decrease up to 32 CPUs, then an I/O-bound plateau
- Each per-strain runtime falls in the seconds-to-minutes range on a laptop
- Peak performance of approximately 2.5 s per sample is observed using 32 CPUs on the HPC cluster
- AlleleCall outputs contain a parseable allelic profile matrix (results_alleles.tsv) for every CPU configuration tested
- Runtime-vs-CPU table (allelecall_runtime_scaling.tsv) is complete, with one row per tested CPU count

## Limitations

- Beyond 32 CPUs, disk/I/O becomes the bottleneck, so adding more cores yields no further speedup
- Execution times depend strongly on hardware; a less powerful machine than the DELL XPS13 (6-core i7-10710U) used in the tutorial can greatly increase analysis duration
- The published benchmark is specific to S. agalactiae assemblies and this schema; other taxa or schema sizes will shift the crossover point

## Evidence

- [abstract] The software performs allele calling in a matter of seconds to minutes per strain on a laptop but is easily scalable: "The software performs allele calling in a matter of seconds to minutes per strain on a laptop but is easily scalable"
- [abstract] The chewBBACA software uses Python 3.4 or higher and can run on a laptop or in high performance clusters: "The chewBBACA software uses Python 3.4 or higher and can run on a laptop or in high performance clusters"
- [readme] chewBBACA.py AlleleCall -i complete_genomes/ -g tutorial_schema/schema_seed -o results32_wgMLST --cpu 6: [evidence span withheld — anchor status: external_doc]
- [readme] The allele call used the default BSR threshold of 0.6 (more information on the threshold: [evidence span withheld — anchor status: external_doc]
- [readme] Using a computer with less powerful specifications can greatly increase the duration of the analyses.: [evidence span withheld — anchor status: external_doc]
