---
name: strain-variant-haplotyping-interpretation
description: Use when a Strainberry run on a strain-oblivious long-read metagenome
  assembly (e.g., Lathe/metaFlye FASTA plus coordinate-sorted Nanopore BAM) has finished
  and you must quantify per-bin additional genomic material, coverage halving, and
  strain-level classifications of the separated scaffolds.
when_to_use_negative:
- The metagenome is high-complexity rather than low-complexity; Strainberry is designed
  for single-sample low-complexity metagenomes and even-coverage separation was shown
  only for a limited number of well-represented strains
- You only need species-level binning or abundance profiling with no strain-resolution
  question — variant calling/haplotyping interpretation of separated scaffolds is
  unnecessary overhead
- Coverage of target strains is far below the ~60–80x depth the paper reports as sufficient
  to assemble individual strains with almost complete coverage and >99.9% sequence
  identity
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Strainberry
  role: Performs the strain separation whose outputs (assembly.scaffolds.fa, assembly.scaffolds.bam)
    are interpreted; run with --nanopore, default max-strains 5 and SNV density 0.1
  repo: https://github.com/rvicedomini/strainberry
- name: samtools
  role: Indexes the input FASTA/BAM (faidx, index) and computes per-scaffold coverage/depth
    statistics used to quantify coverage halving
- name: minimap2
  role: Produces the long-read alignment on the strain-separated scaffolds stored
    as assembly.scaffolds.bam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:f75072168631a7db79f595ff525f3867ac3e69a82a31ecf33a40018d37928cfb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Strainberry
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# strain-variant-haplotyping-interpretation

## Summary

Interpret Strainberry's strain-separated assembly outputs (assembly.scaffolds.fa and assembly.scaffolds.bam) by computing per-scaffold length and depth of coverage and comparing them against the strain-oblivious input assembly, quantifying additional genomic material and confirming halved coverage of separated strains, as demonstrated on the Human Stool Microbiome (HSM) V. atypica and E. eligens bins.

## When to use

Use when a Strainberry run on a strain-oblivious long-read metagenome assembly (e.g., Lathe/metaFlye FASTA plus coordinate-sorted Nanopore BAM) has finished and you must quantify per-bin additional genomic material, coverage halving, and strain-level classifications of the separated scaffolds.

## When NOT to use

- The metagenome is high-complexity rather than low-complexity; Strainberry is designed for single-sample low-complexity metagenomes and even-coverage separation was shown only for a limited number of well-represented strains
- You only need species-level binning or abundance profiling with no strain-resolution question — variant calling/haplotyping interpretation of separated scaffolds is unnecessary overhead
- Coverage of target strains is far below the ~60–80x depth the paper reports as sufficient to assemble individual strains with almost complete coverage and >99.9% sequence identity

## Inputs

- Strainberry output directory containing assembly.scaffolds.fa (strain-separated scaffolds) and assembly.scaffolds.bam (minimap2-based read alignment on the scaffolds)
- Strain-oblivious input assembly FASTA (e.g., Lathe/metaFlye assembly, indexed with samtools faidx) for baseline per-bin comparison
- Coordinate-sorted, samtools-indexed long-read (Nanopore) alignment BAM against the input assembly

## Outputs

- Per-scaffold sequence length and depth of coverage statistics (via samtools coverage/depth on assembly.scaffolds.bam)
- Per-bin additional genomic material percentages relative to the strain-oblivious assembly (e.g., ~86% for the V. atypica bin, ~118% for the E. eligens bin on HSM)
- strain_separation_summary.csv tabulating total length increase, additional genomic material per bin, halved coverage of separated scaffolds, and matching reference strains
- Strain-level classifications of separated scaffolds (e.g., strains ACS-134-V-Col7a and ACS-049-V-Sch6 for V. atypica)

## How to apply

Retrieve the strain-separated scaffolds from assembly.scaffolds.fa and the read alignment on them from assembly.scaffolds.bam in the Strainberry output directory. Compute per-scaffold sequence length and depth of coverage with samtools coverage/depth on assembly.scaffolds.bam, then compare these statistics against the input strain-oblivious (Lathe/metaFlye) assembly per bin to derive additional genomic material percentages and coverage changes. In the HSM dataset this pattern was ~+19% total assembly length, ~86% additional genomic material on the V. atypica bin (resolving strains ACS-134-V-Col7a and ACS-049-V-Sch6) and ~118% on the E. eligens bin. Confirm that depth of coverage of separated scaffolds is approximately halved relative to the strain-oblivious input, that the output scaffold count exceeds the input contig count, and tabulate the results into strain_separation_summary.csv. Scaffold sequences can additionally be classified (e.g., Kraken2 against Maxikraken2 as in the HSM workflow) to verify distinct strain-level assignments.

## Related tools

- **Strainberry** (Performs the strain separation whose outputs (assembly.scaffolds.fa, assembly.scaffolds.bam) are interpreted; run with --nanopore, default max-strains 5 and SNV density 0.1) — https://github.com/rvicedomini/strainberry
- **samtools** (Indexes the input FASTA/BAM (faidx, index) and computes per-scaffold coverage/depth statistics used to quantify coverage halving)
- **minimap2** (Produces the long-read alignment on the strain-separated scaffolds stored as assembly.scaffolds.bam)

## Examples

```
conda activate sberry; strainberry --nanopore -r lathe_assembly.fasta -b alignment.sorted.bam -o sberry_hsm_out -c 4; samtools coverage sberry_hsm_out/assembly.scaffolds.bam
```

## Evaluation signals

- Total assembly length increases by roughly +19% on the HSM dataset compared to the strain-oblivious Lathe assembly
- V. atypica bin yields ~86% additional genomic material and E. eligens bin ~118%, consistent with the reported 20–118% range on real datasets
- Depth of coverage of separated scaffolds is approximately halved relative to the strain-oblivious input contigs
- Output scaffold count exceeds the input contig count after separation
- Separated contigs classify to two distinct strain-level labels (ACS-134-V-Col7a and ACS-049-V-Sch6 for V. atypica), confirming strain resolution rather than duplicate sequences

## Limitations

- Strainberry version is not explicitly stated in the article; the analyses repo specifies version 1.1 with Snakemake 6.0.5
- Haplotype phasing assumes evenly split read coverage, which does not hold when strains have different abundance/coverage levels; even separation was shown only for a limited number of well-represented strains
- In presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which is not handled
- Limited read length (N50 around 5 Kbp due to DNA extraction constraints) and the relationship between coverage, read length, and haplotype resolvability merit further study

## Evidence

- [abstract] On real datasets, Strainberry generated 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes.: "We also applied Strainberry on real datasets for which it improved assemblies generating 20-118% additional genomic material than conventional metagenome assemblies"
- [results] HSM V. atypica additional genomic material finding: [evidence span withheld — anchor status: not_found]
- [readme] Output files structure and coverage alignment: [evidence span withheld — anchor status: not_found]
- [discussion] Coverage sufficiency for strain assembly: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
- [discussion] Even-coverage assumption caveat: "Haplotype phasing in single genomes benefits from read coverage being evenly split among the different haplotypes. This assumption no longer holds in a metagenomic setting"
