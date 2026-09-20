---
name: target-decoy-fdr-estimation
description: Use when validating RiPP peptide-spectrum matches produced by searching
  spectra against a putative precursor-peptide database (e.g., MetaMiner/Dereplicator
  output) and an acceptance threshold (e.g., 1% FDR per dataset) is needed before
  reporting identifications.
when_to_use_negative:
- Spectra lacking paired genomic/metagenomic data — the paper notes 99% of GNPS spectra
  cannot be searched, so there is no target-decoy RiPP search to validate
- Non-RiPP compound classes (e.g., non-ribosomal peptides) — Dereplicator+ structure-based
  search, not RiPP target-decoy peptidogenomics, is the appropriate route
- Blind modification searches on cyclic peptides — MetaMiner does not currently perform
  them due to computational complexity, so FDR results in that regime are out of scope
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: MetaMiner
  role: Runs the peptidogenomics pipeline and creates the decoy database by randomly
    shuffling each target precursor peptide; available as a GNPS workflow and command
    line tool
  repo: https://github.com/mohimanilab/MetaMiner
- name: Dereplicator
  role: Modified version used to score spectra against target and decoy RiPP databases,
    restricting and penalizing multiple modifications
  repo: https://github.com/mohimanilab/MetaMiner
- name: MS-DPR
  role: Computes p-values of peptide-spectrum matches used for target-decoy FDR estimation
- name: HMMer
  role: Detects modification enzymes that define the putative biosynthetic gene clusters
    from which target precursor peptides are drawn
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:93bcf2da8216c96ee70b430b915ab38d83746eafd65d4f0bc4b4fd189b14bffc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - HMMer
  - MS-DPR
  - MetaMiner
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# target-decoy-fdr-estimation

## Summary

Statistical validation strategy for RiPP identifications in MetaMiner: a decoy database is built by randomly shuffling each target precursor peptide, MS-DPR computes peptide-spectrum match p-values, and FDR estimated via the target-decoy approach gates identifications at a 1% threshold per dataset.

## When to use

Use when validating RiPP peptide-spectrum matches produced by searching spectra against a putative precursor-peptide database (e.g., MetaMiner/Dereplicator output) and an acceptance threshold (e.g., 1% FDR per dataset) is needed before reporting identifications.

## When NOT to use

- Spectra lacking paired genomic/metagenomic data — the paper notes 99% of GNPS spectra cannot be searched, so there is no target-decoy RiPP search to validate
- Non-RiPP compound classes (e.g., non-ribosomal peptides) — Dereplicator+ structure-based search, not RiPP target-decoy peptidogenomics, is the appropriate route
- Blind modification searches on cyclic peptides — MetaMiner does not currently perform them due to computational complexity, so FDR results in that regime are out of scope

## Inputs

- Target database of putative RiPP precursor peptides (motif-ORF and all-ORF strategies from 6-frame-translated genomes/metagenomic contigs)
- GNPS tandem mass spectra from paired genomic/metagenomic datasets (e.g., MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102)
- Dereplicator peptide-spectrum match scores for target and decoy hits

## Outputs

- Per-dataset list of RiPP identifications passing the 1% FDR threshold (known and unknown RiPPs)
- Target-decoy FDR estimates per peptide-spectrum match (from MS-DPR p-values)
- Comparison tables (e.g., 38 RiPPs = 31 known + 7 unknown; ACTI mode x FDR comparison)

## How to apply

After MetaMiner scores spectra against the target precursor database (built via motif-ORF/all-ORF from 6-frame translated genomes or assembled metagenomic contigs, with 10-kbp biosynthetic gene cluster windows around HMMer-detected modification enzymes), create a decoy database by randomly shuffling each target precursor peptide. Search spectra against both target and decoy databases with the modified Dereplicator scoring (restricting the number of blind modifications and penalizing matches with more than one modification), then compute p-values of peptide-spectrum matches with MS-DPR. Estimate FDR with the target-decoy approach and apply a 1% FDR threshold per dataset; enlarge the surviving identification set via spectral networks. Validate against the paper's benchmarks: 38 RiPPs (31 known + 7 unknown) from 10 million spectra, 18/18 known RiPPs in the positive control, and 8 RiPPs on the ACTI dataset at 1% FDR.

## Related tools

- **MetaMiner** (Runs the peptidogenomics pipeline and creates the decoy database by randomly shuffling each target precursor peptide; available as a GNPS workflow and command line tool) — https://github.com/mohimanilab/MetaMiner
- **Dereplicator** (Modified version used to score spectra against target and decoy RiPP databases, restricting and penalizing multiple modifications) — https://github.com/mohimanilab/MetaMiner
- **MS-DPR** (Computes p-values of peptide-spectrum matches used for target-decoy FDR estimation)
- **HMMer** (Detects modification enzymes that define the putative biosynthetic gene clusters from which target precursor peptides are drawn)

## Examples

```
git clone https://github.com/mohimanilab/MetaMiner  # run the MetaMiner CLI/GNPS RIPPQUEST workflow: target + shuffled-decoy search, MS-DPR p-values, 1% target-decoy FDR
```

## Evaluation signals

- Composition of the final table matches the reported 38 RiPPs at 1% FDR: 31 known (including AIP, Mec-PSM, delta-toxin, polytheonamide, wewakezole, lichenicidin) plus 7 unknown
- Positive control: MetaMiner correctly identifies all 18 of 18 known RiPPs at 1% FDR
- ACTI dataset reproduction: at 0% FDR motif-ORF mode yields 3 unknown + 5 known RiPPs while all-ORF yields 2 known; at 1% FDR 8 RiPPs total are identified
- Decoy database size and composition equal the target database (one shuffled peptide per target precursor)
- Per-dataset FDR is recomputed from target-decoy hit counts after MS-DPR p-value calculation, not inherited from other datasets

## Limitations

- The motif-ORF precursor strategy is not sensitive and misses peptides lacking sequence similarity to known RiPPs, shrinking the target space the FDR is computed over
- No blind modification searches are performed for cyclic peptides due to computational complexity
- When the producer genome is unavailable (e.g., the lacticin producer), spectra must be searched against previously described biosynthetic gene clusters instead of a genome-derived target/decoy pair
- The 1% FDR threshold is applied per dataset, so acceptance counts are not directly comparable across datasets without re-estimation

## Evidence

- [readme] MetaMiner is currently available as a workflow at the Global Natural Products Social Molecular Networking (GNPS).: [evidence span withheld — anchor status: not_found]
