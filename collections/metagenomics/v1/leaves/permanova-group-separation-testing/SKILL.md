---
name: permanova-group-separation-testing
description: Use when you have a relative-abundance or compositional feature table
  (e.g. MetaPhlAn2 or MAGinator taxon-by-sample matrices) plus two-level metadata
  (IBD vs control) and need a statistical test of between-group beta-diversity separation.
when_to_use_negative:
- When the input is raw reads or MAG clusters rather than a completed sample-by-taxon
  abundance table — the test requires a distance matrix computed from abundances.
- When group separation in species-level composition has already been established
  and the goal is taxon-level effect direction — use the Wilcoxon two-cohort discovery/replication
  differential abundance workflow instead.
- When samples lack two-level grouping metadata (e.g. IBD vs control); PERMANOVA requires
  a pre-defined grouping factor.
edam_operation: http://edamontology.org/operation_3797
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: MAGinator
  role: Produces the signature-gene-based MAG abundance matrices (abundance_phyloseq.RData,
    relative_abundance_phyloseq.RData) whose Jensen-Shannon beta diversity is tested
    with PERMANOVA against the MetaPhlAn2 baseline
  repo: https://github.com/Russel88/MAGinator
- name: MetaPhlAn
  role: Marker-gene-based profiler providing the original Franzosa et al. abundance
    matrix used as the PERMANOVA comparison baseline
- name: Snakemake
  role: Workflow engine running the MAGinator pipeline that generates the abundance
    tables entering the beta-diversity analysis
  repo: https://github.com/Russel88/MAGinator
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:14c3a35e7e8b49cc0001ae24e7342328c7aac5c0ffb6634cf7316ea7f17a68d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - MetaPhlAn
  - Snakemake
  - conda
  - mamba
  edam_operation: http://edamontology.org/operation_3797
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# permanova-group-separation-testing

## Summary

Tests whether pre-defined sample groups (e.g. IBD patients vs controls) separate in beta-diversity space by running PERMANOVA on a distance matrix and reporting F statistics and R², as done in the MAGinator reanalysis of the Franzosa et al. IBD study using Jensen-Shannon distances on MetaPhlAn2 and MAGinator abundance matrices with PCoA ordination.

## When to use

Use when you have a relative-abundance or compositional feature table (e.g. MetaPhlAn2 or MAGinator taxon-by-sample matrices) plus two-level metadata (IBD vs control) and need a statistical test of between-group beta-diversity separation.

## When NOT to use

- When the input is raw reads or MAG clusters rather than a completed sample-by-taxon abundance table — the test requires a distance matrix computed from abundances.
- When group separation in species-level composition has already been established and the goal is taxon-level effect direction — use the Wilcoxon two-cohort discovery/replication differential abundance workflow instead.
- When samples lack two-level grouping metadata (e.g. IBD vs control); PERMANOVA requires a pre-defined grouping factor.

## Inputs

- Taxon-by-sample abundance matrix from MetaPhlAn2 (original Franzosa et al. profiling)
- MAGinator signature-gene-based abundance matrices (abundance_phyloseq.RData, relative_abundance_phyloseq.RData)
- Filtered MAGinator abundance matrix
- Sample metadata with IBD patient vs healthy control labels

## Outputs

- Jensen-Shannon beta-diversity distance matrix per abundance matrix
- PCoA ordination of samples coloured by IBD/control status
- PERMANOVA pseudo-F and R² values per matrix (e.g. F=10.82/R²=4.81% for MetaPhlAn2)
- Results table ibd_permanova_da_results.tsv

## How to apply

Compute a beta-diversity distance matrix (here Jensen-Shannon distance) from each abundance matrix being compared — the original MetaPhlAn2 profile, the MAGinator matrix, and a filtered MAGinator matrix — then ordinate samples with PCoA to visualise group separation. Run PERMANOVA on each distance matrix against the IBD-vs-control grouping and record the pseudo-F and R² per matrix. In the Franzosa et al. reanalysis the expected benchmarks are F=10.82/R²=4.81% (MetaPhlAn2), F=10.21/R²=4.55% (MAGinator), and F=11.23/R²=4.99% (filtered MAGinator). Tabulate the F and R² values per matrix together with differential-abundance validation counts (18 discovery vs 213 validated taxa) and save as a TSV (ibd_permanova_da_results.tsv) so profiling strategies can be compared quantitatively.

## Related tools

- **MAGinator** (Produces the signature-gene-based MAG abundance matrices (abundance_phyloseq.RData, relative_abundance_phyloseq.RData) whose Jensen-Shannon beta diversity is tested with PERMANOVA against the MetaPhlAn2 baseline) — https://github.com/Russel88/MAGinator
- **MetaPhlAn** (Marker-gene-based profiler providing the original Franzosa et al. abundance matrix used as the PERMANOVA comparison baseline)
- **Snakemake** (Workflow engine running the MAGinator pipeline that generates the abundance tables entering the beta-diversity analysis) — https://github.com/Russel88/MAGinator

## Evaluation signals

- PERMANOVA pseudo-F and R² are reported for each matrix and fall near the published values: F=10.82/R²=4.81% (MetaPhlAn2), F=10.21/R²=4.55% (MAGinator), F=11.23/R²=4.99% (filtered MAGinator)
- PCoA ordination of the Jensen-Shannon distances shows visible IBD-vs-control separation consistent with the PERMANOVA result
- The distance matrix is square and symmetric with sample count matching the 216 assembled Franzosa et al. samples surviving preprocessing
- Results table ibd_permanova_da_results.tsv contains one F/R² row per matrix plus the 18-vs-213 discovery/validated taxa counts
- Beta diversity of the MAGinator matrix is comparable to the original MetaPhlAn2-based analysis (as reported in the paper's discussion of the Franzosa et al. data)

## Limitations

- High completeness of MAGinator's profiling may introduce noise or misclassification that influences the estimation of beta diversity
- PERMANOVA F and R² can drop slightly for the unfiltered MAGinator matrix (F=10.21/R²=4.55%) relative to the filtered matrix (F=11.23/R²=4.99%), so filtering choice affects the test outcome
- PERMANOVA only tests group separation in beta-diversity space; it does not identify which taxa drive the difference — pair it with the Wilcoxon discovery/replication differential abundance workflow

## Evidence

- [discussion] the beta diversity was comparable to the analysis carried out by Franzosa et al. Reanalysing their data demonstrates how MAGinator can be used for a metagenomic association study.: "the beta diversity was comparable to the analysis carried out by Franzosa et al. Reanalysing their data demonstrates how MAGinator can be used for a metagenomic association study."
- [discussion] However, it may also introduce a certain level of noise or misclassification, which influences the estimation of beta diversity.: "However, it may also introduce a certain level of noise or misclassification, which influences the estimation of beta diversity."
- [other] Tabulate the PERMANOVA F and R² values per matrix (target: F=10.82/R²=4.81% MetaPhlAn2, F=10.21/R²=4.55% MAGinator, F=11.23/R²=4.99% filtered): [evidence span withheld — anchor status: not_found]
- [other] Download the Franzosa et al. IBD raw reads from ENA (216 samples surviving assembly) and format them into a reads.csv sample index: [evidence span withheld — anchor status: not_found]
