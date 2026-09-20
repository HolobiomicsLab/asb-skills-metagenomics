---
name: parallel-command-line-pipeline-execution
description: Use when a pipeline step must be repeated independently over many input
  files (e.g. 4,472 vaginal metagenomic assemblies or ~26 million predicted genes)
  and each invocation is a standalone CLI tool that can be embarrassingly parallelized.
when_to_use_negative:
- Steps with inter-item dependencies or shared-state reduction (e.g. dereplication
  or clustering of all ~26 million proteins into VMGC-90) that need a dedicated tool
  like MMseqs2 rather than naive per-item parallelism
- A single large sample where per-item overhead dominates and a multi-threaded single
  invocation (e.g. one bwa-mem2 mem -t $thread run) is simpler
edam_operation: http://edamontology.org/operation_2985
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: GNU parallel 20201122
  role: Job fan-out engine executing Mash, bwa-mem2, MetaBAT2, Kraken2/Bracken and
    Prodigal commands per sample with -j concurrency control
- name: Mash 2.3
  role: Computes sketch distances whose top-20 neighbor lists parallel consumes to
    select samples for multiple-coverage depth calculation
- name: bwa-mem2 2.2.1
  role: Read mapping step parallelized per (neighbor-sample, assembly) pair to produce
    contig sequencing-depth BAMs
- name: combine.pl
  role: Depth-integration script invoked under parallel before multiple-coverage binning
  repo: https://github.com/WatsonLab/single_and_multiple_binning/blob/main/scripts/combine.pl
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:c26ad87cc87739abe322c8e38ab6a83a91b42a1d79a33d1795d4bfce0aa4a600
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNU parallel
  - GNU parallel 20201122
  - Mash 2.3
  - Prodigal
  - bwa-mem2 2.2.1
  - combine.pl
  edam_operation: http://edamontology.org/operation_2985
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# parallel-command-line-pipeline-execution

## Summary

Use GNU parallel to fan out per-sample command-line steps (Mash distance, bwa-mem2 mapping, MetaBAT2 binning, Kraken2 profiling, Prodigal gene calling) across thousands of genomes/samples with controlled job concurrency (-j), as done throughout the VMGC pipeline.

## When to use

Use when a pipeline step must be repeated independently over many input files (e.g. 4,472 vaginal metagenomic assemblies or ~26 million predicted genes) and each invocation is a standalone CLI tool that can be embarrassingly parallelized.

## When NOT to use

- Steps with inter-item dependencies or shared-state reduction (e.g. dereplication or clustering of all ~26 million proteins into VMGC-90) that need a dedicated tool like MMseqs2 rather than naive per-item parallelism
- A single large sample where per-item overhead dominates and a multi-threaded single invocation (e.g. one bwa-mem2 mem -t $thread run) is simpler

## Inputs

- Per-sample assembled contig FASTA files (contigs/*.fasta) from 4,472 vaginal metagenomic samples
- Cleaned paired-end reads (clean_reads/*.1.fq.gz, *.2.fq.gz)
- Mash distance top-20 neighbor lists (mash.sketch.msh.top20)
- Kraken2 report files (prof/*.report) for the Bracken stage

## Outputs

- Per-sample sorted BAMs and depth tables (depth/{sample}/{neighbor}.sort.bam.depth)
- Combined depth files and MetaBAT2 bins (bins/{/.}/, both single- and multiple-coverage)
- Kraken2/Bracken taxonomic profiles (prof/{sample}.report, *.bracken)
- Per-genome Prodigal outputs (*.faa, *.ffn, *.gff) feeding the ~26-million-gene clustering input

## How to apply

Express the per-item work as a single shell command template and let GNU parallel instantiate it over a find-generated file list, tuning -j to available cores (the VMGC README uses -j 10 to -j 20 depending on step cost). For Mash-based multiple-coverage binning, parallel drives mash dist over contigs/*.fasta, then bwa-mem2 index/mem plus jgi_summarize_bam_contig_depths per top-20 Mash-neighbor sample, then combine.pl and MetaBAT2 (-m 2000 -s 200000 --seed 2020) for both multi- and single-coverage bins. The same pattern parallelizes kraken2/bracken taxonomic profiling (--confidence 0.1, -r 150) and, per the source card, Prodigal 2.6.3 (-p meta) gene prediction across all VMGC prokaryotic, fungal and viral genomes. Keep the template's placeholder substitution explicit (--colsep '\t' when consuming tab-delimited Mash top-20 files) and redirect per-item outputs into structured directories (depth/{2}, bins/{/.}/, prof/{/}) so downstream steps can glob them deterministically.

## Related tools

- **GNU parallel 20201122** (Job fan-out engine executing Mash, bwa-mem2, MetaBAT2, Kraken2/Bracken and Prodigal commands per sample with -j concurrency control)
- **Mash 2.3** (Computes sketch distances whose top-20 neighbor lists parallel consumes to select samples for multiple-coverage depth calculation)
- **bwa-mem2 2.2.1** (Read mapping step parallelized per (neighbor-sample, assembly) pair to produce contig sequencing-depth BAMs)
- **combine.pl** (Depth-integration script invoked under parallel before multiple-coverage binning) — https://github.com/WatsonLab/single_and_multiple_binning/blob/main/scripts/combine.pl

## Examples

```
find contigs/*.fasta | parallel -k -j 10 mash dist mash.sketch.msh {} | sort -nk3 | head -n 20 > mash.sketch.msh.top20
```

## Evaluation signals

- Every input item appears exactly once in outputs: the number of .sort.bam.depth files equals the number of lines in mash.sketch.msh.top20 entries processed per assembly
- Parallel's -k flag preserves ordered output for the top-20 neighbor list; shuffled or missing lines indicate a dropped job
- Each bins/{/.}/ directory contains both a .sbin (single-coverage) and .mbin (multiple-coverage) set produced with identical MetaBAT2 parameters (-m 2000 -s 200000 --seed 2020)
- Combined depth file {}.depth exists for every directory listed by find depth/* -type d before MetaBAT2 multi-coverage binning starts
- Resulting MAG count is consistent with the paper's 63,654 preliminary MAGs reducing to 18,570 quality-screened prokaryotic genomes

## Limitations

- Per-job -j values (10-20) assume thread budgets within each command (e.g. bwa-mem2 -t 10, samtools -@ 10); oversubscription on smaller machines will thrash
- The README's perl one-liner read filter (identity>=97) is only offered as a speed optimization for very large datasets; results are stated to be equivalent but the fast path skips the full parallel loop of step 2

## Evidence

- [readme] Calculate the Mash distances between the assembled files, and obtain the top 20 closest samples for each assembled sample: "Calculate the Mash distances between the assembled files, and obtain the top 20 closest samples for each assembled sample"
- [readme] Integrate depth files and run multi-coverage binning under parallel: [evidence span withheld — anchor status: not_found]
- [abstract] Paper-scale motivation for the fan-out pattern: "we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity"
- [methods] GNU parallel listed as a required dependency of the binning workflow: "Mash 2.3<br>
    > GNU parallel 20201122<br>
    > bwa-mem2 2.2.1<br>"
