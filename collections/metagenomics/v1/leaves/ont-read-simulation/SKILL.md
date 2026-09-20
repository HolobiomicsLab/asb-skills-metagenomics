---
name: ont-read-simulation
description: 'Use when benchmarking host-read-removal precision on Nanopore data:
  reference-grade bacterial or mycobacterial assemblies must be turned into simulated
  ONT FASTQ reads so read retention (reads_out/reads_in) can be measured against tools
  like Hostile and HRRT.'
when_to_use_negative:
- You need to benchmark real human read removal on genuine human data (e.g. 1000 Genomes
  Project samples) — this skill covers simulated microbial reads only.
- Short-read (Illumina) benchmarking is the goal — that workflow uses DWGSIM simulation
  and Hostile's Bowtie2 mode, not the Minimap2/map-ont long-read path.
- Input reads are already decontaminated or you only need to inspect an existing index
  (e.g. via `hostile index list`); there is nothing to simulate or benchmark.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PBSIM2
  role: Simulates ONT long reads from FDA-ARGOS bacterial and mycobacterial assemblies
    to create the benchmark read sets
- name: Hostile
  role: Decontaminates simulated ONT reads in long-read (Minimap2, map-ont) mode against
    the human-t2t-hla index, emitting a JSON log with reads_in/reads_out/reads_removed
  repo: https://github.com/bede/hostile
- name: Minimap2
  role: Alignment engine used by Hostile's long-read mode (map-ont preset, secondary
    alignments disabled) to identify host-derived reads
- name: HRRT (sra-human-scrubber)
  role: Comparator tool run on the same simulated ONT read sets to compute reference
    retention percentages
  repo: https://github.com/ncbi/sra-human-scrubber
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:0c746a961a5f06c8c8f1023f714c42d63e62735537239aa11d4048c1a961d37b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - DWGSIM
  - HRRT
  - HRRT (sra-human-scrubber)
  - Hostile
  - Minimap2
  - PBSIM2
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# ont-read-simulation

## Summary

Simulates Oxford Nanopore long reads from reference-grade bacterial assemblies (PBSIM2) to benchmark how many microbial reads survive host-read decontamination with Hostile versus the NCBI Human Read Removal Tool (HRRT).

## When to use

Use when benchmarking host-read-removal precision on Nanopore data: reference-grade bacterial or mycobacterial assemblies must be turned into simulated ONT FASTQ reads so read retention (reads_out/reads_in) can be measured against tools like Hostile and HRRT.

## When NOT to use

- You need to benchmark real human read removal on genuine human data (e.g. 1000 Genomes Project samples) — this skill covers simulated microbial reads only.
- Short-read (Illumina) benchmarking is the goal — that workflow uses DWGSIM simulation and Hostile's Bowtie2 mode, not the Minimap2/map-ont long-read path.
- Input reads are already decontaminated or you only need to inspect an existing index (e.g. via `hostile index list`); there is nothing to simulate or benchmark.

## Inputs

- 985 reference-grade FDA-ARGOS bacterial genome assemblies (BioProject 231221)
- 140 complete mycobacterial assemblies
- Simulated ONT long-read FASTQ files (generated from assemblies with PBSIM2)

## Outputs

- Simulated ONT long-read FASTQ files per input assembly
- Hostile JSON clean log with reads_in, reads_out, and reads_removed counts
- Retention table (reads_out/reads_in) comparing Hostile vs HRRT on simulated ONT reads

## How to apply

Download the 985 reference-grade FDA-ARGOS bacterial genomes (BioProject 231221) and/or 140 complete mycobacterial assemblies, then simulate ONT long reads from each assembly with PBSIM2. Run Hostile in long-read mode (`hostile clean --fastq1 ...`), which uses Minimap2 with the map-ont preset and secondary alignments disabled, against the default human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51). Capture the JSON log with reads_in/reads_out/reads_removed counts, and compute bacterial read retention as reads_out/reads_in. Run HRRT on the same simulated ONT set for comparison, noting HRRT masks identified human reads with 'N' rather than deleting them. Validate against the paper's values: Hostile retained 99.98918% of simulated bacterial ONT reads (891 removed) vs HRRT 99.98901%, and 99.99948% of mycobacterial ONT reads vs HRRT 99.99864%, both exceeding the abstract-level guarantee of ≥99.989% retention.

## Related tools

- **PBSIM2** (Simulates ONT long reads from FDA-ARGOS bacterial and mycobacterial assemblies to create the benchmark read sets)
- **Hostile** (Decontaminates simulated ONT reads in long-read (Minimap2, map-ont) mode against the human-t2t-hla index, emitting a JSON log with reads_in/reads_out/reads_removed) — https://github.com/bede/hostile
- **Minimap2** (Alignment engine used by Hostile's long-read mode (map-ont preset, secondary alignments disabled) to identify host-derived reads)
- **HRRT (sra-human-scrubber)** (Comparator tool run on the same simulated ONT read sets to compute reference retention percentages) — https://github.com/ncbi/sra-human-scrubber

## Examples

```
hostile clean --fastq1 long.fastq.gz  # Creates long.clean.fastq.gz
```

## Evaluation signals

- Hostile ONT bacterial retention (reads_out/reads_in) equals 99.98918% with 891 reads removed, matching the paper's reported value
- Retention must exceed the abstract-level guarantee of at least 99.989% of simulated bacterial reads
- For 140 mycobacterial assemblies, Hostile ONT retention is ~99.99948% vs HRRT ~99.99864%
- Hostile retains more simulated ONT reads than HRRT in both bacterial and mycobacterial comparisons
- The Hostile JSON log records complete reads_in/reads_out/reads_removed counts for every simulated read set

## Limitations

- HRRT masks identified human reads with 'N' instead of removing them, so its retention figures must be interpreted against masked (not deleted) reads.
- Long-read decontamination with Minimap2 requires 13GB of RAM, versus 4GB for short reads with Bowtie2.
- Using a masked index raises bacterial retention to ≥99.997% but with ≤0.001% reduction in human read removal performance — a trade-off to state in any benchmark.
- HRRT's k-mer database tends to be aggressive about identifying human reads since it includes k-mers common to primates and other Eukaryota, which affects cross-tool comparability.

## Evidence

- [abstract] removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads: "removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [intro] Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin.: "Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin."
- [readme] In benchmarks, bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads.: "bacterial ONT reads at 22Mbp/s, using 8 alignment threads"
- [methods] The default index human-t2t-hla comprises T2T-CHM13v2.0 and IPD-IMGT/HLA v3.51, and is downloaded automatically when running Hostile unless another index is specified.: "The default index `human-t2t-hla` comprises [T2T-CHM13v2.0]"
- [methods] 985 reference grade bacterial genomes including common human pathogens: "985 reference grade bacterial genomes](https://www.ncbi.nlm.nih.gov/bioproject/231221) including common human pathogens"
