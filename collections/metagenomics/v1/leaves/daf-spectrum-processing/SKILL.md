---
name: daf-spectrum-processing
description: Use when input is a tab-delimited DAF spectrum plus divergence counts
  (Ps, Pn, Ds, Dn) per gene and the goal is to correct alpha estimates for slightly
  deleterious rare variants via a user-specified frequency cutoff (eMKT/FWWMKT-style
  exclusion).
when_to_use_negative:
- The analysis goal is the standard MKT or asymptotic MKT without low-frequency exclusion
  — use the corresponding iMKT method instead of applying a DAF cutoff
- Input data are raw FASTA alignments or 1000GP VCF files rather than an already-computed
  DAF/divergence table — run the iMKTData pipelines first to generate the SFS table
- Polymorphism counts are not stratified into selected (0-fold) and neutral (4-fold)
  functional classes, since the alpha calculation requires both classes
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3056
tools:
- name: iMKT
  role: Web service performing the four MKT types including eMKT, used to estimate
    alpha from the processed DAF and divergence data
  repo: https://imkt.uab.cat
- name: iMKT (R package)
  role: R implementation whose eMKT formulation defines the alpha computation on cutoff-corrected
    counts; functions take `daf` (DAF, Pi, P0) and `divergence` (mi, Di, m0, D0) data
    frames
  repo: https://github.com/BGD-UAB/iMKT
- name: Python
  role: Used in the Jupyter/python/bash notebooks to load the DAF/divergence table
    with pandas and apply the frequency cutoff filtering
- name: bash
  role: Raw scripts and custom parallel steps in the iMKTData pipelines that generate
    the tab-delimited DAF spectrum and divergence inputs
- name: Jupyter
  role: Notebook environment (Python 3.6 kernel) for the step-by-step dmelProteins/humanProteins
    pipelines producing the SFS tables
  repo: https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:bd9ec77c2c7ac59428493c3549c7933d84673d71de260ce1723266d5c0840452
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3056
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# daf-spectrum-processing

## Summary

Process an unfolded site frequency spectrum (DAF) and divergence count table (Ds, Dn, Ps, Pn for selected and neutral classes) to exclude low-frequency, weakly deleterious polymorphisms before estimating the fraction of adaptive substitutions (alpha) with the eMKT, as implemented in the iMKT R package.

## When to use

Use when input is a tab-delimited DAF spectrum plus divergence counts (Ps, Pn, Ds, Dn) per gene and the goal is to correct alpha estimates for slightly deleterious rare variants via a user-specified frequency cutoff (eMKT/FWWMKT-style exclusion).

## When NOT to use

- The analysis goal is the standard MKT or asymptotic MKT without low-frequency exclusion — use the corresponding iMKT method instead of applying a DAF cutoff
- Input data are raw FASTA alignments or 1000GP VCF files rather than an already-computed DAF/divergence table — run the iMKTData pipelines first to generate the SFS table
- Polymorphism counts are not stratified into selected (0-fold) and neutral (4-fold) functional classes, since the alpha calculation requires both classes

## Inputs

- Tab-delimited DAF spectrum table with per-gene unfolded derived allele frequency bins and polymorphism counts (Pi, P0 or Ps, Pn) for selected (0-fold) and neutral (4-fold) classes
- Divergence count table (mi, Di, m0, D0 or Ds, Dn) per gene, population, and MKT functional class
- User-specified low-frequency cutoff value(s) for excluding slightly deleterious polymorphisms

## Outputs

- emkt_results.tsv: per-gene tab-delimited table with cutoff-corrected alpha estimates and their components
- Recomputed corrected polymorphism counts (Ps, Pn) after cutoff filtering

## How to apply

