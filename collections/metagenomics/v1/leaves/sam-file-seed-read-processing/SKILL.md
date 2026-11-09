---
name: sam-file-seed-read-processing
description: Use when you have a SAM file of reads already aligned to a partial or
  remotely related viral reference (seed reads) and need to recruit additional related
  reads from a larger metagenomic read set for strain reconstruction.
when_to_use_negative:
- The virus shares no conservation with any reference genome, since TAR-VIR's read
  classification requires seed reads mappable to a partial or remotely related reference
- The input is raw, uncorrected reads — the README requires error correction (karect
  by default) and Bowtie2 alignment before running overlap extension
- You need reference-free detection of completely novel viruses rather than enrichment
  of reads related to a known reference
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: TAR-VIR
  role: Provides the Overlap_extension module (build and overlap binaries) that consumes
    the SAM seed alignment and recruits reads via BWT/FM-index overlap detection
  repo: https://github.com/chjiao/TAR-VIR
- name: Bowtie2
  role: Recommended mapper to align the read set against the reference (default parameters,
    error function 'L,0,-0.6') and produce the SAM seed file
- name: PEHaplo
  role: Downstream de novo haplotype assembler that consumes the recruited-reads FASTA
    (single-file input via '-f')
  repo: https://github.com/chjiao/PEHaplo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:c896e6b6d196cc786f49f8c0525abe58b28e0b221bdabb6fb47a6d279acd5ce5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/TAR-VIR
  tools:
  - Bowtie2
  - PEHaplo
  - TAR-VIR
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# sam-file-seed-read-processing

## Summary

Uses a SAM alignment file of seed reads as input to TAR-VIR's overlap-extension read recruitment step, which iteratively recruits reads far beyond the seed set from a BWT/FM-index of the full read set.

## When to use

Use when you have a SAM file of reads already aligned to a partial or remotely related viral reference (seed reads) and need to recruit additional related reads from a larger metagenomic read set for strain reconstruction.

## When NOT to use

- The virus shares no conservation with any reference genome, since TAR-VIR's read classification requires seed reads mappable to a partial or remotely related reference
- The input is raw, uncorrected reads — the README requires error correction (karect by default) and Bowtie2 alignment before running overlap extension
- You need reference-free detection of completely novel viruses rather than enrichment of reads related to a known reference

## Inputs

- SAM file of seed reads aligned to a reference sequence (e.g. produced by Bowtie2 with error function 'L,0,-0.6')
- FASTA file of the full read set (e.g. error-corrected viral metagenomic reads in virus.fa)
- Overlap cutoff parameter in bp (set longer than half the read size, e.g. 180 for 250 bp reads)

## Outputs

- FASTA file of recruited reads (e.g. virus_recruit.fa), ready for de novo assembly with PEHaplo via its '-f' single-fasta option

## How to apply

First align the read set against the reference with Bowtie2 (default parameters plus the allowed error function 'L,0,-0.6') to produce the SAM file of seed reads. Build a BWT/FM-index (generalized suffix array over the concatenated reads) with the Overlap_extension 'build' binary: './build -f reads.fa -o prefix'. Then run recruitment with './overlap -S align.sam -x prefix -f reads.fa -c overlap_cutoff -o recruited_reads.fa', choosing the overlap cutoff longer than half the read size (e.g. 180 bp for 250 bp reads) to avoid recruiting chimeric reads. The tool iteratively uses newly recruited reads as new queries to the BWT index until no additional reads can be recruited; the resulting FASTA can be passed to PEHaplo for de novo haplotype assembly. In the paper's runnable test case, 287 HIV-1 seed reads in HIV.sam recruit 8,008 reads from virus.fa with cutoff 180, which is the expected validation number.

## Related tools

- **TAR-VIR** (Provides the Overlap_extension module (build and overlap binaries) that consumes the SAM seed alignment and recruits reads via BWT/FM-index overlap detection) — https://github.com/chjiao/TAR-VIR
- **Bowtie2** (Recommended mapper to align the read set against the reference (default parameters, error function 'L,0,-0.6') and produce the SAM seed file)
- **PEHaplo** (Downstream de novo haplotype assembler that consumes the recruited-reads FASTA (single-file input via '-f')) — https://github.com/chjiao/PEHaplo

## Examples

```
./overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa
```

## Evaluation signals

- The recruited-reads FASTA is produced and is non-empty when seed reads align to the reference
- On the repository test data (virus.fa + HIV.sam, cutoff 180), the recruited read count is exactly 8,008
- The overlap cutoff is set longer than half the read size (e.g. 180 for 250 bp reads), so chimeric reads are excluded
- Recruited reads substantially expand the seed set — in the paper's HPgV test, 19,714 seed reads yielded 118,339 recruited reads at cutoff 150
- Recruitment iterates until no additional reads can be recruited (convergence), using newly recruited reads as new BWT-index queries

## Limitations

- A completely new virus that does not share any conservation with any reference genome won't be detected by this method, since seed reads must be mappable to a reference
- The overlap threshold must exceed half the read size to prevent recruiting chimeric reads; no single universally recommended cutoff value is specified and users must choose based on common string sizes between species
- For large datasets, the thread parameter '-t' must satisfy k = nt (n = 1, 2, 3, ...) due to index partitioning in the implementation

## Evidence

- [readme] This SAM file contains a small subset of aligned HIV-1 reads. With these aligned (287) reads, more HIV-1 reads can be recruited from the viral metagenomics data set.: "This SAM file contains a small subset of aligned HIV-1 reads. With these aligned (287) reads, more HIV-1 reads can be recruited from the viral metagenomics data set."
- [methods] we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [methods] All the reads that can be mapped to the reference constitute the set of “seed reads”.: "All the reads that can be mapped to the reference constitute the set of “seed reads”."
- [readme] If everything is good, the recruited reads number should be 8008.: "If everything is good, the recruited reads number should be 8008."
