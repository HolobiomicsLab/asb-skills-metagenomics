---
name: spectral-networking-identification-propagation
description: Use when a validated set of RiPP peptide-spectrum matches exists and
  neighboring GNPS spectra (analyte analogs/variants) remain unannotated and should
  inherit identifications via molecular-networking families.
when_to_use_negative:
- Spectra lack paired genomic/metagenomic data — the article notes ~99% of GNPS spectra
  cannot be searched, and networking cannot create RiPP identifications where no seed
  PSM exists
- The compound class is non-ribosomal or of unknown origin with no RiPP seed match
  — families belong to non-RiPP classes and Dereplicator+ rather than RiPP propagation
  is appropriate
- Blind modification search is needed for cyclic peptides — MetaMiner does not perform
  blind modification searches for cyclic peptides due to computational complexity
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: MetaMiner
  role: Produces the validated RiPP peptide-spectrum matches (1% FDR, target-decoy)
    that seed identification propagation; available as the RIPPQUEST workflow on GNPS
  repo: https://github.com/mohimanilab/MetaMiner
- name: GNPS
  role: Hosts the molecular networking infrastructure that clusters spectra into families
    and propagates identifications across family members
- name: Dereplicator+
  role: Used to annotate spectral families against known chemical structures, revealing
    families that belong to non-ribosomal peptides
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:3c60869aa0db28beeeb70cf8773cc49a0cbd38318604e0d8b6f5fa81c1b01846
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - Dereplicator+
  - GNPS
  - HMMer
  - MS-DPR
  - MetaMiner
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spectral-networking-identification-propagation

## Summary

Propagates MetaMiner RiPP identifications (peptide-spectrum matches at 1% FDR) to related, unannotated spectra by clustering tandem MS spectra into spectral families, enlarging the identification set to analogs and variants within each family.

## When to use

Use when a validated set of RiPP peptide-spectrum matches exists and neighboring GNPS spectra (analyte analogs/variants) remain unannotated and should inherit identifications via molecular-networking families.

## When NOT to use

- Spectra lack paired genomic/metagenomic data — the article notes ~99% of GNPS spectra cannot be searched, and networking cannot create RiPP identifications where no seed PSM exists
- The compound class is non-ribosomal or of unknown origin with no RiPP seed match — families belong to non-RiPP classes and Dereplicator+ rather than RiPP propagation is appropriate
- Blind modification search is needed for cyclic peptides — MetaMiner does not perform blind modification searches for cyclic peptides due to computational complexity

## Inputs

- GNPS tandem mass spectrometry datasets (e.g., MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102)
- Validated RiPP peptide-spectrum matches from MetaMiner at 1% FDR
- MetaMiner target/decoy RiPP precursor database built from motif-ORF and all-ORF strategies

## Outputs

- Spectral families clustering the searched spectra (e.g., 8071 families from 10 million spectra)
- Enlarged RiPP identification set including propagated analogs (e.g., 38 RiPPs: 31 known, 7 novel)
- Family-level annotations distinguishing RiPP families from non-ribosomal peptide families

## How to apply

After MetaMiner identifies target RiPPs (p-values from MS-DPR, FDR via target-decoy at a 1% threshold per dataset), submit the GNPS tandem MS datasets to spectral networking so that the millions of spectra collapse into families (in this study, 10 million spectra clustered into only 8071 families). Identities validated for a seed spectrum are propagated across its family, enlarging the set of RiPP identifications to analogs without re-searching each spectrum against the target/decoy RiPP database. When applying Dereplicator-style family annotation to non-RiPP families, families can be further dereplicated against known chemical structures (e.g., with Dereplicator+), as many families in this study were found to be non-ribosomal peptides. The MetaMiner pipeline is available as the RIPPQUEST workflow on GNPS ProteoSAFe, so networking and propagation can run inside the same platform as the peptidogenomic search.

## Related tools

- **MetaMiner** (Produces the validated RiPP peptide-spectrum matches (1% FDR, target-decoy) that seed identification propagation; available as the RIPPQUEST workflow on GNPS) — https://github.com/mohimanilab/MetaMiner
- **GNPS** (Hosts the molecular networking infrastructure that clusters spectra into families and propagates identifications across family members)
- **Dereplicator+** (Used to annotate spectral families against known chemical structures, revealing families that belong to non-ribosomal peptides)

## Examples

```
https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22RIPPQUEST%22%7D
```

## Evaluation signals

- Spectra count vs. family count ratio is reasonable: in this study 10 million spectra collapsed into 8071 families
- Propagated identifications stay within 1% FDR: the seed PSMs must have passed the target-decoy FDR estimate before propagation
- Final identification table composition matches the expected 38 RiPPs (31 known, including AIP, Mec-PSM, delta-toxin, polytheonamide, wewakazole, lichenicidin; 7 unknown/novel)
- Propagated analogs appear as in-family spectral neighbors of seed PSMs (e.g., numerous analogs of the three lanthipeptides, lassopeptide, and cyclic cyanobactin)
- Known RiPPs such as wewakezole identified in families are confirmable by orthogonal evidence (isolation and NMR)

## Limitations

- Only ~1% of GNPS spectra have paired genomics/metagenomics data; 99% of spectra cannot seed RiPP identifications to propagate
- 10 million spectra yielded only 38 RiPPs, since many spectral families belong to non-ribosomal peptide classes rather than RiPPs
- No blind modification searches are performed for cyclic peptides due to inherent computational complexity
- Motif-ORF strategy misses precursor peptides without sequence similarity to known RiPPs, limiting the seed set available for propagation

## Evidence

- [readme] MetaMiner available as GNPS workflow: [evidence span withheld — anchor status: not_found]
