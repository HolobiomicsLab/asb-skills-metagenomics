---
name: overlap-extension-read-recruitment
description: Use when a small set of reads mapped to a partial or remotely related
  viral reference (SAM file) must be expanded to recover more reads of the same virus
  from a larger metagenomic read set (FASTA) for downstream haplotype assembly.
when_to_use_negative:
- The target virus shares no conservation with any reference genome — TAR-VIR's classification
  (seed alignment) step will produce no seed reads to extend
- Reads are already classified/enriched and only haplotype assembly is needed (use
  PEHaplo directly instead)
- Reads are highly chimeric and the overlap cutoff cannot be set longer than half
  the read size without losing recruitment sensitivity
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: TAR-VIR
  role: Provides the Overlap_extension module (build and overlap binaries) that performs
    the iterative BWT-index-based read recruitment
  repo: https://github.com/chjiao/TAR-VIR
- name: Bowtie2
  role: Recommended upstream aligner (default parameters, error function 'L,0,-0.6')
    to produce the seed-read SAM file used as recruitment queries
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: PEHaplo
  role: Downstream de novo haplotype assembly tool that consumes the recruited-reads
    FASTA via its '-f' option
  repo: https://github.com/chjiao/PEHaplo
- name: karect
  role: Error correction tool applied to reads as preprocessing before alignment and
    recruitment
  repo: https://github.com/aminallam/karect
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:1570805924ac908eef8705fb6d0b0d5542a69ef40c97b2c9d4155b133778a5fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/TAR-VIR
  tools:
  - Bowtie2
  - PEHaplo
  - TAR-VIR
  - karect
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# overlap-extension-read-recruitment

## Summary

Iterative overlap-extension read recruitment from TAR-VIR that expands a small seed set of aligned viral reads (e.g., 287 HIV-1 reads in a SAM file) into a much larger recruited set (e.g., 8,008 reads) using a BWT/FM-index over the read set and an overlap cutoff longer than half the read length.

## When to use

Use when a small set of reads mapped to a partial or remotely related viral reference (SAM file) must be expanded to recover more reads of the same virus from a larger metagenomic read set (FASTA) for downstream haplotype assembly.

## When NOT to use

- The target virus shares no conservation with any reference genome — TAR-VIR's classification (seed alignment) step will produce no seed reads to extend
- Reads are already classified/enriched and only haplotype assembly is needed (use PEHaplo directly instead)
- Reads are highly chimeric and the overlap cutoff cannot be set longer than half the read size without losing recruitment sensitivity

## Inputs

- SAM file of seed reads aligned to a partial or remotely related reference sequence (e.g., HIV.sam with 287 aligned HIV-1 reads)
- FASTA file of the full metagenomic read set (e.g., virus.fa with simulated HIV-1, HCV genotype 1, and HGV reads)
- Overlap cutoff parameter in bp (set longer than half the read length; 180 for 250 bp reads)

## Outputs

- FASTA file of recruited reads (e.g., virus_recruit.fa), containing both single- and paired-end reads suitable for PEHaplo input via its '-f' option
- BWT/FM-index prefix files built from the read FASTA

## How to apply

First, build a read index from the metagenomic read FASTA by running the Overlap_extension 'build' binary, which constructs a BWT/FM-index (generalized suffix array over concatenated reads) for efficient overlap detection. Then run './overlap' with the seed alignment SAM (-S), the index (-x), the read FASTA (-f), and an overlap cutoff (-c) set longer than half the read size — e.g., 180 bp for 250 bp reads — to prevent recruiting chimeric reads. The tool iteratively uses newly recruited reads as new queries against the BWT index until no additional reads can be recruited, then writes recruited reads to a FASTA. Correctness is judged by the output FASTA being produced and its read count matching the documented expectation (8,008 for the HIV test example).

## Related tools

- **TAR-VIR** (Provides the Overlap_extension module (build and overlap binaries) that performs the iterative BWT-index-based read recruitment) — https://github.com/chjiao/TAR-VIR
- **Bowtie2** (Recommended upstream aligner (default parameters, error function 'L,0,-0.6') to produce the seed-read SAM file used as recruitment queries) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **PEHaplo** (Downstream de novo haplotype assembly tool that consumes the recruited-reads FASTA via its '-f' option) — https://github.com/chjiao/PEHaplo
- **karect** (Error correction tool applied to reads as preprocessing before alignment and recruitment) — https://github.com/aminallam/karect

## Examples

```
./build -f test_data/virus.fa -o virus; ./overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa
```

## Evaluation signals

- Running the documented test example ('./build -f test_data/virus.fa -o virus' then './overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa') must yield exactly 8,008 recruited reads
- The recruited-read FASTA (virus_recruit.fa) is produced and is non-empty
- The overlap cutoff (-c) is greater than half the read length (e.g., 180 for 250 bp reads) to prevent recruiting chimeric reads
- The number of recruited reads greatly exceeds the number of seed reads in the input SAM (e.g., 8,008 vs. 287 seeds), confirming iterative expansion occurred
- The tool terminates (no infinite querying), indicating convergence of the iterative recruitment step

## Limitations

- A completely new virus that does not share any conservation with any reference genome will not be detected, since no seed reads can be aligned
- The overlap threshold must be set longer than half of the read size to prevent recruiting chimeric reads; no specific recommended value is given beyond this guidance
- In the large real-data test, reads had to be pre-processed: karect error correction, bamtagger removal of human reads, and Bowtie2 removal of bacteria and phage reads before classification
- Parallel runs constrain the thread count: k = n*t (n = 1, 2, 3, ...) because of how the index is partitioned

## Evidence

- [readme] Iterative overlap-extension recruitment expands a small seed set: "This SAM file contains a small subset of aligned HIV-1 reads. With these aligned (287) reads, more HIV-1 reads can be recruited from the viral metagenomics data set."
- [readme] Documented example command and expected output count: "./overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa    If everything is good, the recruited reads number should be 8008."
- [methods] Rationale for the overlap cutoff threshold: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [methods] Seed reads are defined by mappability to the reference: "All the reads that can be mapped to the reference constitute the set of “seed reads”."
- [intro] Scope limitation: no-detection case for novel viruses: [evidence span withheld — anchor status: not_found]
