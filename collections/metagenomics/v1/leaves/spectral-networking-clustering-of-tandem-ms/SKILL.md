---
name: spectral-networking-clustering-of-tandem-ms
description: Use when you have millions of tandem MS/MS spectra (e.g. GNPS molecular
  networking datasets such as MSV000079506–MSV000080102) and need to group spectra
  with related fragmentation patterns into spectral families before or after database
  search.
when_to_use_negative:
- The goal is de novo peptide-spectrum identification rather than clustering/diversity
  estimation — use MetaMiner/RiPPquest search with MS-DPR p-values and the 1% FDR
  target-decoy threshold instead.
- Spectra are cyclic peptides requiring blind modification search — the article notes
  blind modification searches are not performed for cyclic peptides due to computational
  complexity.
- Only a single spectrum or a tiny spectrum set is available — pairwise spectral networking
  adds no value over direct annotation (e.g. Dereplicator+).
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS (Global Natural Products Social molecular networking)
  role: Infrastructure hosting the spectra and running the spectral networks / molecular
    networking clustering step
- name: MetaMiner
  role: Pipeline whose spectral networking step clusters the searched spectra into
    families to enlarge RiPP identifications
  repo: https://github.com/mohimanilab/MetaMiner
- name: Dereplicator+
  role: Optionally searches the resulting spectral families against known chemical
    structures to assign compound classes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:2e3abdc7246c6a7cf25c4919ebddde7ecd9049e5434c7cbfb5ed06b15e42401a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator+
  - GNPS (Global Natural Products Social molecular networking)
  - MetaMiner
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectral-networking-clustering-of-tandem-ms

## Summary

Clusters large collections of tandem mass spectra into spectral families using GNPS molecular networking / spectral networks (Bandeira et al., 2007; Watrous et al., 2012), reducing millions of spectra to their effective chemical diversity. In MetaMiner it is used to enlarge RiPP identifications and to contextualize discovery yield (10 million spectra -> 8,071 families).

## When to use

Use when you have millions of tandem MS/MS spectra (e.g. GNPS molecular networking datasets such as MSV000079506–MSV000080102) and need to group spectra with related fragmentation patterns into spectral families before or after database search.

## When NOT to use

- The goal is de novo peptide-spectrum identification rather than clustering/diversity estimation — use MetaMiner/RiPPquest search with MS-DPR p-values and the 1% FDR target-decoy threshold instead.
- Spectra are cyclic peptides requiring blind modification search — the article notes blind modification searches are not performed for cyclic peptides due to computational complexity.
- Only a single spectrum or a tiny spectrum set is available — pairwise spectral networking adds no value over direct annotation (e.g. Dereplicator+).

## Inputs

- ~10 million tandem mass spectra from eight GNPS molecular networking datasets (MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102, plus two others)
- GNPS molecular networking workflow parameters (spectral similarity network settings)

## Outputs

- Spectrum-to-family assignment table in TSV format (spectral_families.tsv)
- Count of distinct spectral families (expected: 8,071 for the MetaMiner dataset)
- Enlarged set of RiPP identifications with spectral analogs

## How to apply

Retrieve the tandem mass spectra of interest (in MetaMiner's case, ~10 million spectra from eight GNPS datasets, e.g. MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102) from the GNPS infrastructure. Run GNPS molecular networking to compute pairwise spectral similarity between all spectra, then cluster the similarity network so that spectra with related fragmentation patterns are grouped into spectral families (spectral networks per Bandeira et al., 2007 and Watrous et al., 2012). Export the spectrum-to-family assignment as a TSV table (e.g. spectral_families.tsv) and count distinct families to quantify effective chemical diversity. In the MetaMiner pipeline, spectral networking is applied downstream of peptide-spectrum matching (MS-DPR p-values, target-decoy 1% FDR threshold) to enlarge the set of RiPP identifications with unannotated analogs; family members can then be searched against known structures with Dereplicator+ to assign compound classes.

## Related tools

- **GNPS (Global Natural Products Social molecular networking)** (Infrastructure hosting the spectra and running the spectral networks / molecular networking clustering step)
- **MetaMiner** (Pipeline whose spectral networking step clusters the searched spectra into families to enlarge RiPP identifications) — https://github.com/mohimanilab/MetaMiner
- **Dereplicator+** (Optionally searches the resulting spectral families against known chemical structures to assign compound classes)

## Examples

```
Run the GNPS spectral networking workflow on the eight GNPS datasets (MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102, ...) via https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22RIPPQUEST%22%7D; then verify: awk -F'\t' 'NR>1{print $2}' spectral_families.tsv | sort -u | wc -l  # expect 8071
```

## Evaluation signals

- Number of distinct spectral families in the exported TSV equals 8,071 for the ~10-million-spectrum MetaMiner dataset.
- Every input tandem mass spectrum is assigned to exactly one spectral family in spectral_families.tsv (no unassigned spectra).
- Family count is orders of magnitude smaller than raw spectrum count, confirming the expected compression of chemical diversity.
- Known RiPP identifications (e.g. the 38 RiPPs at 1% FDR) fall within families that contain their unannotated analog spectra.
- Families searched with Dereplicator+ classify sensibly (many belong to non-ribosomal peptides, not RiPPs).

## Limitations

- Only 38 RiPPs were identified from 10 million spectra, and many spectral families belong to non-RiPP compound classes — clustering indicates chemical diversity but not RiPP membership.
- 99% of GNPS spectra lack paired genomics/metagenomics data and cannot be searched against genomes, limiting downstream RiPP annotation of the families.
- Spectral networking does not perform blind modification searches for cyclic peptides due to inherent computational complexity.

## Evidence

- [readme] workflow availability at GNPS: [evidence span withheld — anchor status: not_found]
