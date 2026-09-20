---
name: iterative-strain-separation-control-loop
description: Use when running Strainberry on a strain-oblivious long-read assembly
  plus indexed BAM alignment and you must decide how many separation iterations to
  run, when to stop, and how to organize per-iteration outputs (strainberry_n2/, strainberry_n3/,
  ...).
when_to_use_negative:
- Highly complex metagenomes — Strainberry is designed for single-sample low-complexity
  metagenomes and relies on well-represented strains
- Contigs that would need a variable number of strains separated within the same sequence
  — the paper notes this case is not handled
- Plasmid genome resolution — Strainberry's ability to resolve plasmids was not evaluated
  and was left for future work
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Strainberry
  role: 'Runs the iterative strain separation loop: phasing, per-haplotype assembly,
    and scaffolding, with stopping controlled by --max-strains (default 5) and the
    Hamming-rate improvement criterion'
  repo: https://github.com/rvicedomini/strainberry
- name: minimap2
  role: Generates the long-read alignments against the current scaffolds at each iteration
    and produces the final assembly.scaffolds.bam read alignment of the output assembly
- name: samtools
  role: Sorts, indexes (samtools index) and faidx-indexes the assembly and BAM inputs
    required by Strainberry's loop
- name: metaFlye
  role: Produces the initial strain-oblivious metagenome assembly that seeds the iterative
    separation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:3a60ac1e827d2b9de36d4996514aa2ceeda605223572f1d9eff4d6fca29a8f8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Strainberry
  - metaFlye
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# iterative-strain-separation-control-loop

## Summary

Controls Strainberry's iterative strain separation: repeated rounds of read re-phasing and per-haplotype re-assembly at increasing strain multiplicity, stopped by an average Hamming rate improvement threshold or the maximum multiplicity (--max-strains, default 5).

## When to use

Use when running Strainberry on a strain-oblivious long-read assembly plus indexed BAM alignment and you must decide how many separation iterations to run, when to stop, and how to organize per-iteration outputs (strainberry_n2/, strainberry_n3/, ...).

## When NOT to use

- Highly complex metagenomes — Strainberry is designed for single-sample low-complexity metagenomes and relies on well-represented strains
- Contigs that would need a variable number of strains separated within the same sequence — the paper notes this case is not handled
- Plasmid genome resolution — Strainberry's ability to resolve plasmids was not evaluated and was left for future work

## Inputs

- Strain-oblivious metagenome assembly in FASTA format (e.g., metaFlye assembly.fasta), samtools faidx-indexed
- Coordinate-sorted long-read alignment in BAM format (e.g., alignment.sorted.bam from minimap2), samtools-indexed
- Input long reads (PacBio HiFi by default, Oxford Nanopore via --nanopore)

## Outputs

- assembly.scaffolds.fa — final strain-separated scaffolds (with .fai index)
- assembly.scaffolds.bam (+ .bai) — minimap2 alignment of input reads on the output assembly
- strainberry_n2/, strainberry_n3/, ..., strainberry_nK/ — per-iteration intermediate results (e.g., variant VCFs) and log files

## How to apply

Start from the strain-oblivious assembly (FASTA, samtools faidx-indexed) and its coordinate-sorted, samtools-indexed BAM produced with metaFlye and minimap2. For iteration k=2..K, re-map long reads against the current Strainberry scaffolds and run long-read variant calling and haplotyping using an SNV density threshold of 0.1 (--snv-density) to define haplotype blocks; reads overlapping a phased region by at least 3 Kbp are assigned a Hamming rate to their closest haplotype. Trigger an additional separation only when the global average Hamming rate improves by at least 1% (with per-sequence local improvement checks), and stop when the criterion fails or the maximum strain multiplicity is reached (--max-strains, default 5). Store each iteration's intermediates in strainberry_n2/, strainberry_n3/, ... and emit the final strain-separated scaffolds as assembly.scaffolds.fa plus the minimap2 read alignment assembly.scaffolds.bam. For a sample with n strains the loop is expected to terminate after n-1 separation iterations (e.g., 1 iteration for the Mock3 ssFlye dataset with n=2).

## Related tools

- **Strainberry** (Runs the iterative strain separation loop: phasing, per-haplotype assembly, and scaffolding, with stopping controlled by --max-strains (default 5) and the Hamming-rate improvement criterion) — https://github.com/rvicedomini/strainberry
- **minimap2** (Generates the long-read alignments against the current scaffolds at each iteration and produces the final assembly.scaffolds.bam read alignment of the output assembly)
- **samtools** (Sorts, indexes (samtools index) and faidx-indexes the assembly and BAM inputs required by Strainberry's loop)
- **metaFlye** (Produces the initial strain-oblivious metagenome assembly that seeds the iterative separation)

## Examples

```
strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4
```

## Evaluation signals

- Per-iteration directories strainberry_n2/ through strainberry_nK/ exist and contain intermediates such as variant VCFs and logs
- For a dataset with n true strains, the loop terminates after n-1 separation iterations (e.g., strainberry_n2/ only for Mock3 with n=2)
- Final output contains assembly.scaffolds.fa (.fai), assembly.scaffolds.bam (.bai), and the scaffolds are strain-separated (e.g., one scaffold per strain in the E. coli K12/W example)
- The stopping logic respects --max-strains (default 5) and the >=1% average Hamming rate improvement threshold, with per-sequence local improvement checks applied
- Downstream evaluation shows near-complete reference coverage and ~99.9% base accuracy per strain genome on mock communities

## Limitations

- Haplotype phasing assumes evenly split read coverage; this does not hold when strains have different abundance/coverage levels, though it did not prevent separation of well-represented strains
- High strain variability might require separating a contig into a variable number of strains, which Strainberry does not handle
- Short read lengths (N50 around 5 Kbp due to DNA extraction limits) constrain haplotype resolution; 60-80x average coverage suffices for low-complexity metagenomes
- Relationship between coverage depth, read length, and haplotype resolution, as well as plasmid resolution, remain unaddressed

## Evidence

- [intro] Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding.: "Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding."
- [readme] All sub-directories named strainberry_n{k} contain intermediate results of Strainberry iterations (e.g. the VCF of variants found by mapping reads to contigs) and log files.: "All sub-directories named `strainberry_n{k}` contain intermediate results of Strainberry iterations (e.g. the VCF of variants found by mapping reads to contigs) and log files."
- [readme] -n int, --max-strains int Attempt strain-separation at most for the provided strain multiplicity [5]: "-n int, --max-strains int
                        Attempt strain-separation at most for the provided
                        strain multiplicity [5]"
- [abstract] We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy.: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."
- [readme] Strainberry should take around 5 minutes to finish. The file assembly.fasta contains a single sequence which is a consensus of a small region of E. coli strains K12 and W.: "Strainberry should take around 5 minutes to finish. The file `assembly.fasta` contains a single sequence which is a consensus of a small region of *E. coli* strains K12 and W."
