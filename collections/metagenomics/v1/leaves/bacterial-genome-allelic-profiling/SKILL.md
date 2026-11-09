---
name: bacterial-genome-allelic-profiling
description: Use when you have draft/complete genome FASTA files (e.g. S. agalactiae
  assemblies) and a cgMLST/wgMLST schema seed, and need to determine allelic profiles
  or measure how AlleleCall runtime scales with CPU count.
when_to_use_negative:
- Input is raw sequencing reads rather than FASTA draft/complete assemblies — chewBBACA's
  AlleleCall engine uses FASTA files with draft assemblies or complete genomes as
  input
- The goal is to build or validate a schema rather than call alleles on strains —
  schema construction is a separate CreateSchema/LoadSchema workflow
- The schema comes from an external platform (EnteroBase, PubMLST, Ridom) without
  prior PrepExternalSchema adaptation
tools:
- name: chewBBACA
  role: Provides the AlleleCall module that performs BLAST-based (BSR) allele calling
    and reports runtime; also supplies ExtractCgMLST and RemoveGenes for downstream
    profile handling
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: Alignment engine underlying the BSR-based allele-calling algorithm in AlleleCall
- name: Python 3.4
  role: Runtime environment in which chewBBACA executes, on a laptop or HPC cluster
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:686e660b7c7c911dc6fbbc5357b2b5b87824d303dee8ad05e65ce4581555f293
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - Python 3.4
  - chewBBACA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bacterial-genome-allelic-profiling

## Summary

Performs BLAST-based gene-by-gene allele calling with chewBBACA's AlleleCall module on complete or draft bacterial genome FASTA files, returning an allelic profile matrix, and can benchmark how per-sample runtime scales with CPU count (near-linear up to 32 CPUs, then disk/I/O-bound).

## When to use

Use when you have draft/complete genome FASTA files (e.g. S. agalactiae assemblies) and a cgMLST/wgMLST schema seed, and need to determine allelic profiles or measure how AlleleCall runtime scales with CPU count.

## When NOT to use

- Input is raw sequencing reads rather than FASTA draft/complete assemblies — chewBBACA's AlleleCall engine uses FASTA files with draft assemblies or complete genomes as input
- The goal is to build or validate a schema rather than call alleles on strains — schema construction is a separate CreateSchema/LoadSchema workflow
- The schema comes from an external platform (EnteroBase, PubMLST, Ridom) without prior PrepExternalSchema adaptation

## Inputs

- Draft or complete genome FASTA assemblies (e.g. 100 S. agalactiae draft genomes from the FigShare zip archive)
- cgMLST/wgMLST schema directory with schema_seed loci (produced by CreateSchema)
- Optional list of core loci (cgMLSTschema.txt from ExtractCgMLST) passed via --gl
- Set of CPU counts to benchmark (e.g. 1 to HPC maximum)

## Outputs

- Allelic profile matrix (results_alleles.tsv) in the results_<datestamp> output directory
- RepeatedLoci.txt listing paralogous loci
- FASTA files with the full allelic diversity per locus; novel alleles added to the schema
- Runtime-per-sample scaling table (allelecall_runtime_scaling.tsv: runtime vs CPU count)

## How to apply

Load the schema directory (e.g. the schema_seed produced by CreateSchema for S. agalactiae complete genomes) as the reference loci and run chewBBACA.py AlleleCall on the FASTA assemblies, using the default BSR threshold of 0.6 and passing the --cpu flag with each tested CPU count. To benchmark scaling, invoke AlleleCall once per CPU configuration on the same 100 assemblies, recording the wall-clock time of each run, then parse the outputs (results_alleles.tsv and related datestamped files) to compute per-sample runtime. Benchmarking on 100 S. agalactiae assemblies showed near-linear per-sample runtime decrease up to 32 CPUs, after which disk/I/O becomes the bottleneck, with peak performance of approximately 2.5 s per sample using 32 CPUs on an HPC cluster; on a laptop, expect seconds to minutes per strain (the tutorial's 32-genome wgMLST call averaged 32 s per genome on 6 cores). AlleleCall identifies novel alleles, assigns identifiers, and adds them to the schema (tutorial example: 14,720 novel alleles grew the schema from 3,128 to 17,848), and flags paralogous loci in RepeatedLoci.txt. Validation: accept the runtime-per-sample scaling table (allelecall_runtime_scaling.tsv) if it reproduces the near-linear speedup up to 32 CPUs followed by an I/O-bound plateau, with each per-strain runtime in the seconds-to-minutes range.

## Related tools

- **chewBBACA** (Provides the AlleleCall module that performs BLAST-based (BSR) allele calling and reports runtime; also supplies ExtractCgMLST and RemoveGenes for downstream profile handling) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (Alignment engine underlying the BSR-based allele-calling algorithm in AlleleCall)
- **Python 3.4** (Runtime environment in which chewBBACA executes, on a laptop or HPC cluster)

## Examples

```
chewBBACA.py AlleleCall -i complete_genomes/ -g tutorial_schema/schema_seed -o results32_wgMLST --cpu 6
```

## Evaluation signals

- The runtime-vs-CPU table shows near-linear per-sample runtime decrease up to 32 CPUs followed by an I/O-bound plateau
- Peak per-sample runtime is approximately 2.5 s using 32 CPUs on the HPC; all per-strain runtimes fall in the seconds-to-minutes range
- AlleleCall output directory contains results_alleles.tsv (allelic profile matrix) and RepeatedLoci.txt; default BSR threshold of 0.6 was used
- Novel alleles found during calling are assigned identifiers and appended to the schema (tutorial: 3,128 -> 17,848 alleles over 32 genomes)
- Paralogous loci (tutorial: 20 loci) can be removed with RemoveGenes to yield a paralog-filtered profile matrix (results_alleles_NoParalogs.tsv)

## Limitations

- Beyond 32 CPUs, disk/I/O becomes the bottleneck, so additional CPUs yield little speedup on large allele-calling runs
- RepeatedLoci.txt paralogous loci should be removed from profiles because allele assignment is uncertain for paralogs
- PrepExternalSchema modifies the original source FASTA, so users should back up data before adapting external schemas
- Using a computer with less powerful specifications than the tutorial's 6-core DELL XPS13 can greatly increase analysis duration

## Evidence

- [abstract] The software performs allele calling in a matter of seconds to minutes per strain on a laptop but is easily scalable: "The software performs allele calling in a matter of seconds to minutes per strain on a laptop but is easily scalable"
- [discussion] AlleleCall output description: "returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus"
- [readme] Default BSR threshold and runtime: [evidence span withheld — anchor status: external_doc]
- [readme] AlleleCall command from tutorial: [evidence span withheld — anchor status: external_doc]
