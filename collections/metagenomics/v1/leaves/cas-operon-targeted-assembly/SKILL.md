---
name: cas-operon-targeted-assembly
description: Use when raw paired-end Illumina FASTQ reads (isolates or metagenomes)
  must be screened for CRISPR arrays and cas operons without performing a complete
  de novo genome assembly, e.g. for phage-susceptibility assessment.
when_to_use_negative:
- The question requires whole-genome de novo assembly or genome-wide annotation beyond
  CRISPR/Cas loci—CasCollect only assembles genomic regions seeded by cas HMM matches
  or CRISPR repeats.
- The input is already assembled contigs rather than raw short reads; run CRISPRCasFinder
  (or similar) directly on the assemblies instead.
- Reads are not short paired-end Illumina data (the pipeline is built around bbduk/bbmerge
  trimming/merging and SPAdes short-read assembly).
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: CasCollect
  role: Python/Perl wrapper orchestrating the five-step pipeline (read filtering,
    seed generation, read subset expansion, assembly, annotation)
  repo: https://github.com/sandialabs/CasCollect
- name: HMMER
  role: searches read ORFs against the default 120 Cas protein family profile HMMs
    to generate seed reads (protein mode)
- name: VSEARCH
  role: identity matching in DNA mode against user-defined CRISPR repeats and read
    subset expansion at the default 95% match
  repo: https://torognes.github.io/vsearch/
- name: SPAdes
  role: assembles the seed+expanded read subset (metaSPAdes for metagenomes)
- name: CRISPRCasFinder
  role: annotates targeted assembly contigs to identify CRISPR arrays and cas genes
- name: bbduk
  role: quality trimming, adapter removal and read merging via bbmerge when --trim/--clean
    is set
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: Seqtk
  role: sequence utility dependency checked/installed by CasCollect
  repo: https://github.com/lh3/seqtk
- name: FragGeneScan
  role: short-read open reading frame prediction dependency for protein-mode seed
    generation
  repo: https://github.com/hallamlab/FragGeneScanPlus
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:7326139e0fc9e5c83af9a6f254556ef90e1fccb019c4f7dce94ec83464afe445
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CRISPRCasFinder
  - CasCollect
  - FragGeneScan
  - HMMER
  - SPAdes
  - Seqtk
  - VSEARCH
  - bbduk
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cas-operon-targeted-assembly

## Summary

A targeted assembly pipeline (CasCollect) that detects cas genes and CRISPR arrays directly from unassembled Illumina reads by seeding on cas protein HMM matches and/or CRISPR repeat sequences, expanding read subsets with VSEARCH, then assembling and annotating with SPAdes and CRISPRCasFinder—avoiding complete genome assembly.

## When to use

Use when raw paired-end Illumina FASTQ reads (isolates or metagenomes) must be screened for CRISPR arrays and cas operons without performing a complete de novo genome assembly, e.g. for phage-susceptibility assessment.

## When NOT to use

- The question requires whole-genome de novo assembly or genome-wide annotation beyond CRISPR/Cas loci—CasCollect only assembles genomic regions seeded by cas HMM matches or CRISPR repeats.
- The input is already assembled contigs rather than raw short reads; run CRISPRCasFinder (or similar) directly on the assemblies instead.
- Reads are not short paired-end Illumina data (the pipeline is built around bbduk/bbmerge trimming/merging and SPAdes short-read assembly).

## Inputs

- paired-end Illumina genomic FASTQ reads (isolate or metagenomic)
- user-defined CRISPR repeat FASTA (for DNA mode via --nucl -query); 105 repeat sequences included with CasCollect
- default set of 120 Cas protein family profile HMMs (protein mode)
- optional contaminant reference FASTA for --clean (-ref, default: human genome)

## Outputs

- targeted SPAdes assemblies of CRISPR/Cas loci contigs
- CRISPRCasFinder annotations of CRISPR arrays and cas protein genes per dataset, written to the -out folder
- seed and expanded read subset FASTQs per expansion cycle

## How to apply

