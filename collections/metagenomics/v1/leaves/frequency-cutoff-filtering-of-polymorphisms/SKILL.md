---
name: frequency-cutoff-filtering-of-polymorphisms
description: Use when estimating the fraction of adaptive substitutions (α) from DAF/polymorphism
  and divergence counts and slightly deleterious rare variants must be excluded to
  avoid biasing α downward.
when_to_use_negative:
- When performing the standard MKT, which uses all polymorphisms without any frequency
  cutoff
- When the input is raw variation data (FASTA alignments or VCF) that has not yet
  been converted into a DAF spectrum and divergence count table by the iMKTData pipelines
- When polarized derived-allele states are unavailable, since the unfolded DAF spectrum
  (as produced from 1000GP ancestral-state or DGN data) is required to define the
  low-frequency cutoff
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: iMKT
  role: Web service that performs the eMKT (and FWWMKT, aMKT) MK-derived tests using
    the frequency-cutoff-corrected DAF and divergence counts
  repo: https://imkt.uab.cat
- name: iMKT (R package)
  role: R implementation that takes daf (daf, Pi, P0) and divergence (mi, Di, m0,
    D0) data frames, applies the cutoff, and returns alpha estimates
  repo: https://github.com/BGD-UAB/iMKT
- name: Python
  role: Used in the Jupyter (python/bash) notebooks to load the DAF/divergence table
    and apply the frequency cutoff to filter polymorphic variants
- name: bash
  role: Used alongside Python in the iMKTData pipeline notebooks to execute filtering
    and table-preparation steps
- name: Jupyter
  role: Notebook environment for running the step-by-step python/bash pipeline that
    produces the DAF spectrum files used in the cutoff analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:322b2e095b30a745c2ef900570d5492187d9a425b3c384c0ce18145e367630e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# frequency-cutoff-filtering-of-polymorphisms

## Summary

Excludes low-frequency (weakly deleterious) polymorphisms from a DAF spectrum table below a user-specified derived allele frequency cutoff before computing MK-derived statistics such as eMKT, FWWMKT, and aMKT, preventing downward bias in α estimates.

## When to use

Use when estimating the fraction of adaptive substitutions (α) from DAF/polymorphism and divergence counts and slightly deleterious rare variants must be excluded to avoid biasing α downward.

## When NOT to use

- When performing the standard MKT, which uses all polymorphisms without any frequency cutoff
- When the input is raw variation data (FASTA alignments or VCF) that has not yet been converted into a DAF spectrum and divergence count table by the iMKTData pipelines
- When polarized derived-allele states are unavailable, since the unfolded DAF spectrum (as produced from 1000GP ancestral-state or DGN data) is required to define the low-frequency cutoff

## Inputs

- Tab-delimited DAF spectrum table with columns daf, Pi (selected-class polymorphism), P0 (neutral-class polymorphism)
- Tab-delimited divergence table with columns mi, Di, m0, D0 (analyzed sites and divergence for selected and neutral classes)
- User-specified derived allele frequency cutoff threshold(s)

## Outputs

- Cutoff-corrected polymorphism counts (Ps, Pn) for selected and neutral classes
- Per-gene eMKT results (e.g., emkt_results.tsv) including alpha and its components
- MKT table of corrected polymorphism/divergence counts and divergence metrics (Ka, Ks, omega, omegaA)

## How to apply

Load the tab-delimited table containing the unfolded site frequency spectrum (DAF, Pi, P0) and divergence counts (mi, Di, m0, D0), with polymorphisms classified in DAF categories for the neutral (e.g., 4-fold/0) and selected (0-fold/i) classes. Apply the user-specified frequency cutoff by removing polymorphic variants whose derived allele frequency falls below the threshold, then recompute the corrected polymorphism totals (Ps, Pn) for both functional classes. Run the eMKT formulation — as implemented in the iMKT R package — computing alpha = 1 - (Ds/Dn)/(Ps/Pn) from the cutoff-corrected counts; the same cutoff mechanism is shared with the FWWMKT and aMKT methodologies. The cutoff value is chosen by the user and can be varied across several threshold values to assess sensitivity of α to the exclusion of weakly deleterious mutations.

## Related tools

- **iMKT** (Web service that performs the eMKT (and FWWMKT, aMKT) MK-derived tests using the frequency-cutoff-corrected DAF and divergence counts) — https://imkt.uab.cat
- **iMKT (R package)** (R implementation that takes daf (daf, Pi, P0) and divergence (mi, Di, m0, D0) data frames, applies the cutoff, and returns alpha estimates) — https://github.com/BGD-UAB/iMKT
- **Python** (Used in the Jupyter (python/bash) notebooks to load the DAF/divergence table and apply the frequency cutoff to filter polymorphic variants)
- **bash** (Used alongside Python in the iMKTData pipeline notebooks to execute filtering and table-preparation steps)
- **Jupyter** (Notebook environment for running the step-by-step python/bash pipeline that produces the DAF spectrum files used in the cutoff analysis)

## Examples

```
standardMKT(myDafData, myDivergenceData)$alpha.symbol
```

## Evaluation signals

- emkt_results.tsv exists, is a well-formed tab-delimited table, and contains the alpha column
- Recomputing alpha = 1 - (Ds/Dn)/(Ps/Pn) from the stored cutoff-corrected Ds, Dn, Ps, Pn values reproduces the reported alpha estimates
- Polymorphism totals after filtering are strictly less than or equal to the unfiltered totals, and no retained variant has derived allele frequency below the specified cutoff
- Fishers exact test P-value on the 2x2 corrected polymorphism/divergence table is reported and the MKT table row sums match the corrected counts (e.g., reference run gives alpha ≈ 0.2364 with P ≈ 1.48e-183)

## Limitations

- The cutoff is user-specified rather than objectively determined, so alpha estimates can vary with the chosen threshold value
- Excluding rare variants removes real neutral polymorphism as well as weakly deleterious variants, altering the polymorphism-to-divergence ratio used in α
- The input must be an unfolded DAF spectrum; folded spectra or mis-specified ancestral states would corrupt the low-frequency class being excluded

## Evidence

- [other] The eMKT handles rare (slightly deleterious) polymorphisms by excluding them from the analysis, with the user specifying one or several threshold frequency values: [evidence span withheld — anchor status: not_found]
- [other] the same exclusion mechanism shared with the FWWMKT and aMKT methods: [evidence span withheld — anchor status: not_found]
- [readme] It includes five MK derived methodologies which allow inferring the rate of adaptive evolution (α) as well as the fraction of strongly deleterious (d), weakly deleterious (b), and neutral (f) sites.: [evidence span withheld — anchor status: external_doc]
- [intro] a web-based service performing the four MKT types (standard MKT, FWWMKT, eMKT, aMKT) at once: "Here we present iMKT (acronym for integrative McDonald and Kreitman test), a web-based service performing the four MKT types described in Appendix 1 and Figure 1."
- [readme] Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional"
