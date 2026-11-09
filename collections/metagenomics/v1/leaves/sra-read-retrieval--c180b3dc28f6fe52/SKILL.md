---
name: sra-read-retrieval
description: Use when a benchmark workflow specifies an SRA accession (e.g. SRX6602475)
  as the real biological read input and the raw nanopore FASTQ/FASTA reads must be
  downloaded before classification or alignment.
when_to_use_negative:
- When the benchmark requires simulated reads (e.g. PBSIM2-generated CHM13 nanopore
  reads at 90% base-call accuracy) — use the simulator instead of SRA retrieval
- When the accession is not specified or the target data are reference genomes (e.g.
  CHM13, Ashkenazi, GRCh38), which come from genome assemblies, not SRA read archives
edam_operation: http://edamontology.org/operation_0227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SPUMONI
  role: 'Downstream consumer of the retrieved reads: classifies the FASTA read set
    against the SPUMONI index with PML matching statistics'
  repo: https://github.com/oma219/spumoni
- name: minimap2
  role: Downstream baseline aligner that classifies the same retrieved reads for F1
    and throughput comparison
- name: PBSIM2
  role: 'Complementary read source: generates simulated CHM13 nanopore reads at 90%
    base-call accuracy to mix with the real SRA reads'
  repo: https://github.com/yukiteruono/pbsim2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:a8ae8544126c9e1d4629cd6b05b97ea7159be47d24e560d78a0a8058dee44212
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - PBSIM2
  - SPUMONI
  - minimap2
  edam_operation: http://edamontology.org/operation_0227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sra-read-retrieval

## Summary

Retrieve real nanopore sequencing read sets from NCBI SRA by accession (e.g. SRX6602475) so that classification benchmarks like the SPUMONI host-depletion experiment run on genuine data rather than simulations alone.

## When to use

Use when a benchmark workflow specifies an SRA accession (e.g. SRX6602475) as the real biological read input and the raw nanopore FASTQ/FASTA reads must be downloaded before classification or alignment.

## When NOT to use

- When the benchmark requires simulated reads (e.g. PBSIM2-generated CHM13 nanopore reads at 90% base-call accuracy) — use the simulator instead of SRA retrieval
- When the accession is not specified or the target data are reference genomes (e.g. CHM13, Ashkenazi, GRCh38), which come from genome assemblies, not SRA read archives

## Inputs

- NCBI SRA accession identifier (e.g. SRX6602475)
- Network access to the NCBI SRA/ENA repositories

## Outputs

- Downloaded real nanopore read set (FASTQ/FASTA) for the accession
- FASTA-format read file ('reads.fa') suitable for SPUMONI classification and minimap2 alignment

## How to apply

Identify the SRA accession named in the benchmark protocol (here, SRX6602475, the real microbiome nanopore read set used in the SPUMONI host-depletion experiment). Download the reads for that accession and convert them to a FASTA-format read set, since SPUMONI classification ('spumoni run -r <index> -p reads.fa -P -c') consumes FASTA sequences rather than raw signal. Keep the downloaded read set as the fixed 'real' input and pair it with simulated reads (e.g. PBSIM2-generated CHM13 nanopore reads at 90% base-call accuracy) to form the two-class benchmark. Verify retrieval by checking that the read count and total base pairs are non-zero and consistent before feeding the same reads to both SPUMONI and minimap2 so throughput (kbp/s) and F1 comparisons are fair.

## Related tools

- **SPUMONI** (Downstream consumer of the retrieved reads: classifies the FASTA read set against the SPUMONI index with PML matching statistics) — https://github.com/oma219/spumoni
- **minimap2** (Downstream baseline aligner that classifies the same retrieved reads for F1 and throughput comparison)
- **PBSIM2** (Complementary read source: generates simulated CHM13 nanopore reads at 90% base-call accuracy to mix with the real SRA reads) — https://github.com/yukiteruono/pbsim2

## Examples

```
spumoni run -r human3gen.index -p reads.fa -P -c
```

## Evaluation signals

- The download resolves the exact accession (SRX6602475) and returns a non-empty nanopore read set
- The same retrieved read set, converted to FASTA, is used for both SPUMONI and minimap2 classification so throughput (kbp/s) comparisons are apples-to-apples
- Reproduced F1/throughput values fall near the reported figures (F1 97.08 vs 99.17 three-genome; throughput 27.0 vs 13.5 kbp/s)
- Read count and total base pairs from the download remain constant across all benchmark configurations

## Limitations

- The article's methods section names no specific datasets or accessions ('no specific datasets or accessions named'), so the accession must come from the benchmark card; per-experiment KS-stat thresholds were optimized on real nanopore data, so read accuracy of the downloaded set affects classification decisions
- The retrieved reads are base-called sequences, not raw electrical signal; SPUMONI cannot directly interact with the Read Until API, and a separate base caller must already have been run upstream

## Evidence

- [other] Download the real microbiome nanopore read set from SRA accession SRX6602475.: [evidence span withheld — anchor status: not_found]
- [methods] Classify both read sets against each index using 'spumoni run -r <index> -p reads.fa -P -c' (PML mode with per-read classification report) and align the same reads to each reference with minimap2 as the baseline.: [evidence span withheld — anchor status: not_found]
- [discussion] it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API: "it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API"
- [discussion] we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets: "we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets"
