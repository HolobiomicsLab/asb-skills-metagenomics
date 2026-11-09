---
name: strain-duplication-ratio-assessment
description: Use when a strain-separated metagenome assembly (e.g., Strainberry assembly.scaffolds.fa)
  must be quantitatively assessed per reference strain using ANI, NG50, and duplication-ratio
  metrics.
when_to_use_negative:
- Input is a conventional strain-oblivious metagenome assembly (e.g., metaFlye output)
  rather than a strain-separated assembly, since per-strain duplication ratio is meaningless
  for mixed consensus contigs
- The sample is a high-complexity microbiome rather than a low-complexity mock community
  with reference genomes, since ANI/NG50/duplication ratio require strain-level references
  to compare against
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: Strainberry
  role: Produces the strain-resolved assemblies (assembly.scaffolds.fa) that are evaluated;
    run with -n 5, -s 0.1, -c threads on Flye/Canu inputs
  repo: https://github.com/rvicedomini/strainberry
- name: samtools
  role: Indexes the strain-oblivious assembly FASTA (samtools faidx) and coordinate-sorted
    BAM (samtools index) required before running Strainberry
- name: minimap2
  role: Aligns long reads to the assembly to build the sorted BAM input; Strainberry
    also emits a minimap2-based read alignment (assembly.scaffolds.bam) on the output
    assembly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:420da73ff46e618ebad58912cef5325a4390e09ea9785f9ce711aac15bfea100
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Strainberry
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# strain-duplication-ratio-assessment

## Summary

Evaluate strain-resolved metagenome assemblies (e.g., Strainberry output on Mock9) by computing per-strain average nucleotide identity (ANI), NG50, and duplication ratio against reference genomes, then comparing to the paper's reported values (e.g., S. aureus ANI 99.97%, NG50 ~2.2/~2.7 Mbp, duplication ratios 1.08/1.18/1.25).

## When to use

Use when a strain-separated metagenome assembly (e.g., Strainberry assembly.scaffolds.fa) must be quantitatively assessed per reference strain using ANI, NG50, and duplication-ratio metrics.

## When NOT to use

- Input is a conventional strain-oblivious metagenome assembly (e.g., metaFlye output) rather than a strain-separated assembly, since per-strain duplication ratio is meaningless for mixed consensus contigs
- The sample is a high-complexity microbiome rather than a low-complexity mock community with reference genomes, since ANI/NG50/duplication ratio require strain-level references to compare against

## Inputs

- Strain-resolved assembly FASTA (Strainberry assembly.scaffolds.fa) from Flye and Canu inputs
- Mock9 reference genomes (two S. aureus strains, K. pneumoniae, S. sonnei, two E. coli strains)
- Per-iteration Strainberry intermediates in strainberry_n{k} sub-directories (optional)

## Outputs

- Per-strain metrics table (mock9_strainberry_metrics.tsv) with ANI, NG50, and duplication ratio per strain and per input assembler
- Per-strain evaluation values comparable to reported figures (ANI 99.97%/99.99%, NG50 ~2.2/~2.7 Mbp, duplication ratios 1.08/1.18/1.25)

## How to apply

Run Strainberry on the strain-oblivious assemblies (Flye and Canu for Mock9) with default settings (strain multiplicity -n 5, minimum SNV density -s 0.1, multi-threaded via -c; add --nanopore only if reads are ONT rather than PacBio), ensuring both the assembly FASTA (samtools faidx) and the coordinate-sorted long-read BAM (samtools index) are indexed beforehand. Collect the final strain-resolved assemblies (assembly.scaffolds.fa) from each output directory along with the per-iteration intermediates in the strainberry_n{k} sub-directories. Align each output assembly against the Mock9 references (S. aureus ATCC/FDAA, K. pneumoniae, S. sonnei, and the two E. coli strains) to compute per-strain ANI, NG50, and duplication ratio. Compare computed metrics with the reported values: S. aureus ANI 99.97% with NG50 ~2.2/~2.7 Mbp, K. pneumoniae ANI 99.99%, and duplication ratios 1.08/1.18/1.25 for the three-strain S. sonnei/two-E. coli separation. Compile the per-strain metrics into a summary table (mock9_strainberry_metrics.tsv). The duplication ratio (total assembly bases assigned to a strain divided by reference length) flags over-assembly: paper-wide base accuracy was 99.9% on mock communities, and real datasets generated 20-118% additional genomic material over conventional assemblies.

## Related tools

- **Strainberry** (Produces the strain-resolved assemblies (assembly.scaffolds.fa) that are evaluated; run with -n 5, -s 0.1, -c threads on Flye/Canu inputs) — https://github.com/rvicedomini/strainberry
- **samtools** (Indexes the strain-oblivious assembly FASTA (samtools faidx) and coordinate-sorted BAM (samtools index) required before running Strainberry)
- **minimap2** (Aligns long reads to the assembly to build the sorted BAM input; Strainberry also emits a minimap2-based read alignment (assembly.scaffolds.bam) on the output assembly)

## Examples

```
strainberry -r flye_out/assembly.fasta -b flye_out/alignment.sorted.bam -o sberry_mock9_flye -n 5 -s 0.1 -c 12
```

## Evaluation signals

- Computed ANI per strain matches reported values: S. aureus ATCC/FDAA 99.97% and K. pneumoniae 99.99%
- NG50 of separated S. aureus strains is close to ~2.2 Mbp and ~2.7 Mbp
- Duplication ratios for the S. sonnei/two-E. coli three-strain separation are close to 1.08/1.18/1.25
- Overall base accuracy is consistent with the paper's reported 99.9% on mock communities
- Output contains assembly.scaffolds.fa plus strainberry_n2..nK intermediate sub-directories as documented

## Limitations

- Strainberry was benchmarked on low-complexity mock communities; on real datasets it generated 20-118% additional genomic material over conventional assemblies on individual strain genomes
- Haplotype phasing assumes read coverage is evenly split among haplotypes, which does not hold in metagenomes where strains have different abundances; even coverage separation was demonstrated only for a limited number of well-represented strains
- In the presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which is not handled
- Input reads should be indexed with samtools faidx/samtools index; use --nanopore only for ONT reads, otherwise defaults target PacBio

## Evidence

- [abstract] We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy.: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."
- [readme] Mock9 evaluation command: "strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4"
- [abstract] On real datasets, Strainberry generated 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes.: "We also applied Strainberry on real datasets for which it improved assemblies generating 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes."
- [discussion] Phasing coverage assumption limitation: "Haplotype phasing in single genomes benefits from read coverage being evenly split among the different haplotypes. This assumption no longer holds in a metagenomic setting where haplotypes/strains"
- [readme] Read alignment on output assembly: "A minimap2-based alignment of input reads on the output assembly is also available in the `assembly.scaffolds.bam` file."
