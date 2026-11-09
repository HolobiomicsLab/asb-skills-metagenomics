---
name: virus-host-linkage-detection-from-hi-c
description: Use when you have metagenomic Hi-C data (name-sorted BAM aligned to assembled
  contigs), identified viral contigs and host MAGs, and need to determine which phages
  infect which host MAGs and validate those pairings independently.
when_to_use_negative:
- No Hi-C (proximity-ligation) data are available — virus-host inference must then
  rely on co-abundance (CoCoNet, vRhyme) or CRISPR spacers alone
- Contigs are short and sparse Hi-C signal is expected without a complementary information
  source; ViralCC explicitly leverages the host proximity graph to compensate for
  the sparse Hi-C interaction map
- You only need viral genome binning without host assignment — virus-host linkage
  steps are unnecessary overhead
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0622
tools:
- name: ViralCC
  role: Recovers draft viral genomes (viral bins) from Hi-C data via integrated Hi-C
    interaction and host proximity graphs, and detects virus-host pairs from the normalized
    Hi-C linkage graph
  repo: https://github.com/dyxstat/ViralCC
- name: HiCBin
  role: Provides the host MAG contigs that ViralCC viral bins are linked to through
    the Hi-C linkage graph
- name: TAXAassign
  role: Annotates contig taxonomy for HiCBin to generate intra-species contacts during
    host MAG binning
  repo: https://github.com/umerijaz/TAXAassign
- name: PILER-CR
  role: Predicts CRISPR arrays in vMAG contigs to extract spacers for independent
    host-pair validation
- name: BLAST
  role: Aligns extracted CRISPR spacers against host MAG contigs to identify protospacer
    matches
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:968484230e9a0721a7c464c5d6c904e284fe8f55926ca6e5150c38de8c600768
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - BLAST
  - HiCBin
  - PILER-CR
  - TAXAassign
  - ViralCC
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# virus-host-linkage-detection-from-hi-c

## Summary

Infers and validates virus-host MAG pairs from metagenomic Hi-C data by linking ViralCC viral bins to HiCBin host MAG contigs through the normalized Hi-C linkage graph, then cross-validating the pairs with CRISPR spacer-protospacer matches (PILER-CR + BLAST). This is the core application that turns Hi-C proximity-ligation signal into a phage-host network.

## When to use

Use when you have metagenomic Hi-C data (name-sorted BAM aligned to assembled contigs), identified viral contigs and host MAGs, and need to determine which phages infect which host MAGs and validate those pairings independently.

## When NOT to use

- No Hi-C (proximity-ligation) data are available — virus-host inference must then rely on co-abundance (CoCoNet, vRhyme) or CRISPR spacers alone
- Contigs are short and sparse Hi-C signal is expected without a complementary information source; ViralCC explicitly leverages the host proximity graph to compensate for the sparse Hi-C interaction map
- You only need viral genome binning without host assignment — virus-host linkage steps are unnecessary overhead

## Inputs

- Assembled metagenomic contigs (FASTA, minimum contig length 1000)
- Name-sorted Hi-C alignment BAM (samtools sort by name, filtered with samtools view -F 0x904)
- List of identified viral contigs (e.g., from VirSorter)
- HiCBin host MAG contigs from the same Hi-C library
- vMAG contig sequences for CRISPR spacer prediction

## Outputs

- VIRAL_BIN directory of draft viral bins (vMAGs)
- cluster_viral_contig.txt per-contig cluster assignments
- Hi-C-inferred virus-host MAG pair list
- Robust CRISPR spacer-protospacer hit table and overlap fraction with Hi-C pairs (e.g., 13/16, 81.3%)
- Infection-spectrum summary counts separating CRISPR-only from Hi-C-supported pairs (Fig. 6b)

## How to apply

Run the ViralCC pipeline on the assembled contig FASTA, name-sorted Hi-C BAM, and viral contig list with default parameters (--min-len 1000, --min-mapq 30, --min-match 30) to recover draft viral bins and per-contig cluster assignments from cluster_viral_contig.txt. Infer virus-host pairs by linking ViralCC viral bins to HiCBin host MAG contigs through the normalized Hi-C linkage graph. Independently predict CRISPR arrays in the vMAG contigs with PILER-CR, extract spacers, and align them against host MAG contigs with BLAST to find protospacer matches; filter down to robust spacer hits. Intersect the CRISPR-inferred pairs with the Hi-C-inferred pairs to compute an overlap fraction — in the wastewater dataset, 13 of 16 robust CRISPR hits (81.3%) were also supported by Hi-C linkages. Note that CRISPR-only pairs may reflect historical associations absent from the sampled community, so Hi-C provides the present-time infection evidence.

## Related tools

- **ViralCC** (Recovers draft viral genomes (viral bins) from Hi-C data via integrated Hi-C interaction and host proximity graphs, and detects virus-host pairs from the normalized Hi-C linkage graph) — https://github.com/dyxstat/ViralCC
- **HiCBin** (Provides the host MAG contigs that ViralCC viral bins are linked to through the Hi-C linkage graph)
- **TAXAassign** (Annotates contig taxonomy for HiCBin to generate intra-species contacts during host MAG binning) — https://github.com/umerijaz/TAXAassign
- **PILER-CR** (Predicts CRISPR arrays in vMAG contigs to extract spacers for independent host-pair validation)
- **BLAST** (Aligns extracted CRISPR spacers against host MAG contigs to identify protospacer matches)

## Examples

```
python ./viralcc.py pipeline --min-len 1000 --min-mapq 30 --min-match 30 contigs.fa hic_reads.sorted.bam viral_contigs.txt VIRAL_BIN
```

## Evaluation signals

- Reproduce 16 robust CRISPR spacer hits between host MAGs and vMAGs on the wastewater dataset
- 13 of 16 (81.3%) CRISPR-supported virus-host MAG pairs are also associated by Hi-C linkages
- Infection-spectrum counts match Fig. 6b when tabulating CRISPR-only vs Hi-C-supported pairs
- Viral bin completeness assessed with CheckV; contamination cannot be estimated with CheckV so only completeness is meaningful
- cluster_viral_contig.txt assigns every viral contig to exactly one cluster; bins derive only from contigs in the provided viral contig list

## Limitations

- CRISPR-predicted virus-host associations may be historical and not present in the given sample; such pairs cannot be detected by Hi-C
- Some proximity-ligation virus-bacteria associations may arise from mechanisms other than infection and should be validated before drawing conclusions
- TAXAassign can hardly annotate viral contigs, so HiCBin alone cannot bin viral contigs — hence the dedicated ViralCC approach
- Fraction of spurious contacts in the host proximity graph cannot be estimated on real datasets because true viral contig labels are unknown

## Evidence

- [abstract] ViralCC leverages the virus-host proximity structure as a complementary information source for the Hi-C interactions.: "ViralCC leverages the virus-host proximity structure as a complementary information source for the Hi-C interactions."
- [methods] ViralCC pipeline invocation: "python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory"
- [methods] Default alignment filters: "--min-mapq: Minimum acceptable alignment quality (default 30)"
- [intro] HiCBin cannot bin viral contigs: "TAXAassign can hardly annotate viral contigs, resulting in the inability of HiCBin to bin viral contigs"
- [discussion] CRISPR historical-association caveat: "some CRISPR-predicted virus-host associations indicate historical associations that may not be present in a given sample, and such pairs cannot be detected by Hi-C"
- [readme] Reproduction instructions pointer: "If you want to reproduce results in our ViralCC paper, please read our instructions [here](https://github.com/dyxstat/Reproduce_ViralCC)."
