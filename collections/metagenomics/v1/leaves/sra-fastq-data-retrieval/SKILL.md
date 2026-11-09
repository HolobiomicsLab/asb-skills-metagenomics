---
name: sra-fastq-data-retrieval
description: Use when an analysis pipeline starts from a public NCBI SRA run accession
  (e.g. SRR6287056) and raw paired-end FASTQ files for both mates must be downloaded
  before preprocessing steps such as hg19 host-read depletion.
when_to_use_negative:
- The reads are already available locally as FASTQ files — downloading from SRA again
  is unnecessary.
- The required data are aligned reads (BAM) or assembled contigs rather than raw reads;
  fasterq-dump is designed for raw read extraction.
- The target data are not deposited in NCBI SRA/ENA (e.g. only available as in silico
  simulated reads bundled with the contig-extender examples folder).
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: sra-tools
  role: Downloads the SRA run (prefetch) and converts it to paired-end FASTQ for both
    mates (fasterq-dump)
- name: ContigExtender
  role: Consumes the retrieved (and subsequently host-depleted, adaptor-trimmed) FASTQ
    reads to extend seed contigs after de novo assembly
  repo: https://github.com/dengzac/contig-extender
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:dd1a16aa884a13c0015785671e20868d1674e11c06ffabee54d0da956b0b2769
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - ContigExtender
  - sra-tools
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sra-fastq-data-retrieval

## Summary

Retrieves raw paired-end Illumina FASTQ reads for a metagenomic SRA accession (e.g. SRR6287056) from NCBI using sra-tools (prefetch/fasterq-dump), yielding the mate FASTQ files needed downstream for hg19 host depletion and de novo assembly with tools like ContigExtender/metaSPAdes.

## When to use

Use when an analysis pipeline starts from a public NCBI SRA run accession (e.g. SRR6287056) and raw paired-end FASTQ files for both mates must be downloaded before preprocessing steps such as hg19 host-read depletion.

## When NOT to use

- The reads are already available locally as FASTQ files — downloading from SRA again is unnecessary.
- The required data are aligned reads (BAM) or assembled contigs rather than raw reads; fasterq-dump is designed for raw read extraction.
- The target data are not deposited in NCBI SRA/ENA (e.g. only available as in silico simulated reads bundled with the contig-extender examples folder).

## Inputs

- NCBI SRA run accession (e.g. SRR6287056) for a human metagenomic sample
- sra-tools (prefetch, fasterq-dump) installed locally

## Outputs

- Paired-end FASTQ files for both mates (e.g. _1.fastq and _2.fastq) with headers beginning with the SRR accession
- Raw reads ready for hg19 Bowtie2 host depletion and downstream de novo assembly

## How to apply

First run sra-tools prefetch on the SRA run accession to fetch the raw data from NCBI, then run fasterq-dump to convert it into FASTQ files for both mates of the paired-end library. Confirm the output headers begin with the SRR accession so the files can be traced to the run. These FASTQ files are the inputs to the host-depletion step, where they are indexed/aligned against hg19 with Bowtie2 and unmapped (non-human) reads are extracted with samtools view/fastq to produce hg19_depleted_reads.fastq. Before extension with ContigExtender, preprocess the FASTQ by removing adaptors, since the README advises removing adaptors before attempting extension for best results.

## Related tools

- **sra-tools** (Downloads the SRA run (prefetch) and converts it to paired-end FASTQ for both mates (fasterq-dump))
- **ContigExtender** (Consumes the retrieved (and subsequently host-depleted, adaptor-trimmed) FASTQ reads to extend seed contigs after de novo assembly) — https://github.com/dengzac/contig-extender

## Examples

```
prefetch SRR6287056 && fasterq-dump SRR6287056
```

## Evaluation signals

- FASTQ files exist for both mates and each header begins with the expected SRR accession (e.g. SRR6287056).
- File sizes/read counts match the SRA run's reported spot count for both mates (equal numbers of records per mate).
- Re-aligning the retrieved reads (after hg19 depletion) to hg19 with Bowtie2 yields essentially no mapped reads, confirming the correct raw material was fetched and processed.
- The FASTQ files pass adaptor removal and load into downstream tools (Bowtie2 indexing/alignment, ContigExtender wrapper) without format errors.

## Limitations

- Large SRA runs may require substantial disk space and time for prefetch/fasterq-dump; network failures can leave truncated FASTQ files that only manifest as header/count mismatches.
- The retrieved raw reads still contain human host contamination and low-complexity reads; they must pass through Bowtie2 hg19 depletion and PRINSEQ v0.20.4 complexity filtering (default parameters) before assembly.
- ContigExtender only supports Linux (via Docker), so the retrieved FASTQ must be processed on a Linux system for the extension step.

## Evidence

- [methods] sra-tools prefetch/fasterq-dump of SRR6287056 yields paired FASTQ: [evidence span withheld — anchor status: not_found]
- [methods] Validation of retrieved/depleted FASTQ: [evidence span withheld — anchor status: not_found]
- [readme] Adaptor removal before extension: "For best results, preprocess fastq by removing adaptors before attemping extension"
- [readme] Only Linux supported for downstream extension: "The only supported operating system is Linux."
