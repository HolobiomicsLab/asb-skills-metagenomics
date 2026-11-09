---
name: large-scale-spectra-dataset-retrieval-and-processing
description: Use when a peptidogenomics or metabolomics analysis needs to operate
  on millions of GNPS tandem mass spectra at once — e.g. to run MetaMiner/RiPPquest
  searches, spectral networking, or to quantify effective chemical diversity of a
  GNPS dataset collection.
when_to_use_negative:
- When the spectra are not paired with any genomic or metagenomic data — the article
  notes ~99% of GNPS spectra lack such paired data and cannot be searched.
- When you only need a small, manually curated set of known RiPPs and do not need
  genome-linked spectrum searching or spectral networking.
- When the goal is de novo peptide identification from isolated metabolites rather
  than mining spectra against BGCs/genomes.
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: MetaMiner
  role: Searches the retrieved GNPS spectra against metagenomic/genomic short reads
    and motif-derived RiPP databases, producing scored peptide-spectrum matches filtered
    at 1% FDR.
  repo: https://github.com/mohimanilab/MetaMiner
- name: GNPS (Global Natural Products Social molecular networking)
  role: Hosts the ~10 million tandem mass spectra and provides the molecular networking/spectral
    networks pipeline used to cluster spectra into families.
- name: MS-DPR
  role: Computes p-values of peptide-spectrum matches that feed into the FDR estimation
    used on the retrieved spectra.
- name: RiPPquest
  role: Precursor tool extended by MetaMiner for RiPP discovery from mass spectrometry
    data.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:bb4db71057e74895618253c2b202e3ecb99130396cf131715ed99a19cc184c43
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - GNPS (Global Natural Products Social molecular networking)
  - MS-DPR
  - MetaMiner
  - RiPPquest
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# large-scale-spectra-dataset-retrieval-and-processing

## Summary

Retrieves and processes the ~10 million tandem mass spectra hosted on GNPS molecular networking datasets so they can be searched against genomic/metagenomic data and clustered via spectral networks; essential because a huge raw spectrum count collapses into a small effective chemical diversity (8,071 families).

## When to use

Use when a peptidogenomics or metabolomics analysis needs to operate on millions of GNPS tandem mass spectra at once — e.g. to run MetaMiner/RiPPquest searches, spectral networking, or to quantify effective chemical diversity of a GNPS dataset collection.

## When NOT to use

- When the spectra are not paired with any genomic or metagenomic data — the article notes ~99% of GNPS spectra lack such paired data and cannot be searched.
- When you only need a small, manually curated set of known RiPPs and do not need genome-linked spectrum searching or spectral networking.
- When the goal is de novo peptide identification from isolated metabolites rather than mining spectra against BGCs/genomes.

## Inputs

- GNPS molecular networking dataset accessions (e.g. MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102)
- Tandem mass spectra (MS/MS) from the GNPS molecular networking infrastructure
- Eight analyzed genomic and metagenomic datasets

## Outputs

- Set of RiPP peptide-spectrum matches at 1% FDR (38 RiPPs identified in the reference run)
- Spectral families (clusters of spectra with related fragmentation patterns) — 8,071 families for the ~10 million spectra
- Spectrum-to-family assignment table (TSV)

## How to apply

Retrieve the tandem mass spectra from the specific GNPS molecular networking datasets (e.g. MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102) rather than sampling ad hoc, since MetaMiner searches these spectra directly against genomic short reads. Pass the retrieved spectra through the GNPS/MetaMiner workflow so that peptide-spectrum matches are scored (p-values via MS-DPR) and filtered at a 1% FDR threshold using a target-decoy database. Then run spectral networks (Bandeira et al., 2007; Watrous et al., 2012) on the full corpus to cluster spectra with related fragmentation patterns into families. Validate the clustering by checking that the number of distinct families reported matches the published 8,071 for the ~10 million spectra, and that the raw spectrum count is not mistaken for chemical diversity.

## Related tools

- **MetaMiner** (Searches the retrieved GNPS spectra against metagenomic/genomic short reads and motif-derived RiPP databases, producing scored peptide-spectrum matches filtered at 1% FDR.) — https://github.com/mohimanilab/MetaMiner
- **GNPS (Global Natural Products Social molecular networking)** (Hosts the ~10 million tandem mass spectra and provides the molecular networking/spectral networks pipeline used to cluster spectra into families.)
- **MS-DPR** (Computes p-values of peptide-spectrum matches that feed into the FDR estimation used on the retrieved spectra.)
- **RiPPquest** (Precursor tool extended by MetaMiner for RiPP discovery from mass spectrometry data.)

## Examples

```
Run the MetaMiner workflow on GNPS (https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22RIPPQUEST%22%7D) to search ~10 million GNPS spectra against the eight genomic/metagenomic datasets and cluster the results via spectral networks.
```

## Evaluation signals

- Total spectra count should be ~10 million across the eight GNPS datasets analyzed.
- Spectral networking should yield 8,071 families, not a number proportional to the raw spectrum count.
- Peptide-spectrum matches should be filtered at a 1% FDR threshold; verify the FDR estimation was computed with the target-decoy approach.
- RiPP identifications should match the reference result of 38 RiPPs at 1% FDR for the full spectrum set.
- All spectra in the output should be traceable to a GNPS dataset accession listed in the analysis.

## Limitations

- Only ~1% of GNPS spectra have paired genomic/metagenomic data, so most spectra cannot be searched; searching is limited to the eight analyzed datasets.
- Blind modification searches are not currently performed for cyclic peptides due to inherent computational complexity.
- The spectral networking clustering is based on fragmentation similarity and may lump distinct compounds with similar spectra into the same family.
- The motif-ORF strategy misses peptides without sequence similarity to known RiPPs, limiting the recall of the underlying genome-linked search that consumes these spectra.

## Evidence

- [readme] workflow availability: [evidence span withheld — anchor status: not_found]
