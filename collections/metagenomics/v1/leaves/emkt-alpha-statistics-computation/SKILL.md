---
name: emkt-alpha-statistics-computation
description: Use when polymorphism/divergence tables (DAF, Pi, P0, mi, Di, m0, D0)
  are available for D. melanogaster genes and you need alpha estimates per gene or
  per gene set, e.g. comparing high-recombination regions against all genes in a population
  such as ZI.
when_to_use_negative:
- Input is raw variant data (VCF/FASTA alignments) rather than pre-computed polymorphism/divergence
  SFS tables — use the iMKTData D. melanogaster pipeline first
- You need the four MKT variants (standard MKT, FWW, aMKT, eMKT) compared simultaneously
  with custom gene filtering outside iMKT's options rather than a single eMKT alpha
  summary
- The population is not covered by PopFly/DGN pre-loaded data and the user cannot
  supply correctly formatted daf/divergence data frames
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3299
tools:
- name: iMKT (R package)
  role: Runs the eMKT methodology on daf and divergence data frames to estimate alpha
    per gene set
  repo: https://github.com/BGD-UAB/iMKT
- name: Python
  role: Used in the iMKTData D. melanogaster pipeline to extract unfolded SFS, divergence,
    and functional-class (0-fold/4-fold) counts per gene
  repo: github.com/BGD-UAB/iMKTData
- name: bash
  role: Executes and parallelizes pipeline steps of the iMKTData D. melanogaster preprocessing
    workflow
  repo: github.com/BGD-UAB/iMKTData
- name: Jupyter
  role: Hosts the step-by-step dmelProteins notebook that runs the D. melanogaster
    pipeline generating iMKT input tables
  repo: github.com/jmurga/iMKTData
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:e1b43ee58be93cf780664a1f9336861913adc7580b63294f8c04789149c0d14a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGD-UAB/iMKT
  tools:
  - Jupyter
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# emkt-alpha-statistics-computation

## Summary

Compute the rate of adaptive evolution (alpha) per gene or gene set using the eMKT (asymptotic/excluding weakly deleterious) MK-derived methodology implemented in the iMKT R package, and summarize alpha as mean and standard deviation for user-defined gene groups (e.g., high-recombination genes vs. the genome-wide set) to test whether recombination rate modulates adaptation.

## When to use

Use when polymorphism/divergence tables (DAF, Pi, P0, mi, Di, m0, D0) are available for D. melanogaster genes and you need alpha estimates per gene or per gene set, e.g. comparing high-recombination regions against all genes in a population such as ZI.

## When NOT to use

- Input is raw variant data (VCF/FASTA alignments) rather than pre-computed polymorphism/divergence SFS tables — use the iMKTData D. melanogaster pipeline first
- You need the four MKT variants (standard MKT, FWW, aMKT, eMKT) compared simultaneously with custom gene filtering outside iMKT's options rather than a single eMKT alpha summary
- The population is not covered by PopFly/DGN pre-loaded data and the user cannot supply correctly formatted daf/divergence data frames

## Inputs

- Unfolded site frequency spectrum / DAF data frame per gene (columns: daf, Pi, P0) from PopFly or iMKTData pipelines for the ZI population
- Divergence data frame per gene (columns: mi, Di, m0, D0) between D. melanogaster and D. simulans
- Gene set definition partitioning 13,753 protein-coding genes into high-recombination vs. genome-wide groups using the Comeron et al. 2012 recombination map

## Outputs

- Per-gene alpha estimates from eMKT for the high-recombination gene set and the whole-genome gene set
- Group-level summary statistics (mean and SD of alpha) written to results_zi_emkt_recombination.tsv
- MKT table with polymorphism/divergence counts and Fisher's exact test P-value per analysis

## How to apply

Load the pre-calculated D. melanogaster polymorphism and divergence tables (e.g. the Zambia/ZI population data provided with iMKT or downloadable from PopFly/Drosophila Genome Nexus). Partition the 13,753 protein-coding genes into the target set (high-recombination genes defined via the Comeron et al. 2012 recombination map) and the full genome-wide set. Run the eMKT function from the iMKT R package on each gene set, feeding the daf data frame (columns daf, Pi, P0) and the divergence data frame (columns mi, Di, m0, D0) to obtain alpha per gene. Compute mean and standard deviation of alpha for each group and compare against reference values (high-recombination alpha mean 0.633 ± 0.028; genome-wide alpha mean 0.457 ± 0.053 for ZI). Rationale: eMKT corrects the standard MKT alpha estimate by excluding weakly deleterious variants, and iMKT's web/R implementation was designed precisely to compare a user-selected gene set (e.g. recombination rate bins) with the genome-wide distribution.

## Related tools

- **iMKT (R package)** (Runs the eMKT methodology on daf and divergence data frames to estimate alpha per gene set) — https://github.com/BGD-UAB/iMKT
- **Python** (Used in the iMKTData D. melanogaster pipeline to extract unfolded SFS, divergence, and functional-class (0-fold/4-fold) counts per gene) — github.com/BGD-UAB/iMKTData
- **bash** (Executes and parallelizes pipeline steps of the iMKTData D. melanogaster preprocessing workflow) — github.com/BGD-UAB/iMKTData
- **Jupyter** (Hosts the step-by-step dmelProteins notebook that runs the D. melanogaster pipeline generating iMKT input tables) — github.com/jmurga/iMKTData

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- Computed mean alpha for the ZI high-recombination gene set reproduces 0.633 (SD 0.028)
- Computed mean alpha for the full 13,753-gene set reproduces 0.457 (SD 0.053)
- Input data frames match iMKT schema: daf frame has columns daf, Pi, P0; divergence frame has columns mi, Di, m0, D0
- Fisher's exact test P-value reported by the MKT function is significant (as in the README example, ~1.5e-183 scale) indicating the test is valid for the gene set
- Output file results_zi_emkt_recombination.tsv contains per-group alpha statistics for exactly the two gene groups

## Limitations

- The method operates on protein-coding genes with 0-fold (selected) and 4-fold (neutral) classes; non-coding or unannotated sequences are out of scope
- Results depend on the pre-loaded PopFly/DGN variation and divergence data; pipelines required large resources (100GB RAM, 16 Xeon CPUs) to regenerate tables
- MKT-derived methodologies behave differently according to data properties, so eMKT alpha values may differ from standard MKT or aMKT estimates for the same gene set

## Evidence

- [abstract] Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?"
- [intro] It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the: "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the"
- [methods] Recomb data from Comeron et al. 2012: "Recomb data from [Comeron et al. 2012](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1002905)"
- [readme] iMKT is an R package to compute the McDonald and Kreitman test (McDonald and Kreitman 1991 Nature) on polymorphism and divergence genomic data provided by the user or automatically downloaded from PopFly: [evidence span withheld — anchor status: external_doc]
- [readme] most functions require two input parameters: daf (data frame containing DAF, Pi and P0) and divergence (data frame containing mi, Di, m0, D0) and return the estimation of α: [evidence span withheld — anchor status: not_found]
