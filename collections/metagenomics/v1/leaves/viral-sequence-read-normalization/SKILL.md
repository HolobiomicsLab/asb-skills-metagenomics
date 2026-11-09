---
name: viral-sequence-read-normalization
description: Use when raw paired-end virome reads with uneven genome coverage must
  be dereplicated and digitally normalized to default parameters prior to de-novo
  and mapping assembly in a VirMAP classification run.
when_to_use_negative:
- The input reads are already dereplicated and digitally normalized — rerunning bbnorm
  would further reduce coverage needed for a quality assembly.
- The analysis goal is taxonomic classification only on pre-assembled contigs or marker
  genes, not on raw reads, since normalization is a pre-assembly step of the read-based
  VirMAP pipeline.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0080
tools:
- name: VirMAP
  role: Pipeline wrapper whose --useBbnorm flag invokes digital normalization before
    the assembly and taxonomic classification steps
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Provides the bbnorm digital-normalization implementation used by VirMAP on
    input reads
- name: khmer
  role: Cited in the VirMAP methods as the source of the normalize-by-median.py digital-normalization
    script
  repo: https://github.com/dib-lab/khmer
- name: MEGAHIT
  role: Multi-kmer de-novo assembler that consumes the normalized reads downstream
  repo: https://github.com/voutcn/megahit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:41af3806204c46e142cb53501ff64b70eceed0af425d2f4b810234d3521cad0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - MEGAHIT
  - Perl
  - VirMAP
  - khmer
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# viral-sequence-read-normalization

## Summary

Dereplication and digital normalization of raw viral metagenomic reads (e.g., SRR9875293 from the Viral Mock Community PRJNA431646) using BBMap's bbnorm, as enabled by the --useBbnorm flag of Virmap.pl, to even out coverage before multi-kmer de-novo assembly with MEGAHIT.

## When to use

Use when raw paired-end virome reads with uneven genome coverage must be dereplicated and digitally normalized to default parameters prior to de-novo and mapping assembly in a VirMAP classification run.

## When NOT to use

- The input reads are already dereplicated and digitally normalized — rerunning bbnorm would further reduce coverage needed for a quality assembly.
- The analysis goal is taxonomic classification only on pre-assembled contigs or marker genes, not on raw reads, since normalization is a pre-assembly step of the read-based VirMAP pipeline.

## Inputs

- Paired-end FASTQ reads downloaded from SRA (e.g., SRR9875293 via fasterq-dump)
- Raw metagenomic virome sequencing data with uneven viral genome coverage

## Outputs

- Dereplicated read files
- Digitally normalized FASTQ reads passed to MEGAHIT assembly
- Normalized intermediate files written to scratch (e.g., /scratch/tmp, /dev/shm)
- VirMAP taxonomic classification outputs in the run directory (e.g., VirmapRun)

## How to apply

Within a VirMAP run, enable digital normalization by passing --useBbnorm to Virmap.pl alongside --useMegahit, so that input reads are first dereplicated and then digitally normalized with default parameters before the multi-kmer de-novo assembly step (MEGAHIT or Tadpole). The rationale, as stated in the article, is that normalization addresses the challenge that 'achieving the necessary coverage for a quality assembly remains challenging' by reducing redundant high-coverage reads without discarding low-coverage viral signal. Point TMPDIR to scratch storage (e.g., /scratch/tmp) and use /dev/shm as the fasterq-dump temp directory so intermediates do not exhaust the root volume. As an alternative implementation, the khmer normalize-by-median.py script is referenced in the methods. Normalization runs with default parameters — the article does not specify target coverage or k-mer thresholds for bbnorm — so correctness is judged by downstream assembly success rather than explicit parameter tuning.

## Related tools

- **VirMAP** (Pipeline wrapper whose --useBbnorm flag invokes digital normalization before the assembly and taxonomic classification steps) — https://github.com/cmmr/VirMAP
- **BBMap** (Provides the bbnorm digital-normalization implementation used by VirMAP on input reads)
- **khmer** (Cited in the VirMAP methods as the source of the normalize-by-median.py digital-normalization script) — https://github.com/dib-lab/khmer
- **MEGAHIT** (Multi-kmer de-novo assembler that consumes the normalized reads downstream) — https://github.com/voutcn/megahit

## Examples

```
Virmap.pl --threads $(nproc) --useMegahit --useBbnorm --sampleName SRR9875293 --readF /home/$USER/VirmapTest/SRR9875293_1.fastq
```

## Evaluation signals

- The Virmap.pl run log shows --useBbnorm was accepted and executed without error before the MEGAHIT assembly stage.
- The VirmapRun output directory (e.g., /home/$USER/VirmapTest/VirmapRun) exists and contains populated VirMAP taxonomic classification results (script_runs and file_exists checks).
- The de-novo assembly completes and produces contigs, indicating normalization did not strip away coverage required for a quality assembly.
- On the Viral Mock Community benchmark (PRJNA431646), downstream classification recovers all seven intended constituents, consistent with the reported recall of 1.0.

## Limitations

- Normalization is run with default parameters; no bbnorm target-coverage or k-mer thresholds are specified in the article, limiting reproducibility tuning.
- No software versions are stated for VirMAP, BBMap, MEGAHIT, or Tadpole, so the exact normalization behavior may differ across tool releases.
- Because default normalization reduces depth of over-represented regions, very low-coverage or highly divergent viral signal may be sensitive to over-normalization; the article notes that achieving the necessary coverage for a quality assembly remains challenging.

## Evidence

- [discussion] Read dereplication and digital normalization: "input reads are dereplicated and digitally normalized18 using default parameters"
- [methods] Normalization and MEGAHIT flags in the VirMAP invocation: "--useMegahit --useBbnorm --sampleName SRR9875293"
- [discussion] Assembler consuming normalized reads: "a multi-kmer de-novo assembly using either MEGAHIT21,22 or Tadpole19, or both"
- [methods] khmer as cited normalization implementation: "normalize-by-median.py (https://github.com/dib-lab/khmer)"
- [readme] Concrete Virmap.pl entry point from the repository paper abstract: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
- [intro] Assembly coverage challenge motivating normalization: "achieving the necessary coverage for a quality assembly remains challenging"
