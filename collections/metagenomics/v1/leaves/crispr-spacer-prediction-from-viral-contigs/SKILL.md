---
name: crispr-spacer-prediction-from-viral-contigs
description: Use when a metagenomic Hi-C pipeline (e.g. ViralCC + HiCBin) has produced
  draft viral bins and host MAGs, and you need sequence-level CRISPR spacer evidence
  to validate the inferred virus-host pairs or build an infection-spectrum summary
  like Fig. 6b.
when_to_use_negative:
- When the contigs are prokaryotic MAGs rather than viral bins — the skill targets
  CRISPR arrays within vMAGs; TAXAassign-type workflows 'can hardly annotate viral
  contigs', so do not substitute host-side workflows
- When no Hi-C-inferred virus-host pairs exist yet — the skill's validation logic
  (overlap fraction vs Hi-C linkages) requires the pair list from ViralCC/HiCBin as
  an input
- When contigs are already taxonomically classified viral sequences needing classification
  (use Demovir for Order/Family classification instead — it 'should not be used for
  discriminating viral contigs from bacterial/archael/eukaryotic sequences')
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: PILER-CR
  role: Predicts CRISPR arrays in vMAG contig sequences and provides the spacers used
    as probes against host MAGs
- name: BLAST
  role: Aligns extracted CRISPR spacers against HiCBin host MAG contigs to identify
    protospacer matches
- name: ViralCC
  role: Recovers the draft viral bins (vMAGs) whose contigs are scanned for CRISPR
    arrays, and supplies the Hi-C-inferred virus-host pairs being validated
  repo: https://github.com/dyxstat/ViralCC
- name: HiCBin
  role: Provides the host MAG contig sequences used as the BLAST database for spacer-protospacer
    matching
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:76e04c423f17f690c56469cb453baee80845f152bc99a196f66f2c7c50c92dd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - BLAST
  - HiCBin
  - PILER-CR
  - ViralCC
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# CRISPR spacer prediction from viral contigs

## Summary

Predicts CRISPR arrays in viral MAG (vMAG) contigs with PILER-CR, aligns extracted spacers against host MAG contigs with BLAST, and uses the resulting spacer-protospacer hits as an orthogonal, sequence-based validation of Hi-C-inferred virus-host pairs from ViralCC/HiCBin.

## When to use

Use when a metagenomic Hi-C pipeline (e.g. ViralCC + HiCBin) has produced draft viral bins and host MAGs, and you need sequence-level CRISPR spacer evidence to validate the inferred virus-host pairs or build an infection-spectrum summary like Fig. 6b.

## When NOT to use

- When the contigs are prokaryotic MAGs rather than viral bins — the skill targets CRISPR arrays within vMAGs; TAXAassign-type workflows 'can hardly annotate viral contigs', so do not substitute host-side workflows
- When no Hi-C-inferred virus-host pairs exist yet — the skill's validation logic (overlap fraction vs Hi-C linkages) requires the pair list from ViralCC/HiCBin as an input
- When contigs are already taxonomically classified viral sequences needing classification (use Demovir for Order/Family classification instead — it 'should not be used for discriminating viral contigs from bacterial/archael/eukaryotic sequences')

## Inputs

- vMAG contig sequences (FASTA) from ViralCC viral bins (VIRAL_BIN directory / cluster_viral_contig.txt)
- HiCBin host MAG contig sequences (FASTA)
- Hi-C-inferred virus-host MAG pair list from the ViralCC linkage graph
- Wastewater metagenomic Hi-C dataset context (e.g. ERR2282092-style SRA runs)

## Outputs

- Predicted CRISPR arrays per vMAG contig and extracted spacer sequences (PILER-CR output)
- Spacer-vs-host protospacer alignment hits (BLAST tabular output)
- Filtered set of robust CRISPR spacer hits (expected: 16 hits in the wastewater benchmark)
- Summary table of infection-spectrum counts separating CRISPR-only pairs from pairs also supported by Hi-C (as in Fig. 6b)

## How to apply

Run PILER-CR on the vMAG contig sequences (recovered from ViralCC's VIRAL_BIN directory / cluster_viral_contig.txt) to predict CRISPR arrays and extract the spacers. Align the extracted spacers against the HiCBin host MAG contigs using BLAST to identify protospacer matches. Filter alignments down to robust spacer-protospacer hits, then intersect these CRISPR-inferred virus-host pairs with the Hi-C linkage-inferred pairs from the ViralCC/HiCBin graph. Tabulate infection-spectrum counts separating CRISPR-only pairs from pairs also supported by Hi-C linkages, as in Fig. 6b of the ViralCC wastewater analysis (16 robust hits, 13/16 = 81.3% also Hi-C-associated).

## Related tools

- **PILER-CR** (Predicts CRISPR arrays in vMAG contig sequences and provides the spacers used as probes against host MAGs)
- **BLAST** (Aligns extracted CRISPR spacers against HiCBin host MAG contigs to identify protospacer matches)
- **ViralCC** (Recovers the draft viral bins (vMAGs) whose contigs are scanned for CRISPR arrays, and supplies the Hi-C-inferred virus-host pairs being validated) — https://github.com/dyxstat/ViralCC
- **HiCBin** (Provides the host MAG contig sequences used as the BLAST database for spacer-protospacer matching)

## Examples

```
python ./viralcc.py pipeline --min-len 1000 --min-mapq 30 --min-match 30 contigs.fa name_sorted.bam viral_contigs.txt VIRAL_BIN  # then PILER-CR on vMAG contigs; BLAST spacers vs HiCBin host MAG contigs
```

## Evaluation signals

- Count of robust spacer-protospacer hits: must reproduce the benchmark of 16 robust CRISPR spacer hits between host MAGs and vMAGs in the wastewater dataset
- Overlap fraction with Hi-C-inferred pairs: 13 of 16 hits (81.3%) must also be associated by the Hi-C linkages from ViralCC
- Infection-spectrum summary counts must match Fig. 6b of the paper when CRISPR-only pairs are tabulated separately from Hi-C-supported pairs
- Every CRISPR-inferred pair must reference a valid vMAG-to-host-MAG contig assignment from ViralCC's cluster_viral_contig.txt and the HiCBin binning
- BLAST alignments should only survive filtering if they are robust spacer-protospacer matches (min-match/min-mapq thresholds of the underlying Hi-C linkage were 30; apply comparable stringency to spacer hits)

## Limitations

- Some CRISPR-predicted virus-host associations indicate historical associations that may not be present in a given sample, and such pairs cannot be detected by Hi-C — so CRISPR-only pairs are not necessarily false negatives of Hi-C
- Some proximity-ligation virus-bacteria associations may arise from mechanisms other than infection and should be validated before drawing conclusions
- Contamination of vMAGs cannot be estimated with CheckV since there is no marker-gene set available for viruses, so spacer quality cannot be cross-checked against bin contamination directly

## Evidence

- [abstract] On a real wastewater metagenomic Hi-C dataset, ViralCC constructs a phage-host network, which is further validated using CRISPR spacer analyses.: [evidence span withheld — anchor status: not_found]
- [other] Predict CRISPR arrays in vMAG contigs with PILER-CR and BLAST spacers against host MAGs: [evidence span withheld — anchor status: not_found]
- [discussion] CRISPR-predicted associations may be historical and not detectable by Hi-C: "some CRISPR-predicted virus-host associations indicate historical associations that may not be present in a given sample, and such pairs cannot be detected by Hi-C"
- [intro] ViralCC detects virus-host pairs from recovered viral genomes and Hi-C linkages: [evidence span withheld — anchor status: not_found]
- [readme] Reproduction scripts for ViralCC paper figures are available: "Scripts to process the intermediate data and plot figures of our ViralCC paper are available [here](https://github.com/dyxstat/Reproduce_ViralCC/tree/main/Scripts)."
