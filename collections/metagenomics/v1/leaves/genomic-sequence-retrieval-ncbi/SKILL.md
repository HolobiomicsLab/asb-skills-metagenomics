---
name: genomic-sequence-retrieval-ncbi
description: Use when a downstream analysis (e.g. LCS cross-contamination ROC curves
  or Bowtie2 alignment for read recruitment) requires the exact NCBI RefSeq viral
  genomes cited by accession, such as NC_003977.2 or NC_004102.1.
when_to_use_negative:
- The needed reference is not in NCBI RefSeq — e.g. the HPgV partial CDS (10MYKJ037),
  which must be fetched from ViPR instead
- The target is a completely new virus sharing no conservation with any reference
  genome — retrieval-based reference use will not detect it
- Reads or contigs are already aligned and only the SAM file is needed as TAR-VIR
  input — no further reference retrieval is required
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NCBI viral genome database
  role: Source of RefSeq records for HBV, HCV genotypes, HPgV, and other viral/microbial
    genomes used in the LCS cutoff analysis and read mapping
- name: ViPR
  role: Alternative database for non-RefSeq references, e.g. the HPgV partial CDS
    10MYKJ037 used in the real-data analysis
- name: TAR-VIR
  role: 'Downstream consumer of the retrieved references: genomes support Bowtie2
    alignment and the overlap-extension read recruitment stage'
  repo: https://github.com/chjiao/TAR-VIR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:6983e5be0f1ab1c953068f5478559eb5221ebe8a832c4b3e3eb2d0f1579053e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/TAR-VIR
  tools:
  - NCBI viral genome database
  - Python
  - TAR-VIR
  - ViPR
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# genomic-sequence-retrieval-ncbi

## Summary

Retrieve RefSeq viral reference genomes (e.g. HBV NC_003977.2, seven HCV genotype records, HPgV NC_001710.1) from the NCBI genome/viral database by accession, so that pairwise longest-common-substring (LCS) analyses and read-mapping experiments in TAR-VIR run on the exact reference sequences the paper used.

## When to use

Use when a downstream analysis (e.g. LCS cross-contamination ROC curves or Bowtie2 alignment for read recruitment) requires the exact NCBI RefSeq viral genomes cited by accession, such as NC_003977.2 or NC_004102.1.

## When NOT to use

- The needed reference is not in NCBI RefSeq — e.g. the HPgV partial CDS (10MYKJ037), which must be fetched from ViPR instead
- The target is a completely new virus sharing no conservation with any reference genome — retrieval-based reference use will not detect it
- Reads or contigs are already aligned and only the SAM file is needed as TAR-VIR input — no further reference retrieval is required

## Inputs

- NCBI accession numbers for viral RefSeq genomes (e.g. NC_003977.2, NC_004718.3, NC_004102.1, NC_001710.1)
- ViPR record identifier for non-NCBI references (e.g. 10MYKJ037, HPgV partial CDS from Malaysia 2010)

## Outputs

- FASTA files of reference genomes keyed by accession
- Local reference set suitable for pairwise LCS computation and Bowtie2 index building

## How to apply

Collect the accession numbers named in the analysis plan (for the TAR-VIR study: HBV NC_003977.2; HCV genotypes NC_009827.1, NC_009823.1, NC_009825.1, NC_030791.1, NC_004102.1, NC_009826.1, NC_009824.1; HPgV NC_001710.1; plus viral/microbial RefSeq records for the ~142 million virus-vs-other comparisons) and download each record from the NCBI viral genome database in FASTA form. Verify that each downloaded FASTA resolves to the requested accession and that sequences for cross-species pairs (e.g. Vaccinia vs Variola, yielding the 469 bp LCS) are complete enough for suffix-array/dynamic-programming LCS computation. Note that not all references come from NCBI: the HPgV partial CDS used in the HIV-plasma analysis (10MYKJ037) was instead downloaded from ViPR, so route non-RefSeq references accordingly. These genomes then feed two downstream uses: LCS/ROC cutoff justification (100 bp overlap threshold) and Bowtie2 alignment of simulated or real metagenomic reads for TAR-VIR read recruitment.

## Related tools

- **NCBI viral genome database** (Source of RefSeq records for HBV, HCV genotypes, HPgV, and other viral/microbial genomes used in the LCS cutoff analysis and read mapping)
- **ViPR** (Alternative database for non-RefSeq references, e.g. the HPgV partial CDS 10MYKJ037 used in the real-data analysis)
- **TAR-VIR** (Downstream consumer of the retrieved references: genomes support Bowtie2 alignment and the overlap-extension read recruitment stage) — https://github.com/chjiao/TAR-VIR

## Evaluation signals

- Each retrieved FASTA header resolves to the requested accession (e.g. NC_003977.2 for HBV, NC_001710.1 for HPgV)
- All seven HCV genotype records named in the paper (NC_009827.1, NC_009823.1, NC_009825.1, NC_030791.1, NC_004102.1, NC_009826.1, NC_009824.1) are present in the local reference set
- Recomputed Vaccinia-Variola LCS equals 469 bp and HCV genotype 7 vs genotype 5 equals 154 bp on the retrieved sequences
- Only a small number of virus-vs-other-microbe pairs (e.g. 86 of ~142 million) exceed the 100 bp LCS threshold, matching the paper's cutoff justification

## Limitations

- A completely new virus that does not share any conservation with any reference genome won't be detected by reference-based methods downstream
- Some references cited in the paper are partial CDS records from ViPR rather than full NCBI genomes, so lengths and completeness differ across references
- The paper does not state software version numbers or a recommended overlap threshold value in the discussion; cutoff choice must be re-derived from the LCS/ROC analysis

## Evidence

- [results] results: "We downloaded the reference genomes of HBV (NC_003977.2), multiple genotypes of HCV (NC_009827.1, NC_009823.1, NC_009825.1, NC_030791.1, NC_004102.1, NC_009826.1, NC_009824.1), and human pegivirus"
- [readme] readme: "git clone --recursive  https://github.com/chjiao/TAR-VIR.git"
