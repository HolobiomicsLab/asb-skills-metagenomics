---
name: ncbi-sra-dataset-retrieval
description: Use when reproducing the StrainXpress benchmark or any strain-aware metagenome
  assembly comparison that needs the original Bmock12 (SRX4901583), NWCs, or Gut Metagenome
  (PRJNA379884) short-read FASTQ inputs.
when_to_use_negative:
- Input is long-read (PacBio/ONT) data — StrainXpress as published targets Illumina-type
  short reads, and the paper notes third-generation read error rates remain unresolved
- A reference-based or strain-typing analysis is the goal; this skill only supplies
  raw de novo assembly inputs, not reference genomes
- The target study has no deposited SRA accessions in the source card (only Bmock12,
  NWCs and Gut Metagenome accessions are grounded here)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3174
tools:
- name: StrainXpress
  role: Downstream consumer of the retrieved interleaved FASTQ reads for strain-aware
    assembly benchmarking
  repo: https://github.com/HaploKit/StrainXpress
- name: SPAdes
  role: Competing assembler that will be run on the same retrieved read sets
- name: IDBA-UD
  role: Competing assembler that will be run on the same retrieved read sets
- name: Minia3
  role: Competing assembler that will be run on the same retrieved read sets
- name: MEGAHIT
  role: Competing assembler that will be run on the same retrieved read sets
- name: Quast
  role: Downstream evaluator of assemblies produced from the retrieved reads
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:59a74dd5743286bd62ca0892be20ca20a1446a3fd1451d449dd3f75cb7a81e4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - IDBA-UD
  - MEGAHIT
  - Minia3
  - Quast
  - SPAdes
  - StrainXpress
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# ncbi-sra-dataset-retrieval

## Summary

Retrieve the raw interleaved FASTQ read sets underlying the StrainXpress real-data benchmark (Bmock12 mock community, natural whey starter cultures, and 5 Gut Metagenome patient samples) from NCBI SRA/ENA using the accessions reported in the paper, so the benchmark can be reproduced end-to-end.

## When to use

Use when reproducing the StrainXpress benchmark or any strain-aware metagenome assembly comparison that needs the original Bmock12 (SRX4901583), NWCs, or Gut Metagenome (PRJNA379884) short-read FASTQ inputs.

## When NOT to use

- Input is long-read (PacBio/ONT) data — StrainXpress as published targets Illumina-type short reads, and the paper notes third-generation read error rates remain unresolved
- A reference-based or strain-typing analysis is the goal; this skill only supplies raw de novo assembly inputs, not reference genomes
- The target study has no deposited SRA accessions in the source card (only Bmock12, NWCs and Gut Metagenome accessions are grounded here)

## Inputs

- NCBI accession identifiers (e.g. SRX4901583 for Bmock12, PRJNA379884 for Gut Metagenome)
- NCBI SRA/ENA download URLs corresponding to those accessions

## Outputs

- Interleaved FASTQ read files for Bmock12 (11 strains, 9 species)
- Interleaved FASTQ read files for the NWCs samples
- Interleaved FASTQ read files for each of the 5 Gut Metagenome patient datasets
- A per-dataset directory layout feeding downstream assembly (real_data_benchmark.csv inputs)

## How to apply

Look up the study/sample accessions cited in the paper — SRX4901583 for the Bmock12 mock community and PRJNA379884 for the 5 Gut Metagenome patient samples — and download the read files from NCBI SRA (or ENA mirrors), converting to interleaved FASTQ with read pairs labeled /1 and /2, since StrainXpress requires interleaved FASTQ input. The NWCs (natural whey starter cultures) reads, with average strain coverage 25.47× and a minimum of 7.46× for one Lactobacillus helveticus strain, are retrieved from the same accession chain reported in the paper. Verify each download by checking read counts and that the FASTQ interleaving matches the format StrainXpress documents (e.g. @S0R0/1 followed by @S0R0/2). Organize the downloaded files per dataset so the five Gut Metagenome sets can later be assembled individually against the StrainEst-derived E. coli ground truth.

## Related tools

- **StrainXpress** (Downstream consumer of the retrieved interleaved FASTQ reads for strain-aware assembly benchmarking) — https://github.com/HaploKit/StrainXpress
- **SPAdes** (Competing assembler that will be run on the same retrieved read sets)
- **IDBA-UD** (Competing assembler that will be run on the same retrieved read sets)
- **Minia3** (Competing assembler that will be run on the same retrieved read sets)
- **MEGAHIT** (Competing assembler that will be run on the same retrieved read sets)
- **Quast** (Downstream evaluator of assemblies produced from the retrieved reads)

## Examples

```
prefetch SRX4901583 && fastq-dump --split-files SRR*.sra
```

## Evaluation signals

- Each downloaded FASTQ is interleaved with matching /1 and /2 read headers per pair, matching the StrainXpress README example format
- The Bmock12 download corresponds to accession SRX4901583 and contains reads for 11 strains from 9 species
- Five distinct Gut Metagenome read files are obtained under PRJNA379884, one per patient dataset (11 E. coli strains overall)
- NWCs read depth is consistent with the paper's reported average strain coverage of 25.47× (min 7.46× for a Lactobacillus helveticus strain)
- Assemblies run on the retrieved reads reproduce Quast genome fraction ≥ ~95% for StrainXpress on Bmock12 (paper: approaching 99.04%, ≥3.7% above competitors)

## Limitations

- The Gut Metagenome ground truth must be derived separately (StrainEst-based); the SRA retrieval alone does not provide strain-level labels
- No software version numbers for tools or explicit SRA file formats are stated in the paper, so exact release pinning is a reproducibility gap
- Retrieval covers only the three real datasets surfaced by the source cards; synthetic and spike-in Salmonella datasets need separate accessions not given here

## Evidence

- [methods] Bmock12 and Gut Metagenome accessions ground the retrieval: [evidence span withheld — anchor status: not_found]
- [results] Dataset composition of Bmock12 and NWCs: "the ‘Bmock12’ data set contains 11 strains from 9 species, so is of rather low complexity"
- [results] NWCs coverage detail validates the downloaded read depths: "the average coverage of strains was 25.47×, at a minimum of 7.46× for one of the Lactobacillus helveticus strains"
- [results] Gut Metagenome structure: 5 datasets, 11 E. coli strains: "‘Gut Metagenome’ consists of 5 data set that refer to 11 E. coli strains overall"
- [readme] Interleaved FASTQ input requirement from README: "The input file must be interleaved FASTQ and format like below:"
