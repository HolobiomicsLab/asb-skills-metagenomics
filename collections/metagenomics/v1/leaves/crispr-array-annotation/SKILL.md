---
name: crispr-array-annotation
description: Use when assembled contigs (e.g. SPAdes output from seed-expanded read
  subsets) must be annotated to locate CRISPR arrays and cas genes, or when validating
  that a CRISPR/Cas detection pipeline identified cas operons.
when_to_use_negative:
- Input is already assembled and annotated genomes where CRISPR arrays can be called
  directly by CRISPRCasFinder without the targeted read-level pipeline
- Query data are metagenomic and metaSPAdes handling is required but unavailable —
  the paper substitutes metaSPAdes in-place of SPAdes for metagenomes
- The goal is de novo discovery of CRISPR repeats with no prior repeat reference and
  no reliance on the 120 default Cas protein HMMs, since seed generation depends on
  these resources
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CasCollect
  role: Pipeline wrapper whose fifth step (annotation) runs CRISPRCasFinder on the
    targeted-assembly contigs; provides default 120 Cas protein HMMs and 105 CRISPR
    repeat FASTAs
  repo: https://github.com/sandialabs/CasCollect
- name: CRISPRCasFinder
  role: Searches the targeted assembly contigs to identify CRISPR arrays and cas protein
    genes
- name: SPAdes
  role: Assembles the seed and expansion read subsets into contigs that are then annotated
- name: VSEARCH
  role: DNA-mode identity matching of reads to user-defined CRISPR array repeats before
    assembly (default 95% match)
  repo: https://github.com/torognes/vsearch
- name: HMMER
  role: Protein-mode seed generation against the default 120 Cas protein family profile
    HMMs
- name: bbduk
  role: 'Implements the --trim/--clean step: quality trimming, adapter removal, contaminant
    removal and read merging prior to seed generation'
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:165514699be8754817fb75dd12b8b47adfc6ce2b315ce17e57348c247d4415ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - SPAdes
  - VSEARCH
  - bbduk
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# crispr-array-annotation

## Summary

Annotate targeted-assembly (or whole-assembly) contigs with CRISPRCasFinder to identify CRISPR arrays and cas protein genes, the final step of the CasCollect pipeline for detecting CRISPR/Cas systems from raw Illumina reads.

## When to use

Use when assembled contigs (e.g. SPAdes output from seed-expanded read subsets) must be annotated to locate CRISPR arrays and cas genes, or when validating that a CRISPR/Cas detection pipeline identified cas operons.

## When NOT to use

- Input is already assembled and annotated genomes where CRISPR arrays can be called directly by CRISPRCasFinder without the targeted read-level pipeline
- Query data are metagenomic and metaSPAdes handling is required but unavailable — the paper substitutes metaSPAdes in-place of SPAdes for metagenomes
- The goal is de novo discovery of CRISPR repeats with no prior repeat reference and no reliance on the 120 default Cas protein HMMs, since seed generation depends on these resources

## Inputs

- FASTA contigs from SPAdes assembly of CasCollect seed-expanded read subsets (contigs >= 500 bp retained for annotation)
- Raw paired-end Illumina FASTQ reads (via the full CasCollect pipeline: --trim/--clean, seed generation, read subset expansion)
- User-defined CRISPR repeat FASTA for DNA mode (e.g. Pseudomonas_aeruginosa_DK2.fas; 105 CRISPR repeat sequences ship with CasCollect)

## Outputs

- CRISPRCasFinder annotations of CRISPR arrays (repeat-spacer structure) per isolate
- Cas protein gene annotations per isolate (cas operons)
- CasCollect output folder (given by -out) with assembled contigs and annotation results

## How to apply

