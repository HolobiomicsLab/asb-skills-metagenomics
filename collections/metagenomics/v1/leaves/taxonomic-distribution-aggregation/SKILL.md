---
name: taxonomic-distribution-aggregation
description: Use when you have a filtered DiSCo hitlist (disco_hitlist.tsv) of DsrA/DsrB
  hits from (meta)genomic records and need to summarize which lineages and phyla carry
  the Dsr pathway.
when_to_use_negative:
- Input is raw HMMER hmmsearch output that has not yet been filtered with the DiSCo
  cutoffs and best-hit strategy — aggregation would include false positives
- Analysis targets a single protein family or enzyme type without needing taxonomic
  breadth across lineages and phyla
- The dataset consists only of the 103 complete genomes used for the initial benchmark
  rather than the expanded 195,878-record (meta)genomic dataset, so the 1738/1798/66
  aggregation targets do not apply
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: DiSCo
  role: Identifies and classifies DsrA/DsrB protein hits (protein family and predicted
    type) whose results are aggregated taxonomically
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: Scans the input protein set against the DiSCo HMM library (multi-threaded
    via -n) to produce the raw hits feeding the hitlist
- name: Perl 5
  role: Implementation language of DiSCo.pl and its bundled filter script that produces
    the filtered hitlist to aggregate
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:6584a52e38e8c742b0244bd67e182328bafc12cc2ebd308eeee63762d79a77ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - Perl 5
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# taxonomic-distribution-aggregation

## Summary

Aggregates filtered DiSCo hitlist results (DsrA/DsrB protein hits) by lineage and phylum to reproduce the taxonomic distribution of Dsr-dependent dissimilatory sulphur metabolism across 1738 micro-organisms, 1798 lineages, and 66 phyla from over 190 thousand (meta)genomic records.

## When to use

Use when you have a filtered DiSCo hitlist (disco_hitlist.tsv) of DsrA/DsrB hits from (meta)genomic records and need to summarize which lineages and phyla carry the Dsr pathway.

## When NOT to use

- Input is raw HMMER hmmsearch output that has not yet been filtered with the DiSCo cutoffs and best-hit strategy — aggregation would include false positives
- Analysis targets a single protein family or enzyme type without needing taxonomic breadth across lineages and phyla
- The dataset consists only of the 103 complete genomes used for the initial benchmark rather than the expanded 195,878-record (meta)genomic dataset, so the 1738/1798/66 aggregation targets do not apply

## Inputs

- Filtered DiSCo hitlist (disco_hitlist.tsv) with columns for sequence identifier, HMM name (protein family), predicted enzyme type, E-values, accuracy, and score/bias
- Input protein sequences in FASTA format assembled from 193,978 genomic/metagenomic assemblies plus 1900 literature-derived (meta)genomes
- Taxonomic assignment of each source (meta)genome record (lineage/phylum) for aggregation

## Outputs

- Count of micro-organisms with at least one DsrA and/or DsrB protein hit (expected 1738 vs. 103 from complete genomes alone)
- Lineage-level aggregation of classified DsrA/DsrB hits (1798 lineages across both prokaryotic domains)
- Phylum-level aggregation of classified hits (66 phyla)

## How to apply

First run DiSCo (perl DiSCo.pl) with the -i flag on the assembled FASTA protein set from the 193,978 assemblies plus 1900 literature-derived (meta)genomes, keeping only hits that pass the cutoffs (conditional and independent E-value <= 1e-10, accuracy >= 0.5, bias one order of magnitude below score) under the best-hit strategy per input sequence. Then parse the filtered hitlist (e.g. disco_hitlist.tsv, with configurable separator via -s 1 for comma) which lists each hit's protein family and predicted type. Map each hit back to its source (meta)genome record to obtain the micro-organism count (1738 records with at least one DsrA and/or DsrB hit), then group classified hits by lineage (1798 lineages) and phylum (66 phyla) across both prokaryotic domains. The rationale is that lineage/phylum-level aggregation converts per-sequence HMM classifications into the taxonomic diversity picture reported in the paper, enabling identification of newly recognized lineages such as Thaumarchaeota and Spirochaetes with Dsr-pathway metabolic potential.

## Related tools

- **DiSCo** (Identifies and classifies DsrA/DsrB protein hits (protein family and predicted type) whose results are aggregated taxonomically) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (Scans the input protein set against the DiSCo HMM library (multi-threaded via -n) to produce the raw hits feeding the hitlist)
- **Perl 5** (Implementation language of DiSCo.pl and its bundled filter script that produces the filtered hitlist to aggregate) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo

## Examples

```
perl DiSCo.pl -i metagenomes.faa -p disco -s 1 -o -n 8
```

## Evaluation signals

- The filtered hitlist file (disco_hitlist.tsv) exists and is delimited as configured (tab by default, or comma with -s 1)
- Each reported hit fulfils the DiSCo cutoffs: conditional and independent E-value <= 1e-10, accuracy >= 0.5, and bias one order of magnitude below the score
- The best-hit strategy was applied per input sequence before aggregation
- Aggregated lineage count matches the reported 1798 lineages across both prokaryotic domains
- Micro-organism count with at least one DsrA and/or DsrB hit matches the reported 1738 (expanded from 103 complete genomes), spanning 66 phyla

## Limitations

- The aggregation counts depend on the assembled input dataset (193,978 assemblies plus 1900 literature-derived (meta)genomes); different inputs will yield different lineage/phylum totals
- Presence of a reductive-type DsrAB complex alone is not a good proxy for use of oxidized sulphur compounds as electron acceptors, so taxonomic aggregation does not equate to confirmed physiology
- Hits depend on the DiSCo cutoff thresholds; adjusting user-defined cut-offs via the -t option changes which hits enter the aggregation

## Evidence

- [abstract] We ran DiSCo against a dataset composed of over 190 thousand (meta)genomic records and efficiently mapped Dsr-dependent dissimilatory sulphur proteins in 1798 lineages across both prokaryotic domains.: "We ran DiSCo against a dataset composed of over 190 thousand (meta)genomic records and efficiently mapped Dsr-dependent dissimilatory sulphur proteins in 1798 lineages"
- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [intro] The first step of DiSCo.pl consists of an hmmsearch of a given protein sequences set in FASTA format against the DiSCo HMM library: [evidence span withheld — anchor status: not_found]
- [abstract] It takes user-supplied protein sequences and lists the identified proteins and their classification in terms of protein family and predicted type.: "It takes user-supplied protein sequences and lists the identified proteins and their classification in terms of protein family and predicted type."
- [readme] README separator and sequence-output options: "perl DiSCo.pl -i example.faa -p example -s 1 -o #runs DiSCo with given filename prefix, specifies outfile separator, and produces sequence outfile"
