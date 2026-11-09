---
name: short-read-simulation-and-mapping
description: Use when building simulated benchmark read sets for the B. burgdorferi
  MLST deconvolution pipeline (ADP/SDP, EvoMod mechanisms, coverage/edit-distance
  sweeps) or when reads must be mapped to a known allele database to produce ILP input
  files.
when_to_use_negative:
- When the target organism or MLST scheme is not B. burgdorferi — the pipeline, allele
  database, and edit-distance matrices are Borrelia-specific
- When real sequencing data is already available and mapped; this skill is for simulation/benchmarking,
  not for the real-sample analysis path
- When a modern aligner (e.g. Bowtie2) is required for other purposes — the article's
  pipeline is pinned to Bowtie v0.12.7 and its SAM output format
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0085
tools:
- name: ART
  role: Simulates Illumina reads from allele/strain mixtures, following the read characteristics
    of the real tick-sample dataset
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- name: Bowtie v0.12.7
  role: Aligns simulated (and real) reads to the B. burgdorferi allele database via
    bowtie and bowtie-build
  repo: https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- name: samtools v1.3.1
  role: Extracts per-read mapping information from SAM files into ILP input text files
  repo: https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- name: fastq-dump (SRA toolkit)
  role: Downloads the real 24-tick-sample reads (v2.8.2) whose characteristics guide
    ART simulation
  repo: https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/
- name: WGS-TB/MLST BorreliaPipeline
  role: Provides run_sim.py (allele simulation) and mapSamples.py (mapping/extraction)
    driver scripts
  repo: https://github.com/WGS-TB/MLST
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:67a66be15201ddf670133a6e5d44faed9efc2e5733bae68e4c3133eeea306cf5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - Bowtie v0.12.7
  - GAML
  - Kallisto
  - PyEMD
  - Python 2.7
  - WGS-TB/MLST BorreliaPipeline
  - editdistance
  - fastq-dump (SRA toolkit)
  - samtools v1.3.1
  - strainEST
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# short-read-simulation-and-mapping

## Summary

Simulate Illumina-style NGS reads from known B. burgdorferi MLST alleles or strain mixtures with ART, then map them to the 8-gene/1726-allele database with Bowtie v0.12.7 and extract per-read mapping evidence with samtools to feed the allele and strain diversity ILPs.

## When to use

Use when building simulated benchmark read sets for the B. burgdorferi MLST deconvolution pipeline (ADP/SDP, EvoMod mechanisms, coverage/edit-distance sweeps) or when reads must be mapped to a known allele database to produce ILP input files.

## When NOT to use

- When the target organism or MLST scheme is not B. burgdorferi — the pipeline, allele database, and edit-distance matrices are Borrelia-specific
- When real sequencing data is already available and mapped; this skill is for simulation/benchmarking, not for the real-sample analysis path
- When a modern aligner (e.g. Bowtie2) is required for other purposes — the article's pipeline is pinned to Bowtie v0.12.7 and its SAM output format

## Inputs

- B. burgdorferi MLST allele database (8 housekeeping genes: clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA; 1726 alleles)
- Randomly drawn strain-type distributions per locus (EvoMod1/EvoMod2/EvoMod2e/EvoMod2n/EvoMod3 mechanisms, proportions summing to 1)
- Simulation parameters: coverage c∈{30,100,300}, inter-allele edit distance d∈{5,10,15,20,25}, number of iterations (default 40)
- Real-dataset read characteristics from the 24 tick samples for ART profile selection

## Outputs

- ART-simulated paired-end FASTQ read sets per sample/locus
- SAM/BAM alignment files from Bowtie v0.12.7
- Per-sample, per-gene mapping-information text files (sampleX_geneY_paired_reads.txt) consumed by the alleleDiversity.py ILP stage

## How to apply

