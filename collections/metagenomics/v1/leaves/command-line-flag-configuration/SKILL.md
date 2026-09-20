---
name: command-line-flag-configuration
description: Use when running CasCollect on raw Illumina FASTQ reads and you must
  decide which flags to enable (e.g. --trim, --clean, --noprot, --nucl, --seed -define)
  and set parameters like -match and -cycle for a targeted CRISPR/Cas or custom gene-set
  run.
when_to_use_negative:
- Input is already assembled contigs or complete genomes — CasCollect is designed
  for raw, unassembled short-read sequencing datasets
- Only full-length long reads (e.g. PacBio/ONT) are available — the pipeline targets
  short Illumina reads translated/identity-matched by FragGeneScan and VSEARCH
- A complete de novo genome assembly and annotation is required rather than targeted
  recovery of CRISPR/Cas regions or a specialty gene set
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CasCollect
  role: Python-wrapped pipeline whose command-line flags select and parameterize the
    five optional steps (filtering, seed generation, expansion, assembly, annotation)
  repo: https://github.com/sandialabs/CasCollect
- name: VSEARCH
  role: performs DNA-mode identity matching configured via -query and the -match percent
    threshold (default 95%)
  repo: https://github.com/torognes/vsearch
- name: HMMER
  role: executes protein-mode seed search against profile HMMs from the default 120
    Cas HMM set or a user -hmm directory
- name: BBTools
  role: bbduk implements the --trim/--clean flag options (quality trim, adapter removal,
    contaminant removal) and bbmerge merges overlapping reads
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:64c0eee33f8ea527c6693ba724faa8ef72c83b1d88ef998791a6fa165dfd853f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - BBTools
  - CasCollect
  - FragGeneScanPlus
  - HMMER
  - VSEARCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# command-line-flag-configuration

## Summary

Configure CasCollect's command-line flags to select and parameterize its five optional pipeline steps (read filtering/cleaning, seed generation in protein, DNA or user-defined modes, read subset expansion, assembly and annotation), including mode constraints such as --nucl requiring -query and --noprot pairing with -hmm.

## When to use

Use when running CasCollect on raw Illumina FASTQ reads and you must decide which flags to enable (e.g. --trim, --clean, --noprot, --nucl, --seed -define) and set parameters like -match and -cycle for a targeted CRISPR/Cas or custom gene-set run.

## When NOT to use

- Input is already assembled contigs or complete genomes — CasCollect is designed for raw, unassembled short-read sequencing datasets
- Only full-length long reads (e.g. PacBio/ONT) are available — the pipeline targets short Illumina reads translated/identity-matched by FragGeneScan and VSEARCH
- A complete de novo genome assembly and annotation is required rather than targeted recovery of CRISPR/Cas regions or a specialty gene set

## Inputs

- raw unassembled Illumina short-read FASTQ files (paired-end)
- user-defined nucleotide query FASTA of CRISPR or other repeat sequences (for --nucl -query)
- user-defined seed FASTA (for --seed -define)
- user-defined profile HMM directory (for --noprot -hmm)
- user-defined contaminant reference FASTA (for --clean -ref; default human genome)

## Outputs

- seed_reads.fastq combining seeds from all enabled seed-generation modes
- expanded read subset for assembly
- SPAdes/metaSPAdes targeted assembly contigs
- CRISPRCasFinder annotation of cas operons and CRISPR arrays

## How to apply

First parse the desired analysis goals against CasCollect's optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation. For trimming/cleaning choose --trim (quality trim, adapter removal, merging overlapping reads) or --clean, supplying -ref file.fasta to remove user-defined contaminant sequences (default reference is the human genome). For seed generation, protein mode is default and scans reads with HMMER against the default 120 Cas protein family profile HMMs (override with -hmm directory, used with --noprot); DNA mode requires --nucl with -query file.fasta for VSEARCH identity matching against CRISPR repeat FASTAs (CasCollect ships 105 repeat sequences); user-defined mode uses --seed -define file.fasta to take user FASTA entries directly as seeds, bypassing both searches. Tune read subset expansion with -match (default 95% identity for vsearch) and -cycle (default five expansion cycles, with matched reads removed from the database between cycles to eliminate redundant searches). Enforce the documented constraints — --nucl requires -query, and --noprot pairs with -hmm — before execution; the pipeline checks for required software (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) in the user's path, or Check.py can detect and auto-download missing pieces.

## Related tools

- **CasCollect** (Python-wrapped pipeline whose command-line flags select and parameterize the five optional steps (filtering, seed generation, expansion, assembly, annotation)) — https://github.com/sandialabs/CasCollect
- **VSEARCH** (performs DNA-mode identity matching configured via -query and the -match percent threshold (default 95%)) — https://github.com/torognes/vsearch
- **HMMER** (executes protein-mode seed search against profile HMMs from the default 120 Cas HMM set or a user -hmm directory)
- **BBTools** (bbduk implements the --trim/--clean flag options (quality trim, adapter removal, contaminant removal) and bbmerge merges overlapping reads) — https://jgi.doe.gov/data-and-tools/bbtools/

## Evaluation signals

- The command exits without error for every enabled flag combination and the enforced constraints hold (--nucl requires -query; --noprot pairs with -hmm for user-defined HMM directories)
- seed_reads.fastq exists, is a valid FASTQ/FASTA of read sequences, and is non-empty only when the corresponding search returned matches
- Expansion behavior matches configured parameters: default 95% identity (-match) and five cycles (-cycle), with matched reads removed from the database between cycles
- Check.py (or the pre-run check) reports all required software (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) present in the user's path
- With --clean -ref, reads matching the contaminant reference (default human genome) are absent from downstream seed and expansion outputs

## Limitations

- The modes are not benchmarked in isolation; validation is limited to the module executing without error and producing a valid, conditionally non-empty seed_reads.fastq
- Mode constraints must be respected: --nucl requires -query and --noprot pairs with -hmm; mispairing flags is undefined
- --seed -define bypasses both protein and DNA searches entirely, so HMM/VSEARCH-based discovery is skipped when this mode is enabled
- No software version numbers or accession IDs are given for CasCollect or its third-party dependencies; the contaminant reference defaults to the human genome and must be overridden with -ref for non-human samples

## Evidence

- [methods] seed mode constraints: "-ref file.fasta     user defined contaminant reference genome/sequences [default: human genome]"
- [methods] expansion match threshold: "-match number       percent match for vsearch"
- [results] expansion defaults: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] cycle default: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [readme] trim flag semantics: "--trim              runs quality trim, remove adapters, merge overlapping
                      reads"
- [results] pipeline step structure: "The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation"
