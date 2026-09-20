---
name: phage-host-network-crispr-validation
description: Use when you have Hi-C-inferred virus-host pairs from metagenomic Hi-C
  bins and want independent CRISPR spacer validation of those pairs, e.g. reproducing
  the wastewater phage-host network from ViralCC.
when_to_use_negative:
- When no Hi-C proximity-ligation data exist — CRISPR spacers alone cannot confirm
  pairs present in a given sample since some CRISPR associations are historical
- When viral contigs lack identifiable CRISPR arrays (PILER-CR finds none), as no
  spacer-based validation signal will be produced
- For discriminating viral from non-viral contigs — upstream viral contig identification
  (e.g. VirSorter) is a prerequisite, not part of this skill
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: ViralCC
  role: Recovers draft viral bins and Hi-C-inferred virus-host pairs from the Hi-C
    linkage graph to be validated
  repo: https://github.com/dyxstat/ViralCC
- name: HiCBin
  role: Provides host MAG contigs whose linkage to viral bins via the normalized Hi-C
    graph defines the pairs under validation
- name: PILER-CR
  role: Predicts CRISPR arrays in vMAG contig sequences from which spacers are extracted
- name: BLAST
  role: Aligns extracted CRISPR spacers against HiCBin host MAG contigs to identify
    protospacer matches
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:bbca12dbe1ccc1066873c54ac1d7ae3d8b95bdb76e5265b8270165181518df69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - BLAST
  - HiCBin
  - PILER-CR
  - ViralCC
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# phage-host-network-crispr-validation

## Summary

Validates Hi-C-inferred virus-host MAG pairs by independently predicting CRISPR spacers in vMAGs (PILER-CR) and aligning them against host MAG contigs (BLAST), then measuring the overlap between CRISPR-supported pairs and Hi-C linkages.

## When to use

Use when you have Hi-C-inferred virus-host pairs from metagenomic Hi-C bins and want independent CRISPR spacer validation of those pairs, e.g. reproducing the wastewater phage-host network from ViralCC.

## When NOT to use

- When no Hi-C proximity-ligation data exist — CRISPR spacers alone cannot confirm pairs present in a given sample since some CRISPR associations are historical
- When viral contigs lack identifiable CRISPR arrays (PILER-CR finds none), as no spacer-based validation signal will be produced
- For discriminating viral from non-viral contigs — upstream viral contig identification (e.g. VirSorter) is a prerequisite, not part of this skill

## Inputs

- Assembled wastewater shotgun contigs (FASTA, megahit, min-contig-len 1000)
- Name-sorted Hi-C alignment BAM (bwa MEM, samtools view -F 0x904, samtools sort by name)
- Viral contig list (e.g. VirSorter-identified viral contigs)
- HiCBin host MAG contig sequences (FASTA)
- ViralCC draft viral bins (VIRAL_BIN directory) and cluster_viral_contig.txt assignments

## Outputs

- List of robust CRISPR spacer-protospacer hits between vMAGs and host MAGs
- Hi-C-inferred virus-host MAG pair list from the normalized Hi-C linkage graph
- Summary table of infection-spectrum counts (CRISPR-only vs CRISPR+Hi-C supported pairs, as in Fig. 6b)
- Overlap fraction of CRISPR hits also associated by Hi-C (target: 13/16 = 81.3%)

## How to apply

Run the ViralCC pipeline on the wastewater assembled contigs (name-sorted Hi-C BAM, viral contig list, defaults --min-len 1000, --min-mapq 30, --min-match 30) to recover viral bins and infer virus-host pairs by linking ViralCC viral bins to HiCBin host MAG contigs via the normalized Hi-C linkage graph. Predict CRISPR arrays in the vMAG contig sequences with PILER-CR and extract the spacers, then align spacers against the HiCBin host MAG contigs with BLAST to find protospacer matches. Filter to robust spacer-protospacer hits and intersect them with the Hi-C-inferred virus-host pairs to compute the overlap fraction; tabulate infection-spectrum counts separating CRISPR-only pairs from pairs also supported by Hi-C (as in Fig. 6b). Validation: the output must reproduce 16 robust CRISPR hits with 13-of-16 (81.3%) overlap with Hi-C-inferred pairs and infection-spectrum counts matching Fig. 6b.

## Related tools

- **ViralCC** (Recovers draft viral bins and Hi-C-inferred virus-host pairs from the Hi-C linkage graph to be validated) — https://github.com/dyxstat/ViralCC
- **HiCBin** (Provides host MAG contigs whose linkage to viral bins via the normalized Hi-C graph defines the pairs under validation)
- **PILER-CR** (Predicts CRISPR arrays in vMAG contig sequences from which spacers are extracted)
- **BLAST** (Aligns extracted CRISPR spacers against HiCBin host MAG contigs to identify protospacer matches)

## Examples

```
piler-cr -g vMAG_contigs.fa > vMAG_crispr.txt; blastn -query spacers.fa -subject HiCBin_host_MAGs.fa -outfmt 6
```

## Evaluation signals

- The spacer-protospacer filtering yields exactly 16 robust CRISPR spacer hits between host MAGs and vMAGs
- Intersection with Hi-C-inferred pairs gives 13 of 16 (81.3%) overlap
- Infection-spectrum counts match Fig. 6b of the ViralCC paper
- BLAST hits only count as robust when spacer-protospacer alignments pass the quality filtering step
- ViralCC run uses default thresholds (--min-len 1000, --min-mapq 30, --min-match 30) and cluster assignments come from cluster_viral_contig.txt

## Limitations

- Some CRISPR-predicted virus-host associations indicate historical associations that may not be present in a given sample, so such pairs cannot be detected by Hi-C
- Some proximity-ligation virus-bacteria associations may arise from proximity of bacterial and viral DNA from mechanisms other than infection and should be validated before drawing conclusions
- A mismatch between CRISPR-only and Hi-C-supported pair counts is expected and not a pipeline failure, given historical spacer records

## Evidence

- [methods] workflow step 3: [evidence span withheld — anchor status: not_found]
- [methods] workflow step 5: [evidence span withheld — anchor status: not_found]
- [results] finding: [evidence span withheld — anchor status: not_found]
- [abstract] abstract validation: "When applied to a real wastewater metagenomic Hi-C dataset, ViralCC constructs a phage-host network, which is further validated using CRISPR spacer analyses."
- [discussion] CRISPR caveat: "some CRISPR-predicted virus-host associations indicate historical associations that may not be present in a given sample, and such pairs cannot be detected by Hi-C"