Generate random strain-type/allele distributions per locus under the chosen EvoMod mechanism (e.g. EvoMod1: one existing plus one novel strain via m=2 mutations with allele edit distance at most d=15; EvoMod3 adds an allele-exchange recombination), then simulate reads with ART following the read characteristics of the real 24-sample tick dataset. Align the simulated FASTQ reads to the B. burgdorferi allele database (8 housekeeping genes, 1726 alleles, 753 strain types) using Bowtie v0.12.7 (bowtie and bowtie-build), varying coverage c∈{30,100,300} and edit distance d∈{5,10,15,20,25} for ADP benchmarks with ~40 iterations per parameter combination. Extract mapping statistics from the resulting SAM files with samtools v1.3.1 into per-sample, per-gene text files (sampleX_geneY_paired_reads.txt) containing the per-read evidence needed by the first-stage Allele Diversity ILP. The rationale is that benchmarks must match real-data read profiles so downstream precision/recall, TVD, EMD, and soft-precision/soft-recall evaluations reflect true pipeline behavior.

## Related tools

- **ART** (Simulates Illumina reads from allele/strain mixtures, following the read characteristics of the real tick-sample dataset) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- **Bowtie v0.12.7** (Aligns simulated (and real) reads to the B. burgdorferi allele database via bowtie and bowtie-build) — https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- **samtools v1.3.1** (Extracts per-read mapping information from SAM files into ILP input text files) — https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- **fastq-dump (SRA toolkit)** (Downloads the real 24-tick-sample reads (v2.8.2) whose characteristics guide ART simulation) — https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/
- **WGS-TB/MLST BorreliaPipeline** (Provides run_sim.py (allele simulation) and mapSamples.py (mapping/extraction) driver scripts) — https://github.com/WGS-TB/MLST

## Examples

```
python run_sim.py -i 40 -c 30 -e 15 -b ../bowtieFolder/ -s ../../samtools -a art_illumina
```

## Evaluation signals

- Each simulated condition yields per-sample folders in variantsAndProp containing 8 sampleX_geneY_paired_reads.txt files (one per MLST locus)
- ADP benchmarks reproduce published precision ~0.92-0.96 and recall ~0.90-0.92 per locus; per-locus precision exceeds Kallisto (e.g. clpA 0.99±0.009 vs 0.97±0.014)
- Simulated strain proportions sum to 1 at every locus and match the drawn EvoMod distribution when re-estimated (low TVD/EMD)
- Full-pipeline SDP results reproduce the Table 2 pattern: exact precision/recall drop from ~0.8 to ~0.3-0.5 and TVD rises to ~0.6-0.8 when ADP-inferred (rather than perfect) allele calls feed the SDP

## Limitations

- Pipeline is Python 2.7-based; PyEMD explicitly requires Python 2 (Python 3 not supported), constraining the execution environment
- Closely related strain clusters must be separated by ad-hoc post-processing rather than a principled method
- ART must be configured to the real dataset's read characteristics; results may not transfer to different read length/error profiles

## Evidence

- [methods] ART simulation follows real read characteristics: "All reads are simulated using ART [18], following the characteristics of the reads from the real data set"
- [methods] Bowtie version used for alignment: "To align the simulated reads to the alleles of the database, we used Bowtie v0.12.7 [19]."
- [methods] MLST database composition: "the traditional B. burgdorferi MLST scheme [11] composed of 8 housekeeping genes having a combined total of 1726 known alleles"
- [readme] README instructions for mapping and extraction: "`python mapSamples.py [-h] [-c NUMOFCORES] [-b BOWTIE] [-s SAMTOOLS]` to map the sample reads. `-c` specifies the number of core for running Bowtie, default is 4."
- [readme] README instructions for simulation runs: "`python run_sim.py [-h] [-i NUMOFITER] [-f SIMULATIONRESULTFOLDER] [-c COVERAGE] [-e EDITDIST] [-fp FULLPAPER] [-b BOWTIE] [-s SAMTOOLS] [-a ART]`"
- [abstract] Pipeline accuracy on simulated data: "Our method performs accurately on simulated data and generates results on a real data set of Borrelia burgdorferi genomes"
