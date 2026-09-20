---
name: assembly-error-classification-and-counting
description: Use when comparing de novo assembler releases (e.g., SOAPdenovo2 vs SOAPdenovo1.05)
  on GAGE-style benchmark datasets (S. aureus, R. sphaeroides, human chr14) and you
  need per-error-type counts and correct assembly length metrics.
when_to_use_negative:
- The task is metagenomic or single-cell assembly with MEGAHIT rather than GAGE-benchmark-style
  reference-based evaluation of a single isolate genome
- No reference genome is available, so errors cannot be classified against ground
  truth (e.g., de novo projects like the YH genome reassembly where only N50/coverage
  gains are reported)
- Input is already an evaluated assembly (e.g., a published GAGE result table) rather
  than raw assembler output
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: Assembler under evaluation; produces the .contig and .scafSeq assemblies whose
    errors are classified and counted
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1.05
  role: Predecessor baseline assembler against which error reductions (structural
    90.12%, substitution 92.13%, copy number 69.47%) are computed
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: GapCloser
  role: Gap-closure module of the SOAPdenovo2 pipeline whose improvement affects the
    error counts measured on GAGE datasets
  repo: https://github.com/aquaskyline/SOAPdenovo2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:720982ff9a57ad44e20133705b14ba64ceae370eeecceaf8784d289fd20776d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/SOAPdenovo2
  tools:
  - GapCloser
  - SOAPdenovo
  - SOAPdenovo1.05
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# assembly-error-classification-and-counting

## Summary

Classify and quantify de novo assembly errors into structural, substitution, and copy-number categories by aligning contigs/scaffolds to a reference genome, as done in the SOAPdenovo2 GAGE benchmark where errors dropped 90.12% (structural), 92.13% (substitution), and 69.47% (copy number) versus SOAPdenovo1.

## When to use

Use when comparing de novo assembler releases (e.g., SOAPdenovo2 vs SOAPdenovo1.05) on GAGE-style benchmark datasets (S. aureus, R. sphaeroides, human chr14) and you need per-error-type counts and correct assembly length metrics.

## When NOT to use

- The task is metagenomic or single-cell assembly with MEGAHIT rather than GAGE-benchmark-style reference-based evaluation of a single isolate genome
- No reference genome is available, so errors cannot be classified against ground truth (e.g., de novo projects like the YH genome reassembly where only N50/coverage gains are reported)
- Input is already an evaluated assembly (e.g., a published GAGE result table) rather than raw assembler output

## Inputs

- Assembled contig files (*.contig) and scaffold sequences (*.scafSeq) from SOAPdenovo2 and SOAPdenovo1.05
- GAGE benchmark reference genomes for Staphylococcus aureus, Rhodobacter sphaeroides, and human chr14
- GAGE benchmark Illumina read sets (FASTQ/FASTA, paired-end and mate-pair libraries)

## Outputs

- Per-assembly error counts stratified as structural, substitution, and copy-number errors
- Correct assembly length per assembler per GAGE dataset
- Contig N50 and scaffold N50 statistics
- gage_assembly_evaluation_table.csv mirroring Table 1 / Figure 1 of the SOAPdenovo2 paper

## How to apply

Run each assembler through its full pipeline (error correction, de Bruijn graph construction, contig assembly, PE mapping, scaffolding, GapCloser gap closure) on the same benchmark read sets, then align the resulting .contig and .scafSeq outputs to the GAGE references. Classify misalignments into the three GAGE error categories — structural, substitution, and copy number — and compute correct assembly length alongside contig and scaffold N50. Compile the counts into a comparison table mirroring the paper's Table 1 / Figure 1, expressing improvements as percent reductions relative to the baseline assembler (SOAPdenovo1.05). Correct application reproduces the paper's reported reductions: structural error −90.12%, substitution error −92.13%, copy number error −69.47%.

## Related tools

- **SOAPdenovo2** (Assembler under evaluation; produces the .contig and .scafSeq assemblies whose errors are classified and counted) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1.05** (Predecessor baseline assembler against which error reductions (structural 90.12%, substitution 92.13%, copy number 69.47%) are computed) — https://github.com/aquaskyline/SOAPdenovo2
- **GapCloser** (Gap-closure module of the SOAPdenovo2 pipeline whose improvement affects the error counts measured on GAGE datasets) — https://github.com/aquaskyline/SOAPdenovo2

## Evaluation signals

- Structural error count for SOAPdenovo2 is 90.12% lower than SOAPdenovo1 on GAGE datasets (Table 1)
- Substitution error count is 92.13% lower and copy number error count 69.47% lower than SOAPdenovo1 (Table 1, Figure 1)
- Correct assembly length increase falls within the reported ~3 to 80-fold range across GAGE datasets (Tables 2 and 3)
- Assembled scaffold N50 approaches an order of magnitude longer than SOAPdenovo1 with higher accuracy
- Output table contains one row per assembler per GAGE dataset with all three error categories populated and no missing metrics

## Limitations

- Error classification requires a trusted reference genome, so it only applies to benchmark datasets like GAGE, not to genuinely novel genomes
- GAGE results depend on the full six-module pipeline (error correction through GapCloser); changing any module (e.g., the improved Error Correction or Scaffolding modules) changes the error profile
- Larger k-mer sizes require deep sequencing depth and longer read length to guarantee overlap, so error counts are sensitive to k-mer choice (accepted odd values 13–63/127)

## Evidence

- [results] Structural error reduced by 90.12% vs SOAPdenovo1: "reduction of structural error by 90.12%"
- [results] Copy number error reduced by 69.47% vs SOAPdenovo1: "copy number error by 69.47% (Table 1, Figure 1)"
- [abstract] Benchmark datasets used for error comparison: "Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo"
- [supplementary] GAGE evaluation procedure documented in supplementary methods: "Supplementary Method 6 P9 Evaluating the GAGE dataset."
- [readme] Assembler output files that feed into error classification: [evidence span withheld — anchor status: external_doc]
