---
name: assembly-contiguity-metrics-computation
description: Reference for evaluating and comparing de novo short-read assemblies
  (e.g., SOAPdenovo2 vs SOAPdenovo1.05) on benchmark datasets such as GAGE (S. aureus,
  R. sphaeroides, human chr14) or Assemblathon1 using N50, correct assembly length,
  and error counts.
when_to_use_negative:
- Input is raw sequencing reads with no assembly produced yet — build the assembly
  first (pregraph/contig/map/scaff) before computing metrics
- Comparing assemblers on metagenomic or single-cell samples, where the README points
  to MEGAHIT rather than the SOAPdenovo benchmarking framework
- No reference genome or benchmark truth set is available, since correct assembly
  length and error counts require reference-based evaluation
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: Primary assembler whose contig/scaffold outputs (*.contig, *.scafSeq) are
    evaluated for N50, correct assembly length, and error counts
  repo: github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1.05
  role: Predecessor baseline assembly used for the comparison on Assemblathon1 and
    GAGE
- name: GapCloser
  role: Gap-closure module whose improved output affects final scaffold contiguity
    metrics
  repo: github.com/aquaskyline/SOAPdenovo2
- name: ALLPATHS-LG
  role: External assembler compared against on Assemblathon1 (SOAPdenovo2 contig/scaffold
    N50 were ~1.53x and ~1.84x longer)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:a0375f6bdd7f4729833a3f9c08f226762bc3ae10b73f0f483b9489d5f0491ea4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALLPATHS-LG
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# assembly-contiguity-metrics-computation

## Summary

Computes contiguity and accuracy metrics (contig/scaffold N50, correct assembly length, structural/substitution/copy-number error counts) for de novo genome assemblies, as used to benchmark SOAPdenovo2 against SOAPdenovo1.05 on the Assemblathon1 and GAGE datasets.

## When to use

Reference for evaluating and comparing de novo short-read assemblies (e.g., SOAPdenovo2 vs SOAPdenovo1.05) on benchmark datasets such as GAGE (S. aureus, R. sphaeroides, human chr14) or Assemblathon1 using N50, correct assembly length, and error counts.

## When NOT to use

- Input is raw sequencing reads with no assembly produced yet — build the assembly first (pregraph/contig/map/scaff) before computing metrics
- Comparing assemblers on metagenomic or single-cell samples, where the README points to MEGAHIT rather than the SOAPdenovo benchmarking framework
- No reference genome or benchmark truth set is available, since correct assembly length and error counts require reference-based evaluation

## Inputs

- Assembled contig sequences (*.contig) from SOAPdenovo2
- Assembled scaffold sequences (*.scafSeq) from SOAPdenovo2
- GAGE benchmark read sets for Staphylococcus aureus, Rhodobacter sphaeroides, and human chr14
- Assemblathon1 benchmark dataset
- Reference genomes (GAGE references, NCBI human reference genome hg19)

## Outputs

- Contig N50 and scaffold N50 per assembly
- Correct assembly length relative to the reference
- Structural, substitution, and copy-number error counts per assembly
- Comparison table (e.g., gage_assembly_evaluation_table.csv) mirroring the paper's Table 1 / Figure 1

## How to apply

Assemble each read set with the assembler's standard pipeline (error correction, sparse de Bruijn graph construction via sparse_pregraph, contig assembly, PE mapping, scaffolding, GapCloser gap closure), keeping the final .scafSeq and .contig outputs. Align contigs and scaffolds against the benchmark reference (e.g., GAGE references or NCBI hg19 for the YH genome) to compute correct assembly length, contig N50, and scaffold N50. Quantify misassemblies as structural, substitution, and copy-number errors, then compare across assembler versions — SOAPdenovo2 reduced structural error by 90.12%, substitution error by 92.13%, and copy number error by 69.47% versus SOAPdenovo1. Compile the per-dataset comparison (e.g., into gage_assembly_evaluation_table.csv) mirroring the paper's Table 1/Figure 1; validation is successful when compiled reductions reproduce the reported values and contig/scaffold N50 differences match expectations (e.g., YH scaffold N50 ~22 Mbp, 50-fold longer than the first published version).

## Related tools

- **SOAPdenovo2** (Primary assembler whose contig/scaffold outputs (*.contig, *.scafSeq) are evaluated for N50, correct assembly length, and error counts) — github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1.05** (Predecessor baseline assembly used for the comparison on Assemblathon1 and GAGE)
- **GapCloser** (Gap-closure module whose improved output affects final scaffold contiguity metrics) — github.com/aquaskyline/SOAPdenovo2
- **ALLPATHS-LG** (External assembler compared against on Assemblathon1 (SOAPdenovo2 contig/scaffold N50 were ~1.53x and ~1.84x longer))

## Examples

```
SOAPdenovo2 all -s config_file -K 63 -R -F -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- Compiled error reductions reproduce Table 1 values: structural error reduced by 90.12%, substitution error by 92.13%, copy number error by 69.47% versus SOAPdenovo1
- Correct assembly length increase of SOAPdenovo2 over SOAPdenovo1 falls in the reported ~3 to 80-fold range on GAGE datasets
- Scaffold N50 per assembly is finite and reported alongside contig N50; YH values should approximate ~20.9 kbp contig N50 and ~22 Mbp scaffold N50
- Both *.contig and *.scafSeq output files exist and are non-empty after a completed all/pregraph-contig-map-scaff run
- Per-dataset rows in the comparison table cover all GAGE datasets (S. aureus, R. sphaeroides, human chr14) with both assembler versions

## Limitations

- SOAPdenovo targets Illumina GA short reads and 64-bit Linux with at least 5 GB RAM (~150 GB for human genomes), so metrics on other platforms/read types are out of scope
- Benchmark conclusions are grounded in Assemblathon1 and GAGE datasets; results may not transfer to other genome properties
- Error metrics require a reference; the paper aligned YH contigs to NCBI hg19 and 134 Mb of published human SD sequences, so novel genomes lack these validations

## Evidence

- [abstract] the contig and scaffold N50 of the YH genome were ~20.9 kbp and ~22 Mbp, respectively, which is 3-fold and 50-fold longer than the first published version.: "the contig and scaffold N50 of the YH genome were ~20.9 kbp and ~22 Mbp, respectively, which is 3-fold and 50-fold longer than the first published version."
- [abstract] Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo and is competitive to other assemblers on both assembly length and accuracy.: "Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo and is competitive to other assemblers on both assembly length and accuracy."
- [results] copy number error by 69.47% (Table 1, Figure 1): "copy number error by 69.47% (Table 1, Figure 1)"
- [readme] scaffold sequences (final contig sequences can be extracted by breaking down scaffold sequences at gap regions).: [evidence span withheld — anchor status: external_doc]
