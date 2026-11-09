---
name: exclusion-based-nonviral-data-filtering
description: Decision support for identifying viral contigs from assembled sequences
  when direct homology to known viruses is insufficient, e.g., novel phages in metagenomic
  assemblies needing gene-content-based viral classification.
when_to_use_negative:
- Input is raw short-read data where you want virMine's full QC-and-assembly pipeline
  (this skill presumes contigs already exist or are assembled by virMine)
- The goal is taxonomic classification of reads rather than scoring whole assembled
  contigs for viral origin
- No user-supplied nonviral database is available, since the exclusion step requires
  the -nv nonviral protein database
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0797
tools:
- name: virMine 2.0
  role: Implements gene prediction, exclusion-based filtering against the nonviral
    database, and contig scoring by gene-content origin
  repo: https://github.com/putonti/virmine
- name: BLAST
  role: Searches predicted ORF proteins (blastp or blastx selectable via -b) against
    viral and nonviral databases for scoring
- name: Python
  role: virMine is executed as python3 virMine.py; Python 3.9 is the runtime for the
    scoring script
  repo: https://github.com/putonti/virmine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:f5486c06af32c7158a01893a2c0c2667f986af234b81d7a6669dcae73a27fa36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/putonti/virmine
  tools:
  - BLAST
  - Python
  - virMine 2.0
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# exclusion-based-nonviral-data-filtering

## Summary

A contig-scoring strategy from virMine 2.0 that removes matches to a user-supplied nonviral database (exclusion technique) before scoring each contig by the origin of its gene content (cellular, viral, or unknown), eliminating sole dependency on homology to known viruses.

## When to use

Decision support for identifying viral contigs from assembled sequences when direct homology to known viruses is insufficient, e.g., novel phages in metagenomic assemblies needing gene-content-based viral classification.

## When NOT to use

- Input is raw short-read data where you want virMine's full QC-and-assembly pipeline (this skill presumes contigs already exist or are assembled by virMine)
- The goal is taxonomic classification of reads rather than scoring whole assembled contigs for viral origin
- No user-supplied nonviral database is available, since the exclusion step requires the -nv nonviral protein database

## Inputs

- Assembled contigs FASTA (supplied via virMine -A) or assembly produced by virMine from raw reads
- Viral protein database FASTA (user-supplied via -v, e.g., RefSeq eukaryotic viral and phage genomes)
- Nonviral protein database FASTA (user-supplied via -nv, e.g., COG-based cellular genes)

## Outputs

- viral_contigs.fasta — contigs classified as viral
- Predicted ORF files (all predicted ORFs and viral ORFs only)
- BLAST output files (blastp or blastx results against viral and nonviral databases)

## How to apply

Load assembled contigs via virMine's -A option, run gene prediction to obtain all predicted ORFs, then BLAST the predicted ORF proteins against the user-supplied viral database (-v) using blastp or blastx (the -b flag; blastp is faster). Apply the exclusion technique by removing ORFs matching the user-supplied nonviral database (-nv), so scoring does not rely solely on direct viral homology. Score each contig by the origin of its gene content — categorizing genes as cellular, viral, or unknown — and classify contigs whose gene content is viral in likelihood as viral. The rationale is that removing nonviral hits first allows remaining data to be scored on relatedness to viral elements, enabling detection of novel viruses absent from the viral reference database.

## Related tools

- **virMine 2.0** (Implements gene prediction, exclusion-based filtering against the nonviral database, and contig scoring by gene-content origin) — https://github.com/putonti/virmine
- **BLAST** (Searches predicted ORF proteins (blastp or blastx selectable via -b) against viral and nonviral databases for scoring)
- **Python** (virMine is executed as python3 virMine.py; Python 3.9 is the runtime for the scoring script) — https://github.com/putonti/virmine

## Examples

```
python3 virMine.py -A inputFiles/assembled_contigs.fasta -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -b blastp -o output
```

## Evaluation signals

- viral_contigs.fasta contains exactly the contigs whose predicted gene content is classified as viral after nonviral matches are removed
- BLAST outputs show ORFs matched against the viral database (-v) first and nonviral matches removed via the -nv database (exclusion technique applied)
- On the article's test data (E. coli K12 + Escherichia phage Lambda artificial reads assembled into 3 contigs), 1 of the 3 contigs in final_contigs.fasta is classified as viral
- Both ORF prediction files (all predicted ORFs and viral ORFs only) and all BLAST output files are retained in the output directory
- Using blastp (via -b blastp, the faster option) should yield the same classification of viral contigs as blastx

## Limitations

- Scoring depends on user-supplied viral and nonviral databases; coverage of these references affects classification of distant/novel viruses
- Contigs passing through this scoring step can be further reduced by optional filters (minimum/maximum contig size, minimum coverage), which may remove viral contigs the scorer would otherwise call
- The skill operates on assembled contigs; low-abundance viruses that fail to assemble will not be scored

## Evidence

- [abstract] virMine 2.0 uses an exclusion technique to remove nonviral data from sequencing reads and scores the remaining data based on relatedness to viral elements, eliminating the sole dependency on homology: "virMine 2.0 uses an exclusion technique to remove nonviral data from sequencing reads and scores the remaining data based on relatedness to viral elements, eliminating the sole dependency on homology"
- [abstract] Contigs are scored based on their gene content's origin, i.e., cellular, viral, or unknown.: "Contigs are scored based on their gene content’s origin, i.e., cellular, viral, or unknown."
- [readme] Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral: "Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral (viral_contigs.fasta)."
- [readme] python3 virMine.py -A inputFiles/assembled_contigs.fasta -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta: "python3 virMine.py -A inputFiles/assembled_contigs.fasta -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta"
