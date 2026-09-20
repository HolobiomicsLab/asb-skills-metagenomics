---
name: strain-resolution-benchmark-tabulation
description: Use when a strain-oblivious long-read metagenome assembly has been strain-separated
  with Strainberry and you must quantify per-bin additional genomic material, scaffold-count
  gains, and halved coverage into a summary table.
when_to_use_negative:
- The sample is a high-complexity metagenome rather than the low-complexity single-sample
  setting Strainberry was designed and benchmarked for
- You need to resolve plasmid genomes — Strainberry's ability to resolve plasmids
  was not evaluated and was explicitly left for future work
- Strains within a single contig need separation into a variable number of strains
  (high strain variability within one contig is not handled)
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Strainberry
  role: Performs the strain separation that produces the strain-resolved scaffolds
    being tabulated
  repo: https://github.com/rvicedomini/strainberry
- name: samtools
  role: faidx/index preparation of inputs and coverage/depth computation on output
    scaffolds for the summary metrics
- name: minimap2
  role: Underlies the read alignment on output scaffolds (assembly.scaffolds.bam)
    used for coverage estimation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:41fce573b23c4702d1ece55293a004b8b2d591f1f895d8512de81776ace564d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Strainberry
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# strain-resolution-benchmark-tabulation

## Summary

Tabulates Strainberry strain-separation results (per-bin additional genomic material, scaffold counts, and depth-of-coverage changes) against the input strain-oblivious assembly to verify strain-level resolution, e.g. ~86% extra genomic material for the V. atypica bin on the HSM dataset.

## When to use

Use when a strain-oblivious long-read metagenome assembly has been strain-separated with Strainberry and you must quantify per-bin additional genomic material, scaffold-count gains, and halved coverage into a summary table.

## When NOT to use

- The sample is a high-complexity metagenome rather than the low-complexity single-sample setting Strainberry was designed and benchmarked for
- You need to resolve plasmid genomes — Strainberry's ability to resolve plasmids was not evaluated and was explicitly left for future work
- Strains within a single contig need separation into a variable number of strains (high strain variability within one contig is not handled)

## Inputs

- Strain-oblivious metagenome assembly FASTA (e.g. Lathe or metaFlye assembly.fasta) with .fai index
- Coordinate-sorted long-read (Nanopore) alignment BAM with .bai index
- Strainberry output directory containing assembly.scaffolds.fa and assembly.scaffolds.bam

## Outputs

- strain_separation_summary.csv with total length increase, additional genomic material per bin, halved depth of coverage of separated scaffolds, and matching reference strains
- Per-scaffold length and coverage table computed with samtools coverage/depth on assembly.scaffolds.bam

## How to apply

Index the strain-oblivious Lathe/metaFlye assembly FASTA with 'samtools faidx' and the coordinate-sorted Nanopore read alignment with 'samtools index', then run strain separation with the --nanopore flag and defaults (max-strains 5, SNV density 0.1), e.g. 'strainberry --nanopore -r lathe_assembly.fasta -b alignment.sorted.bam -o sberry_hsm_out -c 4'. Retrieve 'assembly.scaffolds.fa' and 'assembly.scaffolds.bam' from the output directory. Compute per-scaffold sequence length and depth of coverage (samtools coverage/depth on assembly.scaffolds.bam) and compare per bin against the input assembly to derive additional genomic material percentages and coverage changes. Tabulate total length increase, additional genomic material per bin, halved depth of coverage of separated scaffolds, matching reference strains (e.g. ACS-134-V-Col7a and ACS-049-V-Sch6 for V. atypica), and scaffold counts into strain_separation_summary.csv. The rationale is that separated strain scaffolds should roughly halve coverage relative to the merged contig while increasing total assembly length, and Kraken2 classification (Maxikraken2 1903 database in the strainberry-analyses workflow) should assign distinct strain-level labels.

## Related tools

- **Strainberry** (Performs the strain separation that produces the strain-resolved scaffolds being tabulated) — https://github.com/rvicedomini/strainberry
- **samtools** (faidx/index preparation of inputs and coverage/depth computation on output scaffolds for the summary metrics)
- **minimap2** (Underlies the read alignment on output scaffolds (assembly.scaffolds.bam) used for coverage estimation)

## Examples

```
strainberry --nanopore -r lathe_assembly.fasta -b alignment.sorted.bam -o sberry_hsm_out -c 4 && samtools coverage sberry_hsm_out/assembly.scaffolds.bam
```

## Evaluation signals

- Total strain-separated assembly length increases by roughly +19% over the input Lathe assembly on the HSM dataset
- Additional genomic material on the V. atypica bin is ~86% and on the E. eligens bin ~118%, matching the reported 20–118% range
- Depth of coverage of separated scaffolds is approximately halved relative to the strain-oblivious input contigs
- Output scaffold count exceeds the input contig count
- Separated V. atypica contigs classify to two distinct strain-level labels (ACS-134-V-Col7a and ACS-049-V-Sch6) rather than a single species-level bin

## Limitations

- Relationship between sample depth of coverage, read length, and haplotype-resolving ability merits further study
- DNA extraction methods producing high molecular weight DNA for all bacteria are lacking, limiting read length to around an N50 of 5 Kbp
- Phasing assumes evenly split read coverage, which does not hold when strains have different abundance/coverage levels; demonstrated only for a limited number of well-represented strains
- Strainberry version was not stated explicitly in the article (analyses repo tested with Strainberry version 1.1)

## Evidence

- [other] On the HSM dataset, Strainberry's separation of the V. atypica bin yielded 86% additional genomic material: [evidence span withheld — anchor status: not_found]
- [abstract] Strainberry generated 20-118% additional genomic material: "We also applied Strainberry on real datasets for which it improved assemblies generating 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes."
- [readme] Output files of Strainberry: "The output directory of Strainberry has the following structure ... The output assembly is stored in the `assembly.scaffolds.fa` file.
A minimap2-based alignment of input reads on the output assembly"
- [discussion] Even coverage assumption limitation: "Haplotype phasing in single genomes benefits from read coverage being evenly split among the different haplotypes. This assumption no longer holds in a metagenomic setting where haplotypes/strains"
- [readme] Required indexed inputs: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."
