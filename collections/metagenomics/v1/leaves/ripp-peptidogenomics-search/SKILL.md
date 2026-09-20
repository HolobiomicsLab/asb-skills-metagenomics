---
name: ripp-peptidogenomics-search
description: Use when paired mass spectrometry spectra and genome or metagenome assemblies
  are available and the goal is RiPP discovery by searching spectra against precursor
  peptide databases, e.g. ACTI/GNPS datasets like MSV000078839.
when_to_use_negative:
- The mass spectra are not accompanied by paired genomics/metagenomics data — per
  the article, about 99% of GNPS spectra cannot be searched this way
- The target compounds are non-ribosomal peptides; searching spectral families against
  known structures with Dereplicator+ is the appropriate route
- Blind modification search is needed for cyclic peptides — MetaMiner does not currently
  perform blind modification searches for cyclic peptides due to computational complexity
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0622
tools:
- name: MetaMiner
  role: 'Core tool: builds precursor databases (motif-ORF/all-ORF) and searches spectra
    with modified Dereplicator scoring; available as a command line tool and a GNPS
    workflow'
  repo: https://github.com/mohimanilab/MetaMiner
- name: HMMer
  role: Detects RiPP modification enzymes on translated ORF sequences to define 10kbp
    biosynthetic gene cluster windows
- name: Dereplicator
  role: Modified version used by MetaMiner for searching spectra against the target/decoy
    RiPP database
- name: MS-DPR
  role: Computes p-values of peptide-spectrum matches used for FDR estimation
- name: GNPS (Global Natural Products Social molecular networking)
  role: Hosts the MetaMiner workflow and the molecular networking/spectral data being
    searched
- name: antiSMASH
  role: Genome mining of biosynthetic gene clusters used for context (e.g. identifying
    RiPP-encoding clusters in reference genomes)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:780ea99fda967c42cc8fa2edac05f8e0752f967e0763f624263534b0119efbb6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - GNPS (Global Natural Products Social molecular networking)
  - HMMer
  - MS-DPR
  - MetaMiner
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# ripp-peptidogenomics-search

## Summary

A peptidogenomics workflow (MetaMiner-style) that builds putative RiPP precursor databases from genome/metagenome sequences and searches GNPS tandem mass spectra against them with target-decoy FDR estimation, enabling discovery of ribosomal peptide natural products with blind modifications.

## When to use

Use when paired mass spectrometry spectra and genome or metagenome assemblies are available and the goal is RiPP discovery by searching spectra against precursor peptide databases, e.g. ACTI/GNPS datasets like MSV000078839.

## When NOT to use

- The mass spectra are not accompanied by paired genomics/metagenomics data — per the article, about 99% of GNPS spectra cannot be searched this way
- The target compounds are non-ribosomal peptides; searching spectral families against known structures with Dereplicator+ is the appropriate route
- Blind modification search is needed for cyclic peptides — MetaMiner does not currently perform blind modification searches for cyclic peptides due to computational complexity

## Inputs

- GNPS tandem MS/MS spectra (e.g. datasets MSV000078839, MSV000078604)
- RefSeq genome sequences (e.g. 36 Actinomycetes genomes paired with ACTI spectra)
- Metagenomic short reads for assembly into contigs
- Known RiPP motif/sequence information for the motif-ORF precursor database

## Outputs

- Identified known and unknown RiPPs per genome-mining mode and FDR threshold (e.g. acti_ripp_mode_fdr_comparison.csv)
- Peptide-spectrum matches with p-values (MS-DPR) and target-decoy FDR estimates at 0%/1%
- Spectral network families enlarging RiPP identifications

## How to apply

Retrieve the spectra (e.g. GNPS datasets MSV000078839 and MSV000078604) together with paired RefSeq genomes or metagenomic short reads. Assemble short reads into contigs, then 6-frame translate the DNA and run HMMer to detect modification enzymes, constructing a 10kbp putative biosynthetic gene cluster window centered at each enzyme; build precursor databases with both motif-ORF and all-ORF strategies (all-ORF enumerates ORFs shorter than the default 200 amino acid threshold). Search spectra against the target database and a decoy database created by random shuffling of each precursor peptide, using MetaMiner's modified Dereplicator scoring, and compute peptide-spectrum match p-values with MS-DPR. Estimate FDR via the target-decoy approach (0% and 1% thresholds) and enlarge identifications via spectral networking. Choose motif-ORF when known RiPP similarity is expected; note that motif-ORF misses peptides without sequence similarity to known RiPPs, so compare against all-ORF for unknown RiPP discovery.

## Related tools

- **MetaMiner** (Core tool: builds precursor databases (motif-ORF/all-ORF) and searches spectra with modified Dereplicator scoring; available as a command line tool and a GNPS workflow) — https://github.com/mohimanilab/MetaMiner
- **HMMer** (Detects RiPP modification enzymes on translated ORF sequences to define 10kbp biosynthetic gene cluster windows)
- **Dereplicator** (Modified version used by MetaMiner for searching spectra against the target/decoy RiPP database)
- **MS-DPR** (Computes p-values of peptide-spectrum matches used for FDR estimation)
- **GNPS (Global Natural Products Social molecular networking)** (Hosts the MetaMiner workflow and the molecular networking/spectral data being searched)
- **antiSMASH** (Genome mining of biosynthetic gene clusters used for context (e.g. identifying RiPP-encoding clusters in reference genomes))

## Examples

```
git clone https://github.com/mohimanilab/MetaMiner && python MetaMiner.py --spectra MSV000078839 --reads simulated_metagenome.fastq --mode motif-ORF --fdr 0.01
```

## Evaluation signals

- On the ACTI dataset at 0% FDR, motif-ORF mode must yield 3 unknown plus 5 known RiPPs while all-ORF mode yields only 2 known RiPPs; at 1% FDR, 8 RiPPs total are identified in this dataset
- Decoy database must be generated by random shuffling of each target precursor peptide and used for target-decoy FDR estimation at the 1% (or stricter 0%) threshold
- Each putative biosynthetic gene cluster window must be a 10kbp window centered at an HMMer-detected modification enzyme
- In the all-ORF mode, enumerated ORFs must all be shorter than the default 200 amino acid threshold
- Positive-control datasets should recover known RiPPs (MetaMiner identified all 18 known RiPPs in its positive control dataset)

## Limitations

- The motif-ORF strategy is not sensitive — peptides without sequence similarity to known RiPPs are missed
- Only ~1% of GNPS spectra have paired genomics/metagenomics data and can be searched
- No blind modification searches are performed for cyclic peptides due to computational complexity
- The number of modifications is restricted and high-score matches with more than one modification are penalized by the dynamic programming search

## Evidence

- [readme] MetaMiner is currently available as a workflow at the Global Natural Products Social Molecular Networking (GNPS).: [evidence span withheld — anchor status: not_found]
