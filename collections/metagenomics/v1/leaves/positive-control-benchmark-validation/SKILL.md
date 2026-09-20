---
name: positive-control-benchmark-validation
description: Use when a peptidogenomic or genome-vs-spectra search tool (e.g., MetaMiner/RiPPquest)
  must be benchmarked on known purified RiPP spectra before trusting identifications
  on novel microbial community datasets.
when_to_use_negative:
- Spectra lack paired genomic/metagenomic data — the article notes 99% of GNPS spectra
  are unaccompanied by genomics/metagenomics data and cannot be searched
- Cyclic peptide candidates requiring blind modification searches — these are not
  currently supported due to computational complexity
- The producer genome is unavailable and no described biosynthetic gene cluster sequences
  exist to substitute (as with lacticin, whose spectrum was searched against its described
  BGCs)
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0622
tools:
- name: MetaMiner
  role: 'Core peptidogenomics pipeline: 6-frame translation, modification-enzyme detection,
    precursor database construction, decoy generation, and RiPP identification of
    the positive-control spectra'
  repo: https://github.com/mohimanilab/MetaMiner
- name: Dereplicator
  role: Modified dynamic-programming search scoring peptide-spectrum matches against
    target/decoy RiPP precursor databases
- name: HMMer
  role: Detects RiPP modification enzymes in the 6-frame-translated producer genomes
- name: MS-DPR
  role: Computes p-values of peptide-spectrum matches used for the 1% FDR target-decoy
    filter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:c75ba9759c1cfff62a33453998bb0bdab78bf681a70b54ba4ca0ef822206d0bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - HMMer
  - MS-DPR
  - MetaMiner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# positive-control-benchmark-validation

## Summary

Validates a RiPP peptidogenomics pipeline by running it on a positive-control set of purified, known RiPPs against their producer genomes and confirming expected identifications at a defined FDR threshold, before applying the tool to discovery data.

## When to use

Use when a peptidogenomic or genome-vs-spectra search tool (e.g., MetaMiner/RiPPquest) must be benchmarked on known purified RiPP spectra before trusting identifications on novel microbial community datasets.

## When NOT to use

- Spectra lack paired genomic/metagenomic data — the article notes 99% of GNPS spectra are unaccompanied by genomics/metagenomics data and cannot be searched
- Cyclic peptide candidates requiring blind modification searches — these are not currently supported due to computational complexity
- The producer genome is unavailable and no described biosynthetic gene cluster sequences exist to substitute (as with lacticin, whose spectrum was searched against its described BGCs)

## Inputs

- Tandem MS spectra (.mgf/.mzML) of 18 purified isolated known RiPPs (prochlorosins, geobacillin, sublancin, haloduracin, lacticin, bicereusins, flavecins)
- NCBI RefSeq genomes of the corresponding RiPP producer organisms
- For lacticin (producer genome unavailable): previously described lacticin biosynthetic gene cluster sequences

## Outputs

- positive_control_results.csv: per-spectrum hit/miss table for the 18 known RiPPs
- List of peptide-spectrum matches with p-values and FDR estimates from MS-DPR
- Benchmark verdict (expected: 18/18 hits at 1% FDR)

## How to apply

Download standard positive-control spectra of 18 purified known RiPPs (prochlorosins, geobacillin, sublancin, haloduracin, lacticin, bicereusins, flavecins) and their corresponding NCBI RefSeq producer genomes. Run the MetaMiner pipeline, which 6-frame-translates each genome, detects modification enzymes with HMMer, builds putative precursor peptide databases via the motif-ORF and all-ORF strategies (all-ORF length threshold 200 amino acids), and creates a decoy database by randomly shuffling each target precursor peptide. Score peptide-spectrum matches with MetaMiner's modified Dereplicator dynamic-programming search (restricting the number of blind modifications and penalizing matches with more than one modification) and compute p-values with MS-DPR. Apply the 1% FDR threshold estimated with the target-decoy approach, then record for each of the 18 spectra whether the identification is a hit or a miss and compile the results (e.g., positive_control_results.csv). Validation succeeds only when all 18 of 18 known RiPPs are identified (18/18 hits at 1% FDR).

## Related tools

- **MetaMiner** (Core peptidogenomics pipeline: 6-frame translation, modification-enzyme detection, precursor database construction, decoy generation, and RiPP identification of the positive-control spectra) — https://github.com/mohimanilab/MetaMiner
- **Dereplicator** (Modified dynamic-programming search scoring peptide-spectrum matches against target/decoy RiPP precursor databases)
- **HMMer** (Detects RiPP modification enzymes in the 6-frame-translated producer genomes)
- **MS-DPR** (Computes p-values of peptide-spectrum matches used for the 1% FDR target-decoy filter)

## Evaluation signals

- All 18 of 18 known RiPPs in the positive-control dataset are identified (18/18 hits) when accepted at the 1% FDR threshold
- FDR is estimated with the target-decoy approach, where the decoy database is built by randomly shuffling each target precursor peptide
- All-ORF precursor extraction uses the default 200 amino acid ORF length threshold
- Each of the 18 spectra's identification matches its known RiPP (hit) rather than an unrelated precursor (miss); the lacticin spectrum is correctly identified against its described biosynthetic gene clusters since its producer genome is unavailable
- Matches with more than one modification are penalized per the modified Dereplicator scoring

## Limitations

- Since the genome sequence of the lacticin producer is not available, its spectrum was searched against its described biosynthetic gene clusters rather than a genome
- Motif-ORF strategy is not sensitive and misses peptides without sequence similarity to known RiPPs
- No blind modification searches are performed for cyclic peptides due to inherent computational complexity
- The tool is available as a GNPS workflow (RIPPQUEST); the repo README provides no standalone CLI usage details

## Evidence

- [readme] Tool availability and GNPS workflow: [evidence span withheld — anchor status: not_found]
