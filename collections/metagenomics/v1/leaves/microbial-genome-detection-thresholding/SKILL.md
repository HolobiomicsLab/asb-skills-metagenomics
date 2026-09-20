---
name: microbial-genome-detection-thresholding
description: 'Use when a mock community with known composition (e.g. ZymoBIOMICS Microbial
  Community Standard #D6300) and matched public reference genomes are available and
  you need to quantify how accurately a tool detects which genomes/strains are present.'
when_to_use_negative:
- No reference genomes matching the community are available — detection is defined
  against public reference genomes (unmaskedBreadth), so the thresholding approach
  cannot be applied without them
- The reference genome set contains near-identical strains that were not dereplicated
  — genome de-replication at 98% ANI is recommended because it reduces multi-mapped
  reads, which would corrupt breadth-based detection
- The goal is de novo strain discovery rather than benchmarking detection of known
  genomes against a ground-truth composition
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: inStrain
  role: Profiles filtered reads per genome (unmaskedBreadth) and computes microdiversity-aware
    popANI pairwise comparisons used to call genome detection
  repo: https://github.com/MrOlm/instrain
- name: Bowtie2
  role: Maps mock-community paired-end reads to the concatenated reference genome
    set with default settings (-X up to 1000 for longer inserts)
- name: Samtools
  role: Required by inStrain for interacting with .bam and .sam files during read
    filtering and coverage calculation
- name: dRep
  role: Competing genome-comparison tool benchmarked on the same reads and references;
    also used to dereplicate genome sets at 98% ANI to reduce multi-mapped reads
  repo: https://github.com/MrOlm/drep
- name: MIDAS
  role: Competing strain-detection tool run on the same Zymo samples and public reference
    genomes for the accuracy comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:f96fc71f172c5a2ed109c2a84c61e608a23951a1d1e81d3e60577602c5e90214
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - Bowtie2
  - MIDAS
  - Samtools
  - dRep
  - inStrain
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# microbial-genome-detection-thresholding

## Summary

Benchmark strain/genome detection accuracy by mapping mock-community reads to known reference genomes, computing breadth-based detection (>= 0.5 unmaskedBreadth per genome), and scoring inStrain's microdiversity-aware popANI comparisons against competing strain-comparison tools.

## When to use

Use when a mock community with known composition (e.g. ZymoBIOMICS Microbial Community Standard #D6300) and matched public reference genomes are available and you need to quantify how accurately a tool detects which genomes/strains are present.

## When NOT to use

- No reference genomes matching the community are available — detection is defined against public reference genomes (unmaskedBreadth), so the thresholding approach cannot be applied without them
- The reference genome set contains near-identical strains that were not dereplicated — genome de-replication at 98% ANI is recommended because it reduces multi-mapped reads, which would corrupt breadth-based detection
- The goal is de novo strain discovery rather than benchmarking detection of known genomes against a ground-truth composition

## Inputs

- Paired-end metagenomic reads from ZymoBIOMICS Microbial Community Standards (Catalog #D6300)
- ZymoBIOMICS.STD.refseq.v2 concatenated reference genome .fasta (https://s3.amazonaws.com/zymo-files/BioPool/ZymoBIOMICS.STD.refseq.v2.zip)
- BAM files from mapping reads to the concatenated reference set (Bowtie2, -X up to 1000)
- Known ground-truth community composition of the mock standard
- inStrain profile objects (IS_profiles) generated per sample

## Outputs

- Per-sample inStrain profiles with coverage, unmaskedBreadth, and SNVs per genome
- Pairwise popANI and coverage-overlap comparisons from 'inStrain compare'
- Detection calls per reference genome at the >= 0.5 unmaskedBreadth threshold
- Supplemental Table S1-format table of raw tool-accuracy values (reproducing Figure 2c)

## How to apply

Map ZymoBIOMICS mock-community paired-end reads to the concatenated ZymoBIOMICS.STD.refseq.v2 reference set with Bowtie2 (default settings; raise the -X flag up to 1000 for longer-insert datasets), then run 'inStrain profile' on each .bam against the reference .fasta with default read filters (paired reads only, proper orientation within expected insert size, minimum read-pair ANI 0.95). Run 'inStrain compare' on all IS_profiles to compute pairwise popANI and coverage overlap between every sample pair. Define detection of each reference genome as >= 0.5 unmaskedBreadth, and run the competing tools dRep, MIDAS, and StrainPhlAn on the same reads and references under the same detection definition. Compile raw per-comparison accuracy values against the known community composition into a Supplemental Table S1-format table reproducing the Figure 2c accuracy comparison; the rationale is that microdiversity-aware population comparison (popANI over ConANI) dramatically increases genomic comparison accuracy.

## Related tools

- **inStrain** (Profiles filtered reads per genome (unmaskedBreadth) and computes microdiversity-aware popANI pairwise comparisons used to call genome detection) — https://github.com/MrOlm/instrain
- **Bowtie2** (Maps mock-community paired-end reads to the concatenated reference genome set with default settings (-X up to 1000 for longer inserts))
- **Samtools** (Required by inStrain for interacting with .bam and .sam files during read filtering and coverage calculation)
- **dRep** (Competing genome-comparison tool benchmarked on the same reads and references; also used to dereplicate genome sets at 98% ANI to reduce multi-mapped reads) — https://github.com/MrOlm/drep
- **MIDAS** (Competing strain-detection tool run on the same Zymo samples and public reference genomes for the accuracy comparison)

## Examples

```
inStrain compare inStrain_profile1 inStrain_profile2
```

## Evaluation signals

- The compiled accuracy table reproduces the tool-accuracy comparison shown in Figure 2c, with inStrain's microdiversity-aware population comparison showing dramatically increased genomic comparison accuracy
- Raw per-comparison values are recorded in Supplemental Table S1 format
- Detection calls respect the definition: a genome is detected only when present with >= 0.5 unmaskedBreadth in a sample
- Accuracy is computed against the known ZymoBIOMICS community composition — false positives on absent genomes and false negatives on present genomes should both decrease relative to dRep/MIDAS/StrainPhlAn
- Each inStrain comparison reports both popANI and coverage overlap between sample pairs

## Limitations

- Raw benchmarking values are deferred to Supplemental Table S1 and are not included in the main text
- Detection is sensitive to reference quality: unmasked breadth depends on the provided reference genomes, so mismatched or non-dereplicated references (multi-mapped reads) skew detection
- Variant sites near the ends of genome fragments can be missed with inStrain
- inStrain's default read filters (paired reads only, proper orientation, expected insert size, minimum read-pair ANI 0.95) may exclude legitimate signal in libraries with unusual insert sizes unless the Bowtie2 -X flag and filters are adjusted

## Evidence
