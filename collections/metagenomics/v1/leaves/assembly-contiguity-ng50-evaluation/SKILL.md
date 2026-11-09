---
name: assembly-contiguity-ng50-evaluation
description: Use when a strain-separated assembly (e.g. assembly.scaffolds.fa from
  Strainberry) must be scored against known reference genomes to quantify contiguity
  (NG50), base accuracy (ANI), and redundancy (duplication ratio).
when_to_use_negative:
- The input assembly is a strain-oblivious metagenome assembly (e.g. raw Flye/Canu
  output) that has not been through strain separation — NG50/ANI of mixed consensus
  contigs do not measure strain separation quality.
- No reference genomes are available for the community, so ANI and duplication ratio
  against references cannot be computed (only de novo contiguity metrics would apply).
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Strainberry
  role: produces the strain-resolved assemblies (assembly.scaffolds.fa) whose NG50,
    ANI, and duplication ratio are evaluated
  repo: https://github.com/rvicedomini/strainberry
- name: samtools
  role: used in the upstream pipeline (faidx/index of assembly and BAM) whose outputs
    feed the evaluation; also part of the strainberry-analyses Snakemake evaluation
    workflows
  repo: https://github.com/rvicedomini/strainberry-analyses
- name: minimap2
  role: provides the minimap2-based alignment of input reads on the output assembly
    used in evaluation steps
  repo: https://github.com/rvicedomini/strainberry
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:619a24077403a261ecb9449b38df08b8a24469a09a0a2380daa1e5c4fcd38a67
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
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# assembly-contiguity-ng50-evaluation

## Summary

Evaluate strain-resolved metagenome assemblies by computing per-strain average nucleotide identity (ANI), NG50, and duplication ratio against reference genomes, as done for Strainberry outputs on the Mock9 community (two S. aureus strains; three-way S. sonnei/two E. coli separation).

## When to use

Use when a strain-separated assembly (e.g. assembly.scaffolds.fa from Strainberry) must be scored against known reference genomes to quantify contiguity (NG50), base accuracy (ANI), and redundancy (duplication ratio).

## When NOT to use

- The input assembly is a strain-oblivious metagenome assembly (e.g. raw Flye/Canu output) that has not been through strain separation — NG50/ANI of mixed consensus contigs do not measure strain separation quality.
- No reference genomes are available for the community, so ANI and duplication ratio against references cannot be computed (only de novo contiguity metrics would apply).

## Inputs

- strain-resolved assembly FASTA (e.g. Strainberry assembly.scaffolds.fa) from Flye and Canu strain-oblivious assemblies
- Mock9 reference genome FASTA files (S. aureus ATCC/FDAA, K. pneumoniae, S. sonnei, E. coli strains)
- per-iteration intermediates in strainberry_n{k} sub-directories (optional)

## Outputs

- summary metrics table mock9_strainberry_metrics.tsv with per-strain ANI, NG50, and duplication ratio
- per-strain evaluation tables and graphs in results/[dataset]/evaluation

## How to apply

Collect the final strain-resolved assembly (assembly.scaffolds.fa) from each Strainberry output directory, including per-iteration intermediates in the strainberry_n{k} sub-directories if needed. Evaluate each output assembly per strain against the Mock9 references by computing ANI, NG50, and duplication ratio for every separated strain. Compare computed values against reported benchmarks: S. aureus ANI 99.97% with NG50 ~2.2/~2.7 Mbp for the two separated strains, K. pneumoniae ANI 99.99%, and three-strain separation (S. sonnei and two E. coli) duplication ratios 1.08/1.18/1.25. Overall base accuracy should be consistent with the paper's reported 99.9% on mock communities. Compile the metrics into a summary table (mock9_strainberry_metrics.tsv).

## Related tools

- **Strainberry** (produces the strain-resolved assemblies (assembly.scaffolds.fa) whose NG50, ANI, and duplication ratio are evaluated) — https://github.com/rvicedomini/strainberry
- **samtools** (used in the upstream pipeline (faidx/index of assembly and BAM) whose outputs feed the evaluation; also part of the strainberry-analyses Snakemake evaluation workflows) — https://github.com/rvicedomini/strainberry-analyses
- **minimap2** (provides the minimap2-based alignment of input reads on the output assembly used in evaluation steps) — https://github.com/rvicedomini/strainberry

## Examples

```
snakemake --use-conda --cores 12 --snakefile workflow/Snakefile-Mock.smk --configfile config/mock9.yaml
```

## Evaluation signals

- Computed S. aureus ANI equals the reported 99.97% and K. pneumoniae ANI equals 99.99%.
- NG50 values for the two separated S. aureus strains are close to ~2.2 Mbp and ~2.7 Mbp.
- Duplication ratios for the three-strain separation (S. sonnei and two E. coli) are close to 1.08/1.18/1.25.
- Overall base accuracy is consistent with the paper's reported 99.9% on mock communities.
- The summary table mock9_strainberry_metrics.tsv is populated for both the Flye and Canu Strainberry inputs.

## Limitations

- Strainberry achieved 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes on real datasets, so duplication ratios above 1 are expected and must be interpreted as redundancy, not failure.
- Haplotype phasing assumes evenly split read coverage, which does not hold when strains differ in abundance; only a limited number of well-represented strains were demonstrated to separate.
- Strainberry's ability to resolve plasmid genomes was not evaluated and is left for future work, so metrics may not capture plasmid content.

## Evidence

- [results] mock9_ng50_s aureus: [evidence span withheld — anchor status: not_found]
- [readme] readme_results_dirs: [evidence span withheld — anchor status: external_doc]
- [readme] readme_output_files: "Strainberry output assembly is stored in the `assembly.scaffolds.fa` file."
- [abstract] abstract_accuracy: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."
- [abstract] readme_extra_material: [evidence span withheld — anchor status: not_found]
