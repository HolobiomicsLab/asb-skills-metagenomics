---
name: taxonomic-phylum-distribution-tabulation
description: Use when a metagenomic genome-reconstruction study reports draft/complete
  genome counts and per-phylum assignments (e.g. from 16S rRNA or SCG-binned genomes)
  that must be extracted, verified, and tabulated into a structured distribution table.
when_to_use_negative:
- When the input is raw metagenomic reads or assembled scaffolds — genome reconstruction
  and binning (e.g. with abawaca) must precede tabulation
- When phylum-level groupings are needed at finer resolution than the paper's 75%
  sequence-identity cluster definition (e.g. species- or genus-level tables)
- When the source only reports relative abundance or 16S amplicon counts rather than
  reconstructed genome counts
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: fix_assembly_errors
  role: Curates the scaffolds/genomes (re_assemble_errors.py, ra2.py) whose counts
    and phylum assignments are tabulated, ensuring the genome set is error-free before
    distribution is recorded
  repo: https://github.com/christophertbrown/fix_assembly_errors
- name: abawaca
  role: Binning program using differential coverage and DNA signature that produced
    the genome bins whose phylum distribution is being tabulated
  repo: https://github.com/CK7/abawaca
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:bc2fe2daf862d89026a2f34ff51ae66910b2fbc1ba62b1df59c29e3eb5e3512b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/christophertbrown/fix_assembly_errors
  tools:
  - abawaca
  - fix_assembly_errors
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# taxonomic-phylum-distribution-tabulation

## Summary

Tabulates reconstructed genome yields and their phylum-level taxonomic distribution from a genome-resolved metagenomic study, e.g. the Brown et al. 2015 CPR dataset (8 manually curated complete and 789 draft genomes across >35 phyla), and assembles them into a reusable CSV.

## When to use

Use when a metagenomic genome-reconstruction study reports draft/complete genome counts and per-phylum assignments (e.g. from 16S rRNA or SCG-binned genomes) that must be extracted, verified, and tabulated into a structured distribution table.

## When NOT to use

- When the input is raw metagenomic reads or assembled scaffolds — genome reconstruction and binning (e.g. with abawaca) must precede tabulation
- When phylum-level groupings are needed at finer resolution than the paper's 75% sequence-identity cluster definition (e.g. species- or genus-level tables)
- When the source only reports relative abundance or 16S amplicon counts rather than reconstructed genome counts

## Inputs

- Article abstract and genome-yield table (Table 1) reporting complete and draft genome counts
- Supplementary genome list with phylum assignments for the manually curated complete genomes
- 16S rRNA gene cluster assignments at 75% sequence identity used for phylum counting

## Outputs

- genome_yields_phylum_distribution.csv with columns: genome category (complete/draft), phylum, count, genome size note
- Validated totals: 8 complete genomes (3+2+1+1+1 across phyla) and 789 draft genomes

## How to apply

Parse the article abstract and genome-yield table to extract headline counts — in the Brown et al. 2015 CPR study, 8 complete and 789 draft genomes from bacteria representing >35 phyla. Extract the phylum assignments of the manually curated complete genomes from the table and supplementary genome list: 3 Microgenomates, 2 Parcubacteria, 1 Kazan, 1 Berkelbacteria, 1 Saccharibacteria. Anchor phylum assignments to the paper's operational definition: 16S rRNA gene sequence clusters created at a 75% sequence identity threshold, using curated genes ≥800 bp. Record the size characteristic of the complete genomes (small, <1 Mb genomes typical of CPR) as a note column. Assemble the values into a CSV with columns for genome category (complete/draft), phylum, count, and genome size note, then validate that the complete-genome phylum rows (3+2+1+1+1) sum to 8 and that the draft row equals 789.

## Related tools

- **fix_assembly_errors** (Curates the scaffolds/genomes (re_assemble_errors.py, ra2.py) whose counts and phylum assignments are tabulated, ensuring the genome set is error-free before distribution is recorded) — https://github.com/christophertbrown/fix_assembly_errors
- **abawaca** (Binning program using differential coverage and DNA signature that produced the genome bins whose phylum distribution is being tabulated) — https://github.com/CK7/abawaca

## Examples

```
python -c "import pandas as pd; pd.DataFrame([['complete','Microgenomates',3],['complete','Parcubacteria',2],['complete','Kazan',1],['complete','Berkelbacteria',1],['complete','Saccharibacteria',1],['draft','>35 phyla',789]],columns=['category','phylum','count']).to_csv('genome_yields_phylum_distribution.csv',index=False)"
```

## Evaluation signals

- CSV contains yield rows with counts 8 (complete) and 789 (draft)
- Phylum-distribution rows for complete genomes (3+2+1+1+1) sum to 8
- Phylum assignments are traceable to 16S rRNA clusters at the 75% identity threshold from curated genes ≥800 bp
- Genome size note present for complete genomes (small, <1 Mb genomes characteristic of CPR)
- Total phylum-level diversity reported as >35 phyla, consistent with the abstract's claim

## Limitations

- Phylum counts depend on the paper's operational 75% sequence-identity clustering threshold, which differs from species-level 97% clustering and may not match other taxonomic schemes
- Draft-genome counts rely on the binning quality criterion (≥50% SCGs with fewer than 1.125 copies) and may shift if bin quality filters change
- CPR 16S rRNA genes often contain self-splicing insertions and divergent sequences, so phylum assignments require intron removal and structural alignment to be reliable

## Evidence
