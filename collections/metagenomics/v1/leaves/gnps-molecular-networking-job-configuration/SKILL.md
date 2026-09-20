---
name: gnps-molecular-networking-job-configuration
description: Use when a large collection of tandem mass spectra (e.g., ~10M MS/MS
  spectra across eight GNPS datasets such as MSV000079506 or MSV000080102) must be
  clustered into spectral families via pairwise spectral similarity before or after
  spectral library searches.
when_to_use_negative:
- The input is already a cluster/family assignment table and the goal is only to count
  or filter families rather than recompute spectral networks
- Only RiPP identification (peptide-spectrum matching) is needed without family-level
  grouping — MetaMiner's MS-DPR p-value/FDR search against the target/decoy RiPP database
  is the wrong substitute for, and separate from, molecular networking
- The spectra of interest lack genomics/metagenomics partner data and the goal is
  RiPP discovery via genome search — 99% of GNPS spectra cannot be searched this way,
  though they can still be networked for family counting
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: GNPS
  role: Hosts and executes the molecular networking / spectral networks workflow that
    computes pairwise spectral similarity and clusters the ~10 million spectra into
    8,071 families; also hosts the MetaMiner workflow for RiPP identification
  repo: https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22RIPPQUEST%22%7D
- name: MetaMiner
  role: Peptidogenomics workflow whose RiPP identifications are enlarged via spectral
    networking; its family-level analysis contextualizes the 38 RiPPs found at 1%
    FDR within 8,071 families
  repo: https://github.com/mohimanilab/MetaMiner
- name: Dereplicator+
  role: Used downstream to search identified spectral families against known chemical
    structures, showing many families belong to non-ribosomal peptides
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:9114de4764b69f7d3424a9f48ee69699810ff9dd688ee67f9fffbee697cbef7e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Dereplicator+
  - GNPS
  - GNPS (Global Natural Products Social molecular networking)
  - MetaMiner
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# GNPS molecular networking job configuration

## Summary

Configure and run GNPS molecular networking (spectral networks, Bandeira et al. 2007; Watrous et al. 2012) jobs on tandem mass spectrometry datasets to compute pairwise spectral similarity and collapse large raw spectrum collections into spectral families. In the MetaMiner context this reduces ~10 million GNPS tandem MS/MS spectra to 8,071 families, exposing effective chemical diversity.

## When to use

Use when a large collection of tandem mass spectra (e.g., ~10M MS/MS spectra across eight GNPS datasets such as MSV000079506 or MSV000080102) must be clustered into spectral families via pairwise spectral similarity before or after spectral library searches.

## When NOT to use

- The input is already a cluster/family assignment table and the goal is only to count or filter families rather than recompute spectral networks
- Only RiPP identification (peptide-spectrum matching) is needed without family-level grouping — MetaMiner's MS-DPR p-value/FDR search against the target/decoy RiPP database is the wrong substitute for, and separate from, molecular networking
- The spectra of interest lack genomics/metagenomics partner data and the goal is RiPP discovery via genome search — 99% of GNPS spectra cannot be searched this way, though they can still be networked for family counting

## Inputs

- Tandem mass spectra (MS/MS) from GNPS molecular networking datasets (MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102, and remaining analyzed datasets)
- GNPS job configuration parameters (workflow selection, e.g., the RIPPQUEST workflow for MetaMiner integration)

## Outputs

- Spectrum-to-family assignment table in TSV format (spectral_families.tsv)
- Spectral similarity network with spectra clustered into families (8,071 families for the ~10 million-spectrum reference analysis)
- Optionally, Dereplicator+ annotations of spectral families against known chemical structures

## How to apply

Retrieve the tandem mass spectra for the target GNPS molecular networking datasets (e.g., MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102, and the remaining analyzed datasets) from the GNPS infrastructure. Launch a GNPS molecular networking / spectral networks job to compute pairwise spectral similarity between the ~10 million spectra (Bandeira et al., 2007; Watrous et al., 2012). Cluster the spectra into families so spectra with related fragmentation patterns are grouped, then export the spectrum-to-family assignment table (spectral_families.tsv) in TSV format. In the MetaMiner workflow, this spectral networking step is also used to enlarge the set of RiPP identifications after 1% FDR-filtered peptide-spectrum matches, and downstream family annotations can be obtained by searching family spectra against known structures with Dereplicator+. Validate by checking that the number of distinct families matches expectation (in the reference analysis, exactly 8,071).

## Related tools

- **GNPS** (Hosts and executes the molecular networking / spectral networks workflow that computes pairwise spectral similarity and clusters the ~10 million spectra into 8,071 families; also hosts the MetaMiner workflow for RiPP identification) — https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22RIPPQUEST%22%7D
- **MetaMiner** (Peptidogenomics workflow whose RiPP identifications are enlarged via spectral networking; its family-level analysis contextualizes the 38 RiPPs found at 1% FDR within 8,071 families) — https://github.com/mohimanilab/MetaMiner
- **Dereplicator+** (Used downstream to search identified spectral families against known chemical structures, showing many families belong to non-ribosomal peptides)

## Examples

```
GNPS web workflow launch: https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22RIPPQUEST%22%7D (per mohimanilab/MetaMiner README)
```

## Evaluation signals

- The exported spectrum-to-family table (spectral_families.tsv) parses as valid TSV and every input spectrum is assigned to exactly one family
- The number of distinct spectral families matches the reported value (8,071 families for the ~10 million-spectrum reference analysis)
- Family count is orders of magnitude smaller than raw spectrum count, consistent with the paper's finding that effective chemical diversity is far below the raw spectrum count
- The job ran on the GNPS infrastructure with the correct workflow parameters (e.g., the RIPPQUEST/MetaMiner workflow URL from the README resolves to a valid GNPS job)
- Downstream Dereplicator+ searches against known structures succeed on the exported families, indicating the family spectra are well-formed

## Limitations

- Only 38 RiPPs were identified from 10 million spectra; many spectral families belong to non-RiPP compound classes, so family clustering alone does not assign chemical class
- 99% of the spectra in GNPS are not accompanied by genomics/metagenomics data, making genome-search-based annotation of the resulting families impossible
- MetaMiner does not currently perform blind modification searches for cyclic peptides due to inherent computational complexity, which limits what can be propagated through RiPP-related families

## Evidence

- [readme] MetaMiner is currently available as a workflow at the Global Natural Products Social Molecular Networking (GNPS).: [evidence span withheld — anchor status: not_found]
