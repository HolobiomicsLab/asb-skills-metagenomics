---
name: mock-community-scaffold-completeness-evaluation
description: Use when a strain-separated long-read metagenome assembly (e.g., assembly.scaffolds.fa
  from Strainberry on a mock community) must be evaluated for completeness and base-level
  accuracy against known reference strain genomes.
when_to_use_negative:
- The sample is a complex real microbiome without known reference strain genomes (e.g.,
  the Human Stool Microbiome dataset), where reference-coverage-based completeness
  benchmarks cannot be computed
- The input is a strain-oblivious assembly (e.g., metaFlye assembly.fasta) rather
  than a strain-resolved Strainberry output to be benchmarked
- The goal is strain separation itself (running Strainberry) rather than post-hoc
  evaluation of already-generated scaffolds
edam_operation: http://edamontology.org/operation_0310
edam_topics: []
tools:
- name: Strainberry
  role: Produces the strain-resolved scaffold assemblies (assembly.scaffolds.fa) that
    are benchmarked for reference coverage and base accuracy on mock communities
  repo: github.com/rvicedomini/strainberry
- name: Snakemake
  role: Runs the strainberry-analyses workflows (e.g., Snakefile-Mock.smk) that generate
    the assemblies and evaluation tables/graphs for each dataset
  repo: rvicedomini/strainberry-analyses
- name: minimap2
  role: Provides read alignments on the scaffolds (assembly.scaffolds.bam) used in
    the pipeline outputs underlying the evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:87491dcac7b134ff8a765bf350e4524abac1764b3eca7864e9c55373c8fc9d24
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Snakemake
  - Strainberry
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# mock-community-scaffold-completeness-evaluation

## Summary

Benchmarks strain-resolved long-read metagenome assemblies (e.g., Strainberry scaffolds from the Mock3/Mock9/NWC2 mock communities) against reference strain genomes, judging success by near-complete reference coverage and ~99.9% base accuracy.

## When to use

Use when a strain-separated long-read metagenome assembly (e.g., assembly.scaffolds.fa from Strainberry on a mock community) must be evaluated for completeness and base-level accuracy against known reference strain genomes.

## When NOT to use

- The sample is a complex real microbiome without known reference strain genomes (e.g., the Human Stool Microbiome dataset), where reference-coverage-based completeness benchmarks cannot be computed
- The input is a strain-oblivious assembly (e.g., metaFlye assembly.fasta) rather than a strain-resolved Strainberry output to be benchmarked
- The goal is strain separation itself (running Strainberry) rather than post-hoc evaluation of already-generated scaffolds

## Inputs

- Strainberry output scaffolds (assembly.scaffolds.fa) from a mock-community dataset
- Reference strain genomes of the mock community (e.g., Mock3)
- Strainberry analysis workflow configuration files (e.g., config/mock3.yaml, config/mock9.yaml, config/nwc2_pacbio.yaml, config/nwc2_ont.yaml)

## Outputs

- Evaluation tables in results/[dataset]/evaluation
- Evaluation graphs in results/[dataset]/evaluation
- Strain-resolved assemblies stored in results/[dataset]/assemblies

## How to apply

Run the Strainberry analysis Snakemake workflow for the appropriate mock dataset (e.g., Snakefile-Mock.smk with config/mock3.yaml), which produces assemblies in results/[dataset]/assemblies and evaluation tables and graphs in results/[dataset]/evaluation. Compare the strain-resolved scaffolds against the reference strain genomes of the mock community, quantifying reference coverage (expected to be near-complete) and base-level accuracy (expected around 99.9% as reported for Strainberry on mock communities). The evaluation should also contrast the strain-resolved output against conventional strain-oblivious metagenome assemblies, since Strainberry generated 20-118% additional genomic material on individual strain genomes for real datasets. Use the Mock3 and Mock9 configurations, and the NWC2 PacBio/Nanopore configurations, to reproduce the paper's main benchmark results. Success is judged by the evaluation tables showing near-complete coverage of each reference strain with sequence identity exceeding 99.9%.

## Related tools

- **Strainberry** (Produces the strain-resolved scaffold assemblies (assembly.scaffolds.fa) that are benchmarked for reference coverage and base accuracy on mock communities) — github.com/rvicedomini/strainberry
- **Snakemake** (Runs the strainberry-analyses workflows (e.g., Snakefile-Mock.smk) that generate the assemblies and evaluation tables/graphs for each dataset) — rvicedomini/strainberry-analyses
- **minimap2** (Provides read alignments on the scaffolds (assembly.scaffolds.bam) used in the pipeline outputs underlying the evaluation)

## Examples

```
snakemake --use-conda --cores 12 --snakefile workflow/Snakefile-Mock.smk --configfile config/mock3.yaml
```

## Evaluation signals

- Assemblies are present in results/[dataset]/assemblies and evaluation outputs in results/[dataset]/evaluation for the processed dataset (mock3, mock9, nwc2_pacbio, nwc2_ont, or hsm)
- Benchmark tables show near-complete reference coverage of the mock-community reference strain genomes
- Reported base accuracy is approximately 99.9%, matching Strainberry's published mock-community results
- Strain-resolved output accounts for more per-strain genomic material (20-118% additional) than a conventional strain-oblivious metagenome assembly on the same sample
- Strainberry output directory contains assembly.scaffolds.fa plus a minimap2-based read alignment assembly.scaffolds.bam (with .bai and .fai index files)

## Limitations

- Average depth of coverage of 60-80x suffices for near-complete strain recovery with >99.9% identity; lower coverage may not be evaluated meaningfully
- Haplotype phasing assumes read coverage split among haplotypes, which does not hold in metagenomes where strains have different abundance levels
- In presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which Strainberry does not handle
- Strainberry's ability to resolve plasmid genomes was not evaluated

## Evidence

- [abstract] We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy.: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."
- [readme] Assemblies and evaluation outputs directory structure: [evidence span withheld — anchor status: external_doc]
- [readme] Evaluation outputs: [evidence span withheld — anchor status: external_doc]
- [discussion] Coverage requirement for strain recovery: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
- [abstract] Additional genomic material vs conventional assemblies: "We also applied Strainberry on real datasets for which it improved assemblies generating 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes."