Run CasCollect.py with -fwd/-rev/-out pointing to paired FASTQ inputs and an output folder, typically with --trim so bbduk/bbmerge quality-trim reads, remove adapters, and merge overlaps. In default protein mode, HMMER searches reads against 120 Cas protein family profile HMMs to identify seed reads; alternatively enable DNA mode with --nucl and -query to seed on user-provided CRISPR repeat FASTA sequences (105 repeats ship with the tool). Expand the seed subset with VSEARCH identity matching at the default 95% (-match) over the default five cycles, override with -cycle (e.g. -cycle 2 suffices, reaching 96.1–99.5% operon coverage on the ground water metagenome), with previously matched reads removed between cycles to eliminate redundant searches. Assemble expanded subsets with SPAdes (metaSPAdes for metagenomes) and annotate contigs with CRISPRCasFinder, applying the 500 bp contig cutoff convention; for metagenomes and clinical isolate panels, validate that cas operons and CRISPR arrays are recovered and agree with complete-assembly results.

## Related tools

- **CasCollect** (Python/Perl wrapper orchestrating the five-step pipeline (read filtering, seed generation, read subset expansion, assembly, annotation)) — https://github.com/sandialabs/CasCollect
- **HMMER** (searches read ORFs against the default 120 Cas protein family profile HMMs to generate seed reads (protein mode))
- **VSEARCH** (identity matching in DNA mode against user-defined CRISPR repeats and read subset expansion at the default 95% match) — https://torognes.github.io/vsearch/
- **SPAdes** (assembles the seed+expanded read subset (metaSPAdes for metagenomes))
- **CRISPRCasFinder** (annotates targeted assembly contigs to identify CRISPR arrays and cas genes)
- **bbduk** (quality trimming, adapter removal and read merging via bbmerge when --trim/--clean is set) — https://jgi.doe.gov/data-and-tools/bbtools/
- **Seqtk** (sequence utility dependency checked/installed by CasCollect) — https://github.com/lh3/seqtk
- **FragGeneScan** (short-read open reading frame prediction dependency for protein-mode seed generation) — https://github.com/hallamlab/FragGeneScanPlus

## Examples

```
python CasCollect.py -fwd reads_1.fastq -rev reads_2.fastq -out cas_output --trim -cycle 2 -cpu 1 -mem 20
```

## Evaluation signals

- Cas operons and CRISPR arrays are successfully annotated by CRISPRCasFinder across the analyzed dataset, matching complete-assembly results (e.g. agreement across the 66 P. aeruginosa CDC/FDA isolate panel with only minor single-strain differences)
- Coverage of a known cas operon saturates across expansion cycles: ~33.2% for seed reads, 96.1% after one cycle, 99.5% after two cycles on the ground water metagenome
- Contigs ≥500 bp are produced from the targeted assembly and are annotatable (e.g. 19 vs 146, 111 vs 162, 89 vs 44 contigs vs complete assembly for E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39)
- All previously known cas genes in an isolate are recovered, as both pipelines did for the three bacterial isolate benchmarks
- CasCollect pre-run/Check.py confirms all required software (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) is present in the user's path

## Limitations

- No software version numbers or dataset accession IDs are provided in the article for tools or inputs, complicating exact reproduction.
- Timing gains versus complete assembly are dataset-dependent: CasCollect was slower than complete assembly for E. coli KLY (23.3 vs 17.4 min) and P. aeruginosa VA-134 (51.2 vs 33.3 min), and only faster for S. pyogenes M39 (21.6 vs 31.5 min).
- Minor single-strain discrepancies can occur versus complete assembly (e.g. strain AR110 spacer count and strain AR356 cas operon detection in the 66-isolate panel).
- DNA mode requires a user-provided CRISPR repeat reference; default protein mode depends on the fixed 120 Cas HMM set and may miss atypical systems.
- VSEARCH expansion identity (default 95%) and cycle count (default five) may need tuning; too few cycles can leave coverage gaps.

## Evidence

- [results] The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation: "The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation"
- [results] The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command.: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] by HMMER (21) with a default set of 120 Cas protein family profile HMMs (13) as query: "by HMMER (21) with a default set of 120 Cas protein family profile HMMs (13) as query"
- [results] This pipeline relies on several third-party programs that include: HMMER (21), VSEARCH (22), SPAdes (23) and CRISPRCasFinder (13): "This pipeline relies on several third-party programs that include: HMMER (21), VSEARCH (22), SPAdes (23) and CRISPRCasFinder (13)"
- [readme] CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads.: "CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads."
