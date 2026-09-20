---
name: mass-spectra-similarity-network-construction
description: Use when thousands-to-millions of tandem MS/MS spectra from LC-MS/MS
  datasets (e.g. GNPS MSV datasets) must be grouped by fragmentation-pattern similarity,
  either to propagate peptide/spectrum identifications across related spectra or to
  estimate true chemical diversity.
when_to_use_negative:
- The goal is de novo identification of individual spectra rather than grouping related
  spectra — use MetaMiner/RiPPquest peptidogenomics search (1% FDR) instead of networking
  alone
- Spectra are known to be highly heterogeneous in precursor type/charge such that
  pairwise spectral similarity is not meaningful; networking assumes comparable fragmentation
  behavior within families
- Input spectra lack paired MS/MS fragmentation information (e.g. MS1-only features)
  — spectral networking requires tandem spectra
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS (Global Natural Products Social molecular networking)
  role: Hosts the molecular networking / spectral networks infrastructure used to
    compute pairwise spectral similarity and cluster ~10 million spectra into 8,071
    families
- name: MetaMiner
  role: Pipeline in which the spectral networking step enlarges RiPP identifications
    (38 RiPPs found) after 1% FDR peptidogenomics search; available as the RIPPQUEST
    workflow at GNPS and as a command line tool
  repo: https://github.com/mohimanilab/MetaMiner
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:4169a3b605eed19ac505e1591f67918422f821f54d6ebc267cb3a432c633d016
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - GNPS (Global Natural Products Social molecular networking)
  - MetaMiner
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mass-spectra-similarity-network-construction

## Summary

Clusters large collections of tandem mass spectra into spectral families via GNPS molecular networking / spectral networks (Bandeira et al., 2007; Watrous et al., 2012), collapsing raw spectrum counts into effective chemical diversity. In MetaMiner it both enlarges RiPP identifications and quantifies dataset redundancy (~10 million spectra resolving into only 8,071 families).

## When to use

Use when thousands-to-millions of tandem MS/MS spectra from LC-MS/MS datasets (e.g. GNPS MSV datasets) must be grouped by fragmentation-pattern similarity, either to propagate peptide/spectrum identifications across related spectra or to estimate true chemical diversity.

## When NOT to use

- The goal is de novo identification of individual spectra rather than grouping related spectra — use MetaMiner/RiPPquest peptidogenomics search (1% FDR) instead of networking alone
- Spectra are known to be highly heterogeneous in precursor type/charge such that pairwise spectral similarity is not meaningful; networking assumes comparable fragmentation behavior within families
- Input spectra lack paired MS/MS fragmentation information (e.g. MS1-only features) — spectral networking requires tandem spectra

## Inputs

- Tandem mass spectra (MS/MS peak lists / mzML-style data) from eight GNPS molecular networking datasets (MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102, and remaining analyzed datasets)
- Optionally, seed peptide-spectrum matches (RiPP identifications at 1% FDR) to propagate via spectral families

## Outputs

- Spectrum-to-family assignment table (spectral_families.tsv, TSV format)
- Count of distinct spectral families (expected 8,071 for the reference ~10-million-spectrum analysis)
- Enlarged set of RiPP identifications covering analog spectra within each family

## How to apply

Retrieve the tandem mass spectra from the GNPS molecular networking datasets (e.g. MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102, and the other analyzed datasets) from the GNPS infrastructure. Run GNPS molecular networking / spectral networks to compute pairwise spectral similarity between the ~10 million spectra, then cluster spectra with related fragmentation patterns into spectral families. In the MetaMiner pipeline this networking step is run after RiPP identification (peptide-spectrum matches scored with MS-DPR p-values and filtered at a 1% FDR target-decoy threshold) so that identified RiPP seeds propagate to their unannotated analog spectra. Export the clustering result as a spectrum-to-family assignment table (spectral_families.tsv) and validate by counting distinct families; in the reference analysis this yielded exactly 8,071 families from 10 million spectra.

## Related tools

- **GNPS (Global Natural Products Social molecular networking)** (Hosts the molecular networking / spectral networks infrastructure used to compute pairwise spectral similarity and cluster ~10 million spectra into 8,071 families)
- **MetaMiner** (Pipeline in which the spectral networking step enlarges RiPP identifications (38 RiPPs found) after 1% FDR peptidogenomics search; available as the RIPPQUEST workflow at GNPS and as a command line tool) — https://github.com/mohimanilab/MetaMiner

## Examples

```
# Launch MetaMiner (includes GNPS spectral networking) as a GNPS workflow: https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params={"workflow":"RIPPQUEST"}
```

## Evaluation signals

- The number of distinct spectral families in the exported spectral_families.tsv equals the reference count of 8,071 when the eight analyzed datasets (~10 million tandem spectra) are clustered
- Every input tandem MS/MS spectrum is assigned to exactly one spectral family (partition, not partial assignment)
- Known RiPP identifications (e.g. the 38 RiPPs at 1% FDR, or the 18 known RiPPs in the positive control dataset) seed families whose member spectra share related fragmentation patterns
- Family count (8,071) is orders of magnitude below raw spectrum count (10 million), confirming the redundancy-collapse behavior of the clustering

## Limitations

- 99% of GNPS spectra are not accompanied by genomics/metagenomics data and cannot be searched against genomes; networking alone identifies families but not their compound classes without additional tools (e.g. Dereplicator+)
- Many spectral families belong to non-ribosomal peptide classes rather than RiPPs, so a small family count does not imply all families are RiPPs
- MetaMiner does not currently perform blind modification searches for cyclic peptides due to inherent computational complexity, limiting what can be propagated within those families

## Evidence
