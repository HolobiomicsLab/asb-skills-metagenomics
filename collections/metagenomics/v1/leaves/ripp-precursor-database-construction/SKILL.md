---
name: ripp-precursor-database-construction
description: Use when paired genomic/metagenomic contigs and GNPS tandem MS spectra
  must be searched for RiPPs and a putative precursor peptide database (target plus
  shuffled decoy) needs to be built first.
when_to_use_negative:
- The tandem mass spectra have no paired genomic/metagenomic data — the paper notes
  99% of GNPS spectra lack such pairing, making precursor database construction impossible
- The candidate RiPP is a cyclic peptide — no blind modification searches are performed
  for cyclic peptides due to computational complexity
- Only the metabolite is available without genome sequence (e.g., lacticin, whose
  producer genome is unavailable and whose spectrum was searched against described
  biosynthetic gene clusters instead)
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0622
tools:
- name: MetaMiner
  role: Command-line pipeline that 6-frame-translates DNA, runs motif-ORF/all-ORF
    precursor peptide database construction, and builds the shuffled decoy database
  repo: https://github.com/mohimanilab/MetaMiner
- name: HMMer
  role: Detects modification enzymes on translated sequences to delimit the 10kbp
    gene-cluster windows from which precursor peptides are extracted
- name: Dereplicator
  role: Modified version used to search spectra against the constructed target/decoy
    RiPP precursor peptide databases with blind modification scoring
- name: MS-DPR
  role: Computes p-values of peptide-spectrum matches against the constructed databases
- name: GNPS
  role: Hosts MetaMiner as the RIPPQUEST workflow on which the precursor database
    construction is run
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:1b6c53553925fa251eafafe832c76daed998bcef8db35cc2bae9ba3b28e7498e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - GNPS
  - HMMer
  - MS-DPR
  - MetaMiner
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ripp-precursor-database-construction

## Summary

Constructs the target and decoy databases of putative RiPP precursor peptides that MetaMiner searches tandem mass spectra against, using motif-ORF and all-ORF strategies on 6-frame-translated genomic/metagenomic sequence after HMMer detection of modification enzymes.

## When to use

Use when paired genomic/metagenomic contigs and GNPS tandem MS spectra must be searched for RiPPs and a putative precursor peptide database (target plus shuffled decoy) needs to be built first.

## When NOT to use

- The tandem mass spectra have no paired genomic/metagenomic data — the paper notes 99% of GNPS spectra lack such pairing, making precursor database construction impossible
- The candidate RiPP is a cyclic peptide — no blind modification searches are performed for cyclic peptides due to computational complexity
- Only the metabolite is available without genome sequence (e.g., lacticin, whose producer genome is unavailable and whose spectrum was searched against described biosynthetic gene clusters instead)

## Inputs

- Genomic or metagenomic DNA sequence (contigs assembled from short reads, or NCBI RefSeq producer genomes)
- HMMer-detected modification enzyme positions defining 10kbp putative biosynthetic gene cluster windows
- Known RiPP sequence motifs for the motif-ORF strategy

## Outputs

- Target database of putative RiPP precursor peptides (motif-ORF and all-ORF derived)
- Decoy database of shuffled RiPP precursor peptides for target-decoy FDR estimation
- Peptide-spectrum matches scored by modified Dereplicator with MS-DPR p-values at 1% FDR

## How to apply

First 6-frame-translate the input DNA (assembled contigs if starting from short reads) and run HMMer to detect RiPP modification enzymes, delimiting each putative biosynthetic gene cluster as a 10kbp window centered on each detected enzyme. Then build putative precursor peptides with two strategies: the motif-ORF strategy (leveraging similarity to known RiPP leaders/cores) and the all-ORF strategy, which flags all ORFs shorter than a pre-defined threshold (default 200 amino acids) within the gene-cluster windows. Create a decoy database by randomly shuffling each peptide in the target precursor database so that FDR can later be estimated with the target-decoy approach. The resulting target/decoy precursor databases are scored with a modified Dereplicator dynamic-programming search (restricting the number of blind modifications and penalizing matches with more than one modification), with p-values from MS-DPR and a 1% FDR acceptance threshold.

## Related tools

- **MetaMiner** (Command-line pipeline that 6-frame-translates DNA, runs motif-ORF/all-ORF precursor peptide database construction, and builds the shuffled decoy database) — https://github.com/mohimanilab/MetaMiner
- **HMMer** (Detects modification enzymes on translated sequences to delimit the 10kbp gene-cluster windows from which precursor peptides are extracted)
- **Dereplicator** (Modified version used to search spectra against the constructed target/decoy RiPP precursor peptide databases with blind modification scoring)
- **MS-DPR** (Computes p-values of peptide-spectrum matches against the constructed databases)
- **GNPS** (Hosts MetaMiner as the RIPPQUEST workflow on which the precursor database construction is run)

## Examples

```
https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22RIPPQUEST%22%7D
```

## Evaluation signals

- All-ORF mode must flag ORFs shorter than the default 200-amino-acid threshold within the 10kbp gene-cluster windows
- Decoy database must contain exactly one shuffled variant per target precursor peptide (random shuffling of each peptide)
- Positive-control benchmark: searching the STANDARD dataset of 18 purified known RiPPs (prochlorosins, geobacillin, sublancin, haloduracin, lacticin, bicereusins, flavecins) must yield 18/18 hits at 1% FDR
- Peptide-spectrum match p-values must come from MS-DPR and identifications must pass the 1% FDR target-decoy threshold before curation into the results table
- Rationale check: on the paper's data the full pipeline identified 38 RiPPs (31 known, 7 unknown) from 10 million spectra; a much larger yield suggests the FDR filter or database construction is misconfigured

## Limitations

- The motif-ORF strategy is not sensitive and misses peptides without sequence similarity to known RiPPs
- The all-ORF strategy relies on an ORF length threshold (default 200 amino acids), which may miss longer precursor peptides
- No blind modification searches are performed for cyclic peptides due to inherent computational complexity
- Only ~1% of GNPS spectra are accompanied by paired genomic/metagenomic data; without paired sequence data the precursor database cannot be built
- If the producer genome is unavailable (as for lacticin), spectra must be searched against previously described biosynthetic gene clusters rather than a constructed precursor database

## Evidence
