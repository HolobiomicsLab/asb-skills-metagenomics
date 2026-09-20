---
name: in-silico-genome-evolution-simulation
description: Use when you need to benchmark a strain-tracking or genome-comparison
  tool's sensitivity to specific mutation types (SNPs vs indels) by simulating controlled
  in silico evolution and comparing pairwise score distributions across mutation regimes.
when_to_use_negative:
- When the goal is to compare real clinical/environmental strains rather than to benchmark
  tool sensitivity against known ground-truth mutation regimes — simulation-based
  APSS values do not transfer directly to biological interpretation.
- When the tool under evaluation is specifically designed to quantify SNP differences
  and structural changes are out of scope, since SynTracker's differentiating feature
  is its low SNP sensitivity and high indel sensitivity.
- When sequencing data are raw reads rather than assemblies or genomes, as SynTracker
  requires per-sample assemblies or assembled genomes as targets.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Computes per-region pairwise synteny scores and APSS from simulated target
    genomes against reference; run in 'new' mode with default parameters to quantify
    strain similarity under each mutation regime.
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: Underlying search engine used by SynTracker to find homologous central regions
    in target genomes with minimal identity 97% and minimal coverage 70% before synteny
    scoring.
- name: Bacmeta
  role: 'Generates the in silico evolved populations: SNP-only regime at 1e-6 events
    per nucleotide per generation and indel-only regime at 1e-7, producing ground-truth
    mutation-type data for benchmarking.'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:fdd120d74cc17dfec953de142f333ee83afca2e3769e5d16a818b7cfb3a9cbdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - Bacmeta
  - SynTracker
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# in-silico-genome-evolution-simulation

## Summary

Simulate bacterial genome evolution under controlled mutation regimes (SNP-only vs indel-only) using Bacmeta to benchmark the sensitivity of strain-comparison tools such as SynTracker, and quantify regime differences with Welch's t-tests on Average Pairwise Synteny Scores (APSS).

## When to use

Use when you need to benchmark a strain-tracking or genome-comparison tool's sensitivity to specific mutation types (SNPs vs indels) by simulating controlled in silico evolution and comparing pairwise score distributions across mutation regimes.

## When NOT to use

- When the goal is to compare real clinical/environmental strains rather than to benchmark tool sensitivity against known ground-truth mutation regimes — simulation-based APSS values do not transfer directly to biological interpretation.
- When the tool under evaluation is specifically designed to quantify SNP differences and structural changes are out of scope, since SynTracker's differentiating feature is its low SNP sensitivity and high indel sensitivity.
- When sequencing data are raw reads rather than assemblies or genomes, as SynTracker requires per-sample assemblies or assembled genomes as targets.

## Inputs

- Reference bacterial genome(s) in FASTA format for the Bacmeta simulation seed population
- Simulated target genomes (FASTA per time point/mutation regime) produced by Bacmeta
- SynTracker raw output table synteny_scores_per_region.csv (per-region pairwise synteny scores and BLAST identities)

## Outputs

- Simulated genome populations per mutation regime (SNP-only and indel-only) with 20 genomes per time point
- avg_synteny_scores_all_regions.csv (APSS computed over all available regions per pair)
- Welch's t-test statistics comparing SNP-based vs indel-based APSS distributions per time point (expected P < 2.2e-16)

## How to apply

Run Bacmeta to evolve genomes under two regimes: a SNP-only regime at 1e-6 events per nucleotide per generation and an indel-only regime at 1e-7, generating 3 genomic regions with 20 genomes per time point, yielding 190 pairwise comparisons per time point. Run SynTracker in 'new' mode with default parameters (minimal BLAST identity 97, minimal BLAST coverage 70, region length 5000, seed=1 for reproducibility) using the simulated genomes as targets. Extract per-region pairwise synteny scores and BLAST identities from synteny_scores_per_region.csv, then compute the Average Pairwise Synteny Score (APSS) over all available regions, producing avg_synteny_scores_all_regions.csv. Compare SNP-based vs indel-based APSS distributions per time point with Welch's t-test; correct application is verified when minimal APSS values match the expected pattern (SNP-based ~0.995/0.995/0.981 vs indel-based ~0.103/0.103/0.206) and P < 2.2e-16, demonstrating SynTracker's higher sensitivity to indels than SNPs.

## Related tools

- **SynTracker** (Computes per-region pairwise synteny scores and APSS from simulated target genomes against reference; run in 'new' mode with default parameters to quantify strain similarity under each mutation regime.) — https://github.com/leylabmpi/SynTracker
- **BLAST** (Underlying search engine used by SynTracker to find homologous central regions in target genomes with minimal identity 97% and minimal coverage 70% before synteny scoring.)
- **Bacmeta** (Generates the in silico evolved populations: SNP-only regime at 1e-6 events per nucleotide per generation and indel-only regime at 1e-7, producing ground-truth mutation-type data for benchmarking.)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Minimal average synteny scores for SNP-based simulations should be ~0.995, 0.995, 0.981, and for indel-based simulations ~0.103, 0.103, 0.206.
- Welch's t-test comparing SNP-based vs indel-based APSS distributions per time point must yield P < 2.2e-16.
- Each time point should contain 20 genomes per mutation regime, giving 190 pairwise comparisons per time point.
- The raw output table synteny_scores_per_region.csv must exist per reference genome under [genome_name]/final_output/, and avg_synteny_scores_all_regions.csv must be produced without subsampling.
- Reproducibility check: with the default seed=1, APSS values must be identical across repeated SynTracker runs on the same simulated inputs.

## Limitations

- SynTracker is deliberately less sensitive to SNPs than to indels, so SNP-only APSS values near 1 do not imply absence of divergence; interpretation requires combining with an SNP-based tool.
- APSS values are averaged over randomly subsampled regions; unless the default seed=1 is kept, results may change between runs.
- The synthetic regime (SNP frequency tenfold higher than indel frequency) is an artificial benchmark; real per-species evolutionary modes may differ, so simulated sensitivity does not directly calibrate APSS cutoffs for natural isolates.
- The APSS 0.95 same-strain cutoff discussed in the paper was based on averaging per-species cutoffs and may not be optimal for all settings.

## Evidence

- [results] Minimal average synteny scores were far higher in SNP-based simulations (0.995, 0.995, 0.981) than in indel-based simulations (0.103, 0.103, 0.206; P < 2.2 × 10−16): [evidence span withheld — anchor status: not_found]
- [methods] python syntracker.py -target <simulated_genomes_dir> -ref <reference_dir>: [evidence span withheld — anchor status: not_found]
- [readme] The table containing the APSS using all regions (without subsampling) is called avg_synteny_scores_all_regions.csv.: "The table containing the APSS using all regions (without subsampling) is called `avg_synteny_scores_all_regions.csv`."
- [readme] By default, a seed=1 is set to enable reproducibility between different runs.: "By default, a seed=1 is set to enable reproducibility between different runs."