Load the tab-delimited DAF/divergence table with pandas in a Jupyter (python/bash) notebook, as produced by the iMKTData pipelines. Apply the user-specified derived allele frequency cutoff, removing polymorphic variants below the threshold to control for weakly deleterious mutations, then recompute corrected polymorphism counts for the selected (0-fold) and neutral (4-fold) classes. Compute alpha = 1 - (Ds/Dn)/(Ps/Pn) from the cutoff-corrected counts, following the eMKT formulation of the iMKT R package (functions consume a `daf` data frame with DAF, Pi, P0 and a `divergence` data frame with mi, Di, m0, D0). Optionally cross-check results by running the same data through the iMKT web service (https://imkt.uab.cat) or the R package, which also reports Fisher's exact test P-value and Ka/Ks/omega/omegaA/omegaA divergence metrics.

## Related tools

- **iMKT** (Web service performing the four MKT types including eMKT, used to estimate alpha from the processed DAF and divergence data) — https://imkt.uab.cat
- **iMKT (R package)** (R implementation whose eMKT formulation defines the alpha computation on cutoff-corrected counts; functions take `daf` (DAF, Pi, P0) and `divergence` (mi, Di, m0, D0) data frames) — https://github.com/BGD-UAB/iMKT
- **Python** (Used in the Jupyter/python/bash notebooks to load the DAF/divergence table with pandas and apply the frequency cutoff filtering)
- **bash** (Raw scripts and custom parallel steps in the iMKTData pipelines that generate the tab-delimited DAF spectrum and divergence inputs)
- **Jupyter** (Notebook environment (Python 3.6 kernel) for the step-by-step dmelProteins/humanProteins pipelines producing the SFS tables) — https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- emkt_results.tsv exists, is a well-formed tab-delimited table, and contains an alpha column
- Recomputing alpha = 1 - (Ds/Dn)/(Ps/Pn) from the stored Ds, Dn, Ps, Pn values reproduces the reported alpha estimates
- Polymorphism counts in the output equal the counts after filtering DAF bins below the specified cutoff — bins below the threshold contribute zero
- The iMKT pipeline returns a Fisher's exact test P-value on the 2x2 polymorphism/divergence table, and the MKT table reports both neutral and selected class counts
- Divergence metrics (Ka, Ks, omega, omegaA, omegaD) accompany alpha and are consistent with the corrected counts

## Limitations

- The cutoff for excluding weakly deleterious variants is user-specified; results depend on the threshold chosen, and eMKT/FWWMKT/aMKT share this exclusion mechanism
- The iMKTData pipelines require downloaded external resources (DGN/PopFly FASTA for D. melanogaster, 1000GP Phase III VCF and PopHuman for human, plus recombination data and the 1000GP pilot mask) with paths that must be manually changed
- Pipeline steps may need to be parallelized via custom bash scripts or run manually; the reference environment used 100GB RAM and 16 Intel Xeon CPUs
- Unfolded SFS requires ancestral-state information (e.g. from PopHuman for human variants); polarized allele frequencies are an assumption of the input

## Evidence

- [full_text] The eMKT handles rare (slightly deleterious) polymorphisms by excluding them from the analysis, with the user specifying one or several threshold frequency values: [evidence span withheld — anchor status: not_found]
- [intro] Here we present iMKT (acronym for integrative McDonald and Kreitman test), a web-based service performing the four MKT types described in Appendix 1 and Figure 1.: "Here we present iMKT (acronym for integrative McDonald and Kreitman test), a web-based service performing the four MKT types described in Appendix 1 and Figure 1."
- [readme] most functions require two input parameters: daf (data frame containing DAF, Pi and P0) and divergence (data frame containing mi, Di, m0, D0) and return the estimation of alpha: [evidence span withheld — anchor status: not_found]
- [readme] Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional classes: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional"
- [methods] Load the tab-delimited DAF spectrum and divergence count table (Ds, Dn, Ps, Pn for selected and neutral site classes) with pandas in a Jupyter (python/bash) notebook.: [evidence span withheld — anchor status: not_found]