Feed the SPAdes-assembled contigs from CasCollect's targeted assembly into the automated annotation step, which searches contigs with CRISPRCasFinder to call CRISPR arrays and cas protein genes. Apply the Perl automation script's 500 bp contig size cutoff, which the paper used to reduce the number of contigs analyzed (e.g. to an average of 161K contigs for complete assemblies). CasCollect relies on HMMER with a default set of 120 Cas protein family profile HMMs and, in DNA mode, VSEARCH identity matching against user-defined CRISPR repeats upstream of assembly, so annotation should be checked against both protein-level (HMM) and repeat-level (nucleotide) seed evidence. Write all outputs to the folder given by -out and compare detected arrays/spacers and cas operon content against a complete-assembly reference pipeline to judge correctness; in the 66-isolate P. aeruginosa CDC & FDA Antibiotic Resistance Isolate Bank case study, the targeted pipeline matched complete assembly with only minor single-strain differences (e.g. AR110 spacer count, AR356 cas operon).

## Related tools

- **CasCollect** (Pipeline wrapper whose fifth step (annotation) runs CRISPRCasFinder on the targeted-assembly contigs; provides default 120 Cas protein HMMs and 105 CRISPR repeat FASTAs) — https://github.com/sandialabs/CasCollect
- **CRISPRCasFinder** (Searches the targeted assembly contigs to identify CRISPR arrays and cas protein genes)
- **SPAdes** (Assembles the seed and expansion read subsets into contigs that are then annotated)
- **VSEARCH** (DNA-mode identity matching of reads to user-defined CRISPR array repeats before assembly (default 95% match)) — https://github.com/torognes/vsearch
- **HMMER** (Protein-mode seed generation against the default 120 Cas protein family profile HMMs)
- **bbduk** (Implements the --trim/--clean step: quality trimming, adapter removal, contaminant removal and read merging prior to seed generation) — https://jgi.doe.gov/data-and-tools/bbtools/

## Examples

```
python CasCollect.py reads_1.fastq reads_2.fastq --trim --nucl -query Pseudomonas_aeruginosa_DK2.fas -cycle 2 -out cascollect_out/
```

## Evaluation signals

- CRISPRCasFinder reports CRISPR arrays with repeat-spacer structure and cas protein genes on the annotated contigs for each input isolate
- Contigs shorter than the 500 bp annotation cutoff are excluded from analysis
- Detected cas operons and CRISPR arrays match a complete assembly + annotation reference pipeline, as in the 66-isolate P. aeruginosa panel case study (only minor single-strain differences expected, e.g. AR110 spacer count, AR356 cas operon)
- CasCollect's pre-run check (or Check.py) confirms all dependencies (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) are present in the user's path
- All outputs are written to the folder specified by -out

## Limitations

- DNA mode requires a user-defined CRISPR repeat query; sensitivity depends on repeat similarity to the provided reference (e.g. the P. aeruginosa CRISPR repeat for the isolate panel)
- Pipeline steps are distinct and optional; skipping read filtering or misconfiguring seed expansion (-cycle, default 5; -match, default 95%) changes which contigs reach annotation
- No software version numbers stated for CasCollect or its third-party dependencies, and no changelog is available
- For metagenomes, metaSPAdes must be substituted for SPAdes

## Evidence

- [results] The targeted assembly contigs are searched with the CRISPRCasFinder program (13).: "The targeted assembly contigs are searched with the CRISPRCasFinder program (13)."
- [results] The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation: "The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation"
- [results] Our Perl script for automating the annotation had set a size cutoff of 500 bp, which reduced the total number of contigs analyzed for the complete assembly: "Our Perl script for automating the annotation had set a size cutoff of 500 bp, which reduced the total number of contigs analyzed for the complete assembly"
- [readme] **CasCollect** is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads.: "**CasCollect** is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads."
- [results] This pipeline relies on several third-party programs that include: HMMER (21), VSEARCH (22), SPAdes (23) and CRISPRCasFinder (13): "This pipeline relies on several third-party programs that include: HMMER (21), VSEARCH (22), SPAdes (23) and CRISPRCasFinder (13)"
