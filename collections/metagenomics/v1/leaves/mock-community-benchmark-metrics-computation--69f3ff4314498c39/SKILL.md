---
name: mock-community-benchmark-metrics-computation
description: Use when a newly generated strain-separated metagenome assembly must
  be quantitatively benchmarked per recovered strain against known reference genomes
  from a mock community (Mock3, Mock9).
when_to_use_negative:
- The sample is a complex/high-diversity microbiome with no closed reference genomes
  available, so ANI and coverage against references cannot be computed
- The input is a raw long-read FASTQ that has not yet been assembled and strain-separated
  (assemble and run Strainberry first)
- Evaluating plasmid or other extrachromosomal elements, which the authors explicitly
  did not assess
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Strainberry
  role: Produces the strain-separated assemblies whose scaffolds and read alignments
    are the subject of the evaluation metrics
  repo: https://github.com/rvicedomini/strainberry
- name: metaFlye
  role: Generates the strain-oblivious baseline (ssFlye) assembly from Mock3 PacBio
    reads that is benchmarked against the strain-separated output
- name: minimap2
  role: Aligns long reads to the output assembly to support coverage- and duplication-based
    evaluation
- name: samtools
  role: Sorts and indexes the read alignments used for computing per-strain metrics
- name: Snakemake
  role: Runs the reproducible mock-community analysis workflows that generate evaluation
    tables and graphs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:cbbaaba3575acf14e4e285ab030d911cc4303bbf728c0d634aa7d2d96f5e40f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Snakemake
  - Strainberry
  - metaFlye
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mock-community-benchmark-metrics-computation

## Summary

Compute per-strain evaluation metrics (average nucleotide identity, reference coverage, duplication ratio) on strain-separated assemblies of mock communities, mirroring the Strainberry Fig. 2 / Supplementary Data 1 tables comparing ssFlye/ssCanu against strain-oblivious Flye/Canu.

## When to use

Use when a newly generated strain-separated metagenome assembly must be quantitatively benchmarked per recovered strain against known reference genomes from a mock community (Mock3, Mock9).

## When NOT to use

- The sample is a complex/high-diversity microbiome with no closed reference genomes available, so ANI and coverage against references cannot be computed
- The input is a raw long-read FASTQ that has not yet been assembled and strain-separated (assemble and run Strainberry first)
- Evaluating plasmid or other extrachromosomal elements, which the authors explicitly did not assess

## Inputs

- Strain-separated assembly FASTA (assembly.scaffolds.fa) from each Strainberry run
- Minimap2-based read alignment on the output assembly (assembly.scaffolds.bam)
- Strain-oblivious baseline assemblies (metaFlye assembly.fasta and Canu assembly of the same reads)
- Reference genomes of the mock community strains (e.g., E. coli K-12 and W)

## Outputs

- Per-strain evaluation table (ANI, reference coverage, duplication ratio) per run, mirroring Fig. 2 / Supplementary Data 1
- Evaluation tables and graphs under results/[dataset]/evaluation

## How to apply

Take the final strain-separated scaffolds (assembly.scaffolds.fa) and the minimap2-based read alignments on the output assembly (assembly.scaffolds.bam) produced by Strainberry for each baseline assembly run (metaFlye -> ssFlye; Canu -> ssCanu). For each recovered strain in each run, align output scaffolds to the reference genomes and compute average nucleotide identity (ANI), reference coverage, and duplication ratio. Compile these per-strain values into a table mirroring Fig. 2 / Supplementary Data 1, placing ssFlye and ssCanu alongside the strain-oblivious Flye and Canu baselines. Accept results when ssFlye reaches 99.97% ANI for both E. coli K-12 and W with near-complete reference coverage and duplication ratios close to 1, while the Canu baseline matches the reported 99.64%/99.74% ANI and 1.23/1.39 duplication ratios. For reproducibility, the strainberry-analyses repository provides Snakemake workflows (Snakefile-Mock.smk with config/mock3.yaml or mock9.yaml) that generate evaluation tables and graphs under results/[dataset]/evaluation.

## Related tools

- **Strainberry** (Produces the strain-separated assemblies whose scaffolds and read alignments are the subject of the evaluation metrics) — https://github.com/rvicedomini/strainberry
- **metaFlye** (Generates the strain-oblivious baseline (ssFlye) assembly from Mock3 PacBio reads that is benchmarked against the strain-separated output)
- **minimap2** (Aligns long reads to the output assembly to support coverage- and duplication-based evaluation)
- **samtools** (Sorts and indexes the read alignments used for computing per-strain metrics)
- **Snakemake** (Runs the reproducible mock-community analysis workflows that generate evaluation tables and graphs)

## Examples

```
snakemake --use-conda --cores 12 --snakefile workflow/Snakefile-Mock.smk --configfile config/mock3.yaml
```

## Evaluation signals

- ssFlye ANI reaches 99.97% for both E. coli K-12 and W
- ssFlye shows near-complete reference coverage with duplication ratios close to 1
- ssCanu baseline reproduces the reported 99.64%/99.74% ANI and 1.23/1.39 duplication ratios
- Strain-separated assemblies show near-complete reference coverage and ~99.9% base accuracy on mock communities as reported in the abstract
- Evaluation tables and graphs appear in results/[dataset]/evaluation after running Snakefile-Mock.smk with the dataset config

## Limitations

- Strainberry was only evaluated on low-complexity mock communities; behavior on complex communities is less characterized
- The ability to resolve plasmid genomes was not evaluated and was left for future work
- In the presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which is not handled
- Duplication ratios greater than 1 (e.g., 1.23/1.39 for the Canu baseline) indicate residual duplication in strain-separated assemblies

## Evidence

- [abstract] Benchmarked mock communities with near-complete coverage and high identity: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."
- [results] Mock3 workflow step 5 from the source card: [evidence span withheld — anchor status: not_found]
- [readme] Evaluation outputs location in the analyses README: [evidence span withheld — anchor status: external_doc]
- [readme] Snakemake mock-community workflow usage: [evidence span withheld — anchor status: external_doc]
- [discussion] Depth of coverage sufficiency for individual strain assembly: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
